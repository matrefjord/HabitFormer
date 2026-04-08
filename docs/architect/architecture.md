# Architecture Specification: Habit Loop Habit Builder

## 1. Executive summary
The recommended architecture is a modular monolith web application with a single web frontend, a single backend application, and one primary relational database. This fits the MVP because the product is web-only, supports one primary user workflow, has modest initial scale expectations, and does not require integrations or distributed processing. The most important tradeoff is choosing simplicity and fast delivery over early flexibility for multi-platform clients, advanced notifications, or high-volume analytics.

## 2. Inputs consumed
Product definition consumed from prior stage:
- Lean PRD for Habit Loop Habit Builder
- MVP scope
- user stories
- functional requirements
- acceptance criteria
- product risks
- product assumptions
- product dependencies
- backlog priorities

Additional constraints and decisions consumed:
- MVP is web only
- MVP supports exactly one active habit
- prebuilt habit templates are included
- reward prompts are a mix of preset and custom
- reminders are optional
- educational tips and tricks are included
- no integrations, no social features, no native mobile requirement

## 3. Architecture drivers

### Functional drivers
- User can create exactly one active habit
- User can choose a prebuilt template or manual setup
- User can define and edit cue, routine, and reward
- User can optionally enable reminders
- User can log routine completion with timestamp
- System shows a reward prompt immediately after completion
- User can view history for the active habit
- User can archive or complete the current habit before starting another
- User sees educational tips during onboarding and ongoing use

### Nonfunctional drivers
- MVP should be fast to build and easy to maintain
- Web-only delivery should work well on desktop and mobile browsers
- Personal habit data should be private per user
- System should be simple to operate for a small team
- Notification and reminder implementation should avoid unnecessary infrastructure complexity
- Content such as templates and tips should be easy to update

### Delivery drivers
- Greenfield product
- Likely small team
- Strong need to keep MVP focused
- Architecture should support rapid iteration after initial launch

## 4. Assumptions
- Authentication is required so each user's habit data is isolated.
- MVP uses email and password or managed passwordless or social auth through a hosted provider rather than custom auth.
- Reminder support in MVP will use browser notifications plus in-app reminder state, with email reminders deferred unless explicitly added.
- Users may access the web app from mobile browsers, but no native push infrastructure is required in MVP.
- Completion history is lightweight and does not require advanced analytics infrastructure.
- Templates and educational tips can be stored in the application database for admin-free seed management in MVP.
- Reward prompts are text-based in MVP.
- There is no need for a separate admin portal in MVP; content can be seeded in code or managed directly in the database.
- A single-region deployment is sufficient for MVP.

## 5. Recommended architecture
Use a three-tier modular monolith:

- Frontend: React-based web app, preferably with Next.js for routing, SSR or SSG flexibility, and a clean full-stack deployment path
- Backend: Next.js server routes or a lightweight Node.js application layer using TypeScript
- Database: PostgreSQL as the single primary data store
- Auth: Managed authentication provider such as Auth0, Clerk, or Supabase Auth
- Hosting: Vercel, Netlify plus hosted DB, or similar managed platform
- Notifications: Browser notification scheduling where supported, backed by server-side reminder metadata

This is the simplest viable option because:
- one user-facing client
- one primary workflow
- one database
- no external integrations
- minimal ops burden
- strong alignment with rapid MVP iteration

Recommended technical shape:
- App style: modular monolith
- Primary stack: TypeScript across frontend and backend
- Persistence: PostgreSQL with ORM such as Prisma
- Content handling: templates and tips stored as relational records or seeded tables
- Background work: avoid queue infrastructure initially; use synchronous flows and lightweight scheduled checks only if needed later

## 6. System components and responsibilities

| Component | Responsibility | Key inputs and outputs | Dependencies |
| --- | --- | --- | --- |
| Web UI | Render onboarding, habit setup, active habit view, history, and tips | Inputs: user actions, API responses. Outputs: API requests, notification permission prompts | Backend API, Auth |
| Auth module | Sign-in, session validation, user identity | Inputs: credentials or provider tokens. Outputs: session and user identity | Managed auth provider |
| Habit management module | Create, read, update, archive active habit; enforce one-active-habit rule | Inputs: habit setup and edit requests. Outputs: habit state and validation errors | DB, Auth |
| Template module | Provide prebuilt habit templates for selection | Inputs: template queries. Outputs: template list and details | DB |
| Reward module | Resolve preset and custom reward prompt shown after completion | Inputs: completion event, habit config. Outputs: reward prompt payload | DB |
| Completion logging module | Record completion events and return updated history and status | Inputs: completion request. Outputs: saved completion record, active habit status, reward prompt | DB, Auth |
| Reminder settings module | Save optional reminder preferences and expose reminder state | Inputs: reminder preferences. Outputs: saved settings | DB, browser notification layer |
| Tip delivery module | Return onboarding and contextual educational tips | Inputs: app context, habit and template context. Outputs: tip content | DB |
| API layer | Expose typed endpoints for UI operations | Inputs: HTTP requests. Outputs: JSON responses | App modules, Auth |
| Persistence layer | Data access and transactional consistency | Inputs: ORM queries. Outputs: persisted entities | PostgreSQL |
| Observability layer | Log errors and basic app metrics | Inputs: app events and errors. Outputs: logs and alerts | Hosting and logging provider |

## 7. Data model

### Core entities

#### User
Represents an authenticated person using the app.

Suggested fields:
- `id`
- `auth_provider_id`
- `email`
- `created_at`
- `updated_at`

#### Habit
Represents the user's current or archived habit.

Suggested fields:
- `id`
- `user_id`
- `title`
- `status` with values `active`, `archived`, `completed`
- `source_type` with values `template`, `manual`
- `template_id` nullable
- `cue_text`
- `routine_text`
- `reward_mode` with values `preset`, `custom`, `mixed`
- `custom_reward_text` nullable
- `preset_reward_id` nullable
- `reminder_enabled`
- `reminder_time` nullable
- `timezone` nullable
- `started_at`
- `archived_at` nullable
- `created_at`
- `updated_at`

Constraint:
- At most one active habit per user

Recommended implementation:
- partial unique index on `user_id` where `status = 'active'`

#### HabitCompletion
Represents a logged completion event.

Suggested fields:
- `id`
- `habit_id`
- `completed_at`
- `created_at`

#### HabitTemplate
Represents a prebuilt template.

Suggested fields:
- `id`
- `name`
- `category`
- `description`
- `default_cue_text`
- `default_routine_text`
- `default_preset_reward_id` nullable
- `is_active`
- `sort_order`

#### RewardPrompt
Represents preset reward options.

Suggested fields:
- `id`
- `title`
- `prompt_text`
- `category` nullable
- `is_active`
- `sort_order`

#### EducationalTip
Represents short guidance content.

Suggested fields:
- `id`
- `title`
- `body`
- `tip_type` with values `onboarding`, `general`, `template_specific`
- `template_id` nullable
- `trigger_context` nullable
- `is_active`
- `sort_order`

#### ReminderDeliveryState
Optional MVP-lite table if reminder display state needs persistence.

Suggested fields:
- `id`
- `habit_id`
- `last_reminder_at` nullable
- `next_reminder_at` nullable
- `permission_state` nullable

### Relationships
- User 1 to many Habit
- Habit 1 to many HabitCompletion
- HabitTemplate 1 to many Habit
- RewardPrompt 1 to many Habit
- HabitTemplate 1 to many EducationalTip optionally via template-specific tips

### Lifecycle concerns
- Only one active habit per user at any time
- Archived or completed habits remain queryable for history and future insights
- Completion logging must be append-only for auditability and trend use later
- Reward prompt lookup should be deterministic at completion time
- Reminder settings should not block core habit flow if browser permissions are denied

## 8. API and interface contracts

### Auth and session endpoints
Purpose: establish and validate user session

Main concepts:
- `GET /api/session`
- response: authenticated user summary or unauthenticated state

Auth:
- session cookie or token from managed auth provider

### Templates endpoints
Purpose: fetch template list and template details

Endpoints:
- `GET /api/templates`
- `GET /api/templates/:id`

Response concepts:
- template metadata
- suggested cue, routine, and reward defaults

Auth:
- authenticated user for MVP consistency, though public read could be allowed later

### Active habit endpoints
Purpose: create, fetch, update, archive the user's active habit

Endpoints:
- `GET /api/habits/active`
- `POST /api/habits`
- `PATCH /api/habits/:id`
- `POST /api/habits/:id/archive`
- `POST /api/habits/:id/complete`

Request concepts:
- title
- source type
- template id
- cue text
- routine text
- reward mode
- custom reward text
- preset reward id
- reminder settings

Response concepts:
- habit summary
- status
- validation messages

Auth:
- required; user can only operate on own habit

Validation:
- reject `POST /api/habits` if user already has an active habit

### Completion endpoints
Purpose: log routine completion and return reward prompt

Endpoints:
- `POST /api/habits/:id/completions`
- `GET /api/habits/:id/completions`

Request concepts:
- optional client timestamp

Response concepts:
- completion record
- reward prompt payload
- updated streak or history summary if added later

Auth:
- required

### Reward prompt endpoints
Purpose: fetch preset reward options if needed separately

Endpoints:
- `GET /api/rewards`

Response concepts:
- preset reward ids and text

### Educational tips endpoints
Purpose: return onboarding or contextual tips

Endpoints:
- `GET /api/tips?type=onboarding`
- `GET /api/tips?templateId=...`
- `GET /api/tips?context=active-habit`

Response concepts:
- short tip list
- associated context metadata

### Reminder preference endpoints
Purpose: save or update reminder settings

Endpoints:
- `PATCH /api/habits/:id/reminder`

Request concepts:
- enabled
- reminder time
- timezone
- browser permission state optional

Response concepts:
- persisted reminder settings

## 9. Cross-cutting concerns

### Authentication and authorization
- Use hosted auth rather than building auth from scratch
- Every habit, completion, and reminder action must be scoped to the authenticated user
- Enforce ownership checks at API layer and query layer
- No role model beyond standard user in MVP

### Logging and monitoring
- Capture structured logs for:
  - auth failures
  - habit create, update, and archive attempts
  - completion logging failures
  - reminder settings failures
  - unhandled API errors
- Add lightweight product telemetry for:
  - template selected
  - habit created
  - reminder enabled
  - completion logged
  - tip viewed
- Use managed error tracking such as Sentry

### Error handling
- Return consistent JSON error shape:
  - `code`
  - `message`
  - `fieldErrors` optional
- Distinguish validation errors from unexpected server failures
- Handle duplicate active-habit creation gracefully with explicit product message

### Configuration and secrets
- Store DB credentials, auth secrets, and environment configuration in managed secret storage
- Separate environments for local, preview, and production
- Seed templates, rewards, and tips per environment

### Security and privacy
- Encrypt transport with HTTPS only
- Minimize stored personal data
- Store only app-relevant habit content
- Protect against common web vulnerabilities via framework defaults, ORM parameterization, CSRF and session best practices
- Avoid collecting unnecessary behavioral metadata in MVP
- Provide account deletion or data deletion path later if needed; not required for first implementation milestone unless policy requires it

## 10. Nonfunctional requirements and technical implications

### Maintainability
Implication:
- modular monolith with clear internal domains:
  - auth
  - habits
  - completions
  - templates
  - rewards
  - tips
  - reminders

### Fast delivery
Implication:
- choose one TypeScript stack end to end
- use managed auth and managed hosting
- avoid background queue and service decomposition

### Privacy
Implication:
- per-user data isolation
- no shared or social data paths
- limited PII footprint

### Reliability
Implication:
- core user actions should be synchronous and transactional
- create, update, archive, and complete flows should use DB transactions where consistency matters

### Performance
Implication:
- app reads and writes are low-complexity
- PostgreSQL with indexed lookups is sufficient
- no caching layer required initially

### Content agility
Implication:
- tips, templates, and rewards should be stored as editable data, not hardcoded UI strings only
- seed scripts should support future content refreshes

## 11. Key tradeoffs

### Chosen approach
Modular monolith with Next.js and PostgreSQL

Why:
- simplest viable build
- low operational overhead
- one team can move quickly
- easy to evolve

### Rejected alternative: microservices
Rejected because:
- no scale or org complexity justifies it
- would slow development
- adds deployment, observability, and data consistency overhead

### Rejected alternative: frontend-only app with local storage
Rejected because:
- cannot reliably support authenticated multi-device usage
- poor durability for user history
- weak foundation for future reminder and tip evolution

### Rejected alternative: NoSQL-first design
Rejected because:
- relational constraints are useful for one-active-habit enforcement and clean lifecycle modeling
- PostgreSQL is simpler and more robust for this use case

## 12. Risks and mitigations

| Risk | Type | Impact | Mitigation |
| --- | --- | --- | --- |
| Browser reminders are inconsistent across browsers and devices | Technical | Reminder feature may feel unreliable | Keep reminders optional, message supported behavior clearly, design core value without dependency on reminders |
| Single active habit rule needs strong enforcement | Technical and Product | Logic bugs could break core product promise | Enforce in both application logic and DB unique constraint |
| Template quality may drive activation results | Product and Content | Poor early retention | Start with small curated template set and test usage |
| Tip content may feel generic | Product and Content | Reduced differentiation | Tie tips to onboarding and template context where possible |
| Auth provider integration complexity | Delivery | Can delay launch | Use a mature hosted auth provider with well-supported SDK |
| Reward prompt design may be underwhelming | Product | Weak reinforcement | Store prompts as data so they can be iterated quickly without architecture changes |

## 13. Implementation phases

### Phase 1: foundation
- Set up repo, TypeScript standards, linting, formatting
- Configure hosting, database, and auth provider
- Establish ORM schema and migrations
- Implement user and session flow
- Seed templates, reward prompts, and educational tips
- Add core API scaffolding and error conventions

### Phase 2: MVP build
- Build onboarding flow
- Build template selection and manual habit creation
- Implement active habit read, update, and archive flows
- Enforce one-active-habit rule
- Implement completion logging
- Return reward prompt on completion
- Build history view
- Build optional reminder settings UI and persistence
- Display onboarding and contextual tips

### Phase 3: hardening
- Add structured logging and error tracking
- Add validation tests and ownership and security tests
- Improve empty states and failure handling
- Verify browser notification behavior across major browsers
- Add analytics events for funnel measurement

### Phase 4: later extensions
- Email reminders if browser reminders prove insufficient
- Template-specific and behavior-triggered tips
- Streak summaries or reflection views
- Multi-habit support if product direction changes
- Admin or content management interface
- Native mobile clients or PWA enhancements

## 14. Open technical questions
- Which auth provider should be used in the stack?
- Should reminder support in MVP use browser notifications only, or also email reminders?
- Is PWA installation or offline support desired for the web app, or explicitly deferred?
- Should archived habits remain viewable in MVP or only the active habit history?
- How dynamic should educational tip triggering be in MVP: fixed placement or context-sensitive rules?
- Should seeded content be managed via migrations, seed scripts, or simple internal JSON bootstrapping?

## 15. Developer handoff

### Suggested repo or module structure
```text
/apps/web
  /app or /pages
  /components
  /features
    /auth
    /habits
    /templates
    /completions
    /rewards
    /tips
    /reminders
  /lib
    /db
    /auth
    /validation
    /errors
    /telemetry
  /api
/prisma
  schema.prisma
  seed.ts
/docs
  prd.md
  architecture.md
```

### Initial build order
1. Project foundation and auth
2. Database schema and seed content
3. Active habit model and one-active-habit enforcement
4. Template and manual habit setup flow
5. Completion logging plus reward prompt response
6. History view
7. Reminder preferences
8. Educational tip surfaces
9. Observability and hardening

### Critical test areas
- one-active-habit enforcement
- authenticated ownership checks
- habit create, edit, and archive lifecycle
- completion logging correctness
- reward prompt resolution logic
- reminder preference validation
- template selection and default field hydration
- educational tip retrieval by context

### First implementation milestones
- User can sign in and create one habit from a template
- User can view and edit the active habit
- User can log a completion and see reward feedback
- User can view completion history
- User can archive the habit and create a new one
