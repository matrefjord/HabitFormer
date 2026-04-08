# PRD: Habit Loop Habit Builder

## Product summary
Habit Loop Habit Builder is a web app that helps individuals build one positive habit at a time using the cue-routine-reward framework. The MVP is intentionally constrained to a single active habit so users focus on consistency instead of juggling multiple goals. The app combines prebuilt habit templates, optional reminders, completion tracking, mixed reward prompts, and lightweight educational tips to help users understand and apply the Habit Loop in practice.

## Problem statement
People often fail to build habits because they try to change too much at once and rely on motivation instead of structure. Most habit tools emphasize tracking, but do not help users define a clear cue, perform a repeatable routine, and reinforce the behavior with a reward. Without that structure and focus, users abandon the habit before it becomes consistent.

## Target users

### Primary user
- **Role:** Individual consumer trying to build one meaningful positive habit  
- **Goals:**
  - Focus on one habit without distraction
  - Use a structured loop to make the habit stick
  - Learn practical behavior-building techniques
  - Track consistency over time  
- **Pain points:**
  - Overcommits to too many habits
  - Falls off after a few missed days
  - Needs more guidance than a simple checklist provides
  - Wants help understanding how to make a habit actually work  

### Secondary users
- None in MVP

## Goals and success metrics
- **Business goal:** Validate a differentiated web habit product based on focused single-habit formation using the Habit Loop model  
- **User outcome:** Users consistently perform one chosen habit more often over time and understand how to improve the loop  
- **Success metrics:**
  - 65% of new users complete setup of one active habit
  - 55% of users who complete setup log at least 3 completions in their first 7 days
  - 35% of week-1 active users remain active in week 4
  - 50% of users engage with at least one educational tip
  - 40% of logged completions include reward prompt interaction  

## MVP scope

### In scope
- Web-only experience
- One active habit per user
- Habit creation via template or manual setup
- Cue, routine, reward configuration
- Mixed reward prompts (preset + custom)
- Optional reminders
- Routine completion logging
- Basic completion history
- Educational tips and tricks
- Edit active habit
- Archive/complete habit before starting a new one  

### Out of scope
- Multiple active habits
- Native mobile apps
- Social features or accountability
- Advanced analytics
- AI recommendations
- Integrations (calendar, wearables, etc.)
- Breaking bad habits
- Coaching marketplace or multi-user roles  

## User stories
- As a user, I want to choose a prebuilt habit template or start from scratch, so that I can begin quickly.  
- As a user, I want to focus on only one active habit at a time, so that I stay consistent.  
- As a user, I want to define a cue, routine, and reward, so that the habit is structured.  
- As a user, I want optional reminders, so that I can decide how much prompting I need.  
- As a user, I want to log completion, so that I can track consistency.  
- As a user, I want a reward prompt after completion, so that I reinforce the behavior.  
- As a user, I want educational tips, so that I can improve my habit loop.  
- As a user, I want to archive my habit before starting another, so that I stay focused.  

## Functional requirements
- **FR-1.** The system must allow a user to create one active habit.  
- **FR-2.** The system must prevent multiple active habits.  
- **FR-3.** The system must support template-based and manual setup.  
- **FR-4.** The system must allow editing of cue.  
- **FR-5.** The system must allow editing of routine.  
- **FR-6.** The system must allow preset + custom reward prompts.  
- **FR-7.** The system must allow optional reminders.  
- **FR-8.** The system must support at least one reminder type.  
- **FR-9.** The system must log completion with timestamp.  
- **FR-10.** The system must show reward prompt after completion.  
- **FR-11.** The system must store and display history.  
- **FR-12.** The system must display educational tips.  
- **FR-13.** The system must allow habit archive/completion.  
- **FR-14.** The system must display current habit status on main screen.  

## Acceptance criteria
- Users can create exactly one active habit.  
- System blocks creating another active habit.  
- Users can choose templates or manual setup.  
- Users can enable or skip reminders.  
- Completion can be logged in ≤2 interactions.  
- Reward prompt appears immediately after completion.  
- Users can view completion history.  
- Users can edit habit details.  
- At least one educational tip appears during onboarding and use.  
- App functions fully in a web browser.  

## Risks

### Risks
- Single-habit constraint may limit appeal  
- Weak reward prompts reduce differentiation  
- Web reminders may be less effective  
- Template quality impacts activation  

### Assumptions
- Focus is a key differentiator  
- Templates improve onboarding  
- Mixed reward model balances structure and flexibility  
- Educational tips improve retention  
- Web-only MVP is sufficient  

### Dependencies
- Template content design  
- Educational content design  
- Reminder mechanism for web  
- UX for habit lifecycle  

## Open questions
- What templates ship in MVP?  
- What defines habit completion lifecycle?  
- Should tips be generic or context-aware?  
- What reminder mechanism (browser, email)?  

## Prioritized backlog

| Priority | Item | User value | Notes |
|----------|------|-----------|-------|
| P0 | Create one habit | Core workflow | Foundation |
| P0 | Enforce single habit | Focus | Key differentiator |
| P0 | Templates | Faster onboarding | Critical |
| P0 | Cue/routine/reward setup | Core model | Essential |
| P0 | Log completion | Core action | Daily use |
| P0 | Reward prompt | Reinforcement | Differentiator |
| P0 | History view | Progress | Retention |
| P1 | Optional reminders | Support cue | Keep optional |
| P1 | Edit habit | Improve fit | Iteration |
| P1 | Educational tips | Guidance | Key positioning |
| P1 | Archive habit | Lifecycle | Required |
| P2 | Template-specific tips | Personalization | Later |
| P2 | Weekly summary | Insight | Later |
| P2 | Advanced rewards | Engagement | Later |

---

# Architecture Handoff Summary

## Lean PRD summary
Build a web-only habit app supporting one active habit. Users define cue, routine, reward, optionally use reminders, log completions, receive reward prompts, and learn via tips.

## MVP scope
- Single habit
- Templates + manual setup
- Cue/routine/reward
- Optional reminders
- Completion logging
- Reward prompts
- History
- Educational tips
- Archive lifecycle

## Critical user stories
- Create one focused habit  
- Log completion  
- Receive reinforcement  
- Learn habit techniques  
- Complete/archive habit  

## Constraints
- Web only  
- No multi-habit support  
- No integrations  
- No social features  

## Risks
- Reminder effectiveness  
- Template quality  
- Tip relevance  
- Perceived limitation of single-habit model  
