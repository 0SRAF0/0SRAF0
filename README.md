# IDEA 1: Schedule Creator

## Agent Loop

- Perception/Inputs: course schedule, work, prior engagements, holidays, special events (irl/online) based on interests + location
    - Google/Apple Calendar
    - Extra ideas:
        - Apple/Samsung health for sleep schedule
        - Microsoft teams, google teams
        - Github project, Jira
        - Canvas: class schedules, assignment deadlines, exams, quizzes
    - Manual input of priority, interests (for searching special events), personality (introvert/extrovert)
        - times to eat (whether they eat breakfast, what times they usually eat)
            - Whether they need time to cook
        - Morning shower/night shower
    - AI asks questions during the day in the first couple of weeks
        - What are you doing right now?
        - Are you still doing [blank]?
- Reasoning/LLM:
    - calculate everyday's/ weekly schedule based on inputs
    - Compare the priority of events to see which needs to be done earlier or at a specific time
    - Mark complications in time or work management
        - E.g., prior events, cannot go to events due to travel time
        - too many tasks required to do within a certain amount of time (unhealthy for the user)
- Actions (tool/API)
    - Display schedule for day/week/month
    - Create alarms for the day (warning 5 min before the event ends)
- Reflection (memory/feedback):
    - Shift the day’s schedule whenever the user wakes up (first time they open their phone)
        - Shift whenever the alarm is snoozed (e.g., the user needs more time to finish the task)
    - New hobbies that they want to set time for
    - Changes in their calendar (Apple/Google)
    - Shift to time to take into account traffic issues/safety issues -> amber alert in areas

## Architecture

Schedule should:

- Start at the user's wake-up:
    - Display the day’s schedule
    - Change the tasks the user wants or does not want to do today
- During the day:
    - User gets alarms for whenever the current event finishes
    - The user may delay the schedule
        - App fixes the rest of the day’s schedule to take into account
    - User marks a task as done
        - Events are automatically shown as completed
- At the end of the day:
    - Display completed tasks and accomplishments
    - Display unfinished tasks
    - Display tomorrow’s schedule
        - Can be adjusted + change priorities
- At the end of the week
    - Display stats of velocity for completed tasks

At all times:

- The schedule can be changed
    - Priority changes
    - Tasks finished/unfinished

Sample Schedule:

9:00 AM - Wake up + get ready

10:00 AM - Breakfast

Future Ideas:

- Sharing with other users
    - Invite to an event
- Create groups of users
    - AI recommends a time/date for events based on all users' schedules
    - Creates group even that is added to each user’s schedule
- Suggest tips while doing the event
    - Recipes for meals
    - Outline objective + steps for each task
    - Events
        - Recommend outfits/clothes
        - Google Maps directions to the location

Cursor results for the step to implement:

### **Phase 0 — Requirements and guardrails**

- Core user outcome: reduce planning overhead; generate feasible, healthy daily plans.
- Success metrics:
- Time-to-first-schedule < 5 minutes after onboarding.
- Daily adherence rate ≥ 60% within 2 weeks.
- Snooze-triggered replans succeed in < 2 seconds.
- Weekly velocity: tasks completed / planned trend upward after week 2.

### **Phase 1 — Architecture and foundations**

- Tech stack (suggested)
- Mobile app: React Native + Expo (iOS/Android).
- Backend: Node.js (Nest/Fastify) or Python (FastAPI) with Postgres.
- Scheduler engine: co-located backend service (TypeScript or Python).
- Queue + jobs: Redis + BullMQ / RQ.
- LLM: OpenAI or local via server-side orchestrator.
- Auth: OAuth2 + JWT. Secrets in Vault or cloud KMS.
- High-level components
- Data ingestion: Google Calendar, manual inputs, later Apple/Samsung Health.
- Core domain: User, Preferences, Event, Task, ScheduleBlock, TravelBlock, Memory.
- Scheduler: constraint solver + heuristics; replan triggers.
- Notifications: OS notifications + alarms.
- Memory/reflection: embeddings + structured preferences updater.
- Observability: logs, metrics, weekly analytics.
- Data flow (MVP)
- Onboarding collects preferences -> store.
- Pull Google Calendar events -> normalize -> DB.
- Generate schedule for today/tomorrow -> store ScheduleBlocks.
- Push to app UI + create alarms.
- Real-time triggers (wake/snooze/delay) -> replan -> update UI/alarms.

### **Phase 2 — Domain modeling (MVP scope)**

- Entities
- User, PreferenceProfile (wake window, meals, shower, cooking needed, intro/extro, interests).
- ExternalCalendarAccount (provider, tokens, sync state).
- Event (external calendar, fixed time, location).
- Task (deadline, duration estimate, priority, flexible).
- ScheduleBlock (type=Task/Event/Sleep/Meal/Shower/Travel, start, end, flexibility, source).
- MemoryEntry (feedback, habit signals, adherence, new hobbies).
- AlertSignal (traffic/safety).
- Key relations
- User 1..* Tasks, Events, ScheduleBlocks, Memories.
- Task may be split into multiple ScheduleBlocks (chunks).
- Event -> ScheduleBlock mirror for fixed items.

### **Phase 3 — Integrations (MVP: Google Calendar + manual prefs)**

- Google Calendar
- OAuth + offline token; use incremental sync tokens.
- Read primary calendar, write planned blocks to a separate “Agent Loop” calendar.
- Handle timezones/daylight transitions.
- Manual inputs
- Onboarding wizard for preferences and constraints.
- Daily add/edit tasks with duration, due date, priority.
- Health (post-MVP)
- Abstract provider interface; implement Apple HealthKit/Samsung Health later.

### **Phase 4 — Scheduling engine v1**

- Inputs
- Today’s fixed events, travel times (optional MVP), tasks with durations/deadlines, preferences (wake time, meals, showers), buffers.
- Constraints
- Hard: fixed events, sleep boundaries, meal windows, task deadlines, max daily hours.
- Soft: personality pacing, break frequency, context switching cost, intro/extro social load.
- Algorithm (pragmatic)
- Build timeline from wake to sleep.
- Place fixed events.
- Insert required routines (meals, shower) respecting windows.
- Greedy pack tasks by priority/deadline with chunking (e.g., 25–90 min blocks), add buffers.
- If overfull: push lower priority tasks to tomorrow; flag “unhealthy load”.
- Outputs
- List of ScheduleBlocks with justifications and flexibility tags.
- Conflicts report and suggestions.
- Replanning triggers
- App open after wake, alarm snooze/delay, event overrun, manual delay.
- Recompute only the remaining day (keep completed blocks frozen).

### **Phase 5 — Notifications and alarms**

- Local notifications for next block start; “5 min before end” wrap-up alerts.
- Quick actions: “start late by X”, “extend 10/20/30”, “done”.
- When snoozed/extended -> replan rest of day and update calendar + alarms.

### **Phase 6 — Memory and reflection**

- Capture: block start/end times, extensions, skips, location adherence, mood.
- Daily review: show completed vs planned; ask lightweight questions (“Are you still doing X?”, “What are you doing now?” for first 2 weeks).
- Update preference priors: typical focus length, meal times drift, wake variability.
- New hobbies pipeline: detect interests; propose time slots next week.

### **Phase 7 — Safety/traffic adjustments (post-MVP)**

- Maps API for travel-time between blocks; add TravelBlocks; prevent impossible transitions.
- Alerts feed (Amber/weather/safety) -> mark areas/time windows as unsafe -> suggest remote/alternate timing.

### **Phase 8 — UI flows**

- Onboarding: prefs, accounts connect, first schedule preview.
- Day view: now/next, timeline, delay/done controls.
- Week/month: high-level plan, deadlines, exams, quizzes.
- End-of-day: accomplishments, unfinished, tomorrow draft.
- End-of-week: velocity stats and insights.

### **Phase 9 — Analytics and weekly reports**

- Metrics: adherence rate, average focus block length, slip ratio, schedule churn.
- Weekly summary with trends; suggest habit tweaks.

### **Phase 10 — Ship MVP**

- Scope: Google Calendar, manual prefs, scheduler v1, notifications, daily replan.
- Non-functional: privacy, data export, rate limits, offline tolerant.

Implementation checklist (ordered)1) Auth + Google Calendar connector with read-only, then write to dedicated calendar.2) Domain models + migrations.3) Preferences onboarding UI and API.4) Task CRUD.5) Scheduler v1 service and endpoints.6) Day view UI + local notifications.7) Replan on snooze/delay/open app after wake.8) Daily review and memory logging.9) Weekly report basic velocity.Stretch goals after MVP

- Apple Calendar, Health integrations.
- Travel-time and safety signals.
- Interest-based event discovery with location filters.
- Advanced optimization (ILP/CP-SAT) for complex weeks.
- Social load modeling based on intro/extro and recovery time.

# Idea 2: AI-Powered Group Travel Planning System
**Enterprise Agentic Platform for Collaborative Trip Coordination**
Eliminate the chaos of group travel planning. This system collects preferences, generates personalized recommendations, facilitates democratic decision-making, and delivers actionable itineraries—all autonomously.

## How It Works
1. **Create a trip and add participants**
2. **System sends SMS surveys** to collect preferences (budget, activities, pace, dietary needs, accessibility)
3. **AI analyzes responses** and generates destination recommendations with real-time pricing
4. **Group votes** using ranked-choice system to reach consensus
5. **System builds itinerary** with bookings, activities, and optimization

## What You Get
### 1. Smart Recommendations
- **Destination matches** based on aggregated group preferences
- **Conflict indicators**: highlights where preferences diverge (e.g., budget vs. luxury split)
- **Seasonality awareness**: suggests optimal travel windows
- **Real-time pricing**: live flight, hotel, and activity costs from multiple sources
- **Trade-off analysis**: "Save $800 by traveling 2 weeks earlier" or "Add $200/person for beachfront"

### 2. Democratic Consensus
- **Ranked-choice voting interface** with visual comparisons
- **Consensus score**: shows alignment percentage for each destination
- **Alternative suggestions**: if no clear winner, system proposes compromise options
- **Instant runoff results**: transparent vote tallying with explanations

### 3. Actionable Itineraries
- **Day-by-day plans** with optimized routing
- **Booking links** for approved options with price locks
- **Group coordination**: shared calendar, task assignments (who books flights, who reserves restaurants)
- **Budget tracking**: live spend vs. plan with alerts

## Inputs & Outputs
### Inputs
- **Trip details**: destination type (beach, city, adventure, cultural), duration, dates (flexible or fixed)
- **Participants**: names, contacts, dietary restrictions, accessibility needs
- **Survey responses**: budget range, activity preferences, pace (relaxed/moderate/packed), must-haves, deal-breakers
- **Constraints**: visa requirements, travel time limits, blackout dates

### Outputs
- **Preference summary**: group trends in one sentence ("Budget-conscious group favoring cultural experiences with moderate activity")
- **Destination shortlist**: 3-5 options with scoring breakdown (preference match %, cost, travel time)
- **Pricing dashboard**: real-time costs for flights, lodging, activities per destination
- **Voting results**: ranked outcomes with consensus metrics
- **Final itinerary**: day-by-day plan with bookings, maps, contact info, contingencies

## Implementation Flow
### 1. Trip Initialization
- User creates trip → Orchestrator Agent spins up workflow
- Sends SMS surveys via Twilio API
- Stores responses in PostgreSQL

### 2. Preference Processing
- Survey responses hit message queue (RabbitMQ/Kafka)
- **Preference Analysis Agent** (microservice) processes them:
  - Calls LLM (GPT-4/Claude) to extract patterns
  - Stores embeddings in vector DB for semantic matching
  - Outputs: preference_package JSON

### 3. Parallel Research Phase
Orchestrator triggers two agents simultaneously:

**Destination Research Agent:**
- Queries vector DB for similar destinations
- Calls external APIs (visa requirements, weather)
- Returns shortlist

**Real-Time Pricing Agent:**
- Polls Amadeus/Skyscanner APIs every 6 hours
- Caches in Redis with TTL
- Pushes updates via WebSocket to frontend

### 4. Voting
- Frontend displays options with live pricing
- **Voting Coordinator Agent** runs instant runoff algorithm
- If consensus < 70% → triggers **Conflict Resolution Agent**
- Stores votes in PostgreSQL, updates state in Redis

### 5. Itinerary Generation
- **Itinerary Generation Agent** (long-running task on Celery):
  - Calls LLM with structured prompt (winning destination + preferences)
  - Uses Google Maps API for routing optimization
  - Generates day-by-day plan
- **Budget Optimization Agent** reviews for savings
- **Verifier Agent** validates (timing, availability, accessibility)

### 6. Delivery
- Final itinerary stored in PostgreSQL
- Booking links generated via affiliate APIs
- Users notified via SMS/email
- Real-time collaboration via WebSocket (comments, task assignments)

## Tech Stack in Action
- **Agent Runtime**: LangGraph orchestrates agent calls, manages state transitions
- **Communication**: gRPC for inter-agent calls (faster than REST)
- **State Management**: Redis for session state, PostgreSQL for persistent data
- **Async Tasks**: Celery workers handle long-running agent operations
- **APIs**: FastAPI microservices, one per agent
- **Frontend**: Next.js with WebSocket client for live updates
- **Deployment**: Kubernetes pods (each agent = separate deployment), auto-scales based on queue depth

## Agent Architecture
### Core Agents

1. **Orchestrator Agent**
   - Coordinates workflow across all agents
   - Manages state transitions
   - Handles failures and retries

2. **Preference Analysis Agent**
   - Processes survey responses using NLP
   - Identifies patterns and conflicts
   - Generates preference_package for downstream agents

3. **Destination Research Agent**
   - Searches destinations using vector similarity
   - Validates visa requirements and travel advisories
   - Checks accessibility infrastructure

4. **Real-Time Pricing Agent**
   - Continuously monitors flight/hotel pricing
   - Detects price drops and trends
   - Alerts users of deals

5. **Voting Coordinator Agent**
   - Manages ranked-choice voting
   - Calculates consensus scores
   - Triggers conflict resolution if needed

6. **Conflict Resolution Agent**
   - Analyzes vote splits
   - Proposes compromise destinations
   - Suggests modified trip structures

7. **Itinerary Generation Agent**
   - Builds detailed day-by-day plans
   - Optimizes routing and timing
   - Accounts for accessibility needs

8. **Budget Optimization Agent**
   - Identifies cost-saving opportunities
   - Suggests quality alternatives within budget
   - Tracks spending against plan

9. **Verifier Agent**
   - Validates itinerary feasibility
   - Checks booking availability
   - Ensures accessibility requirements met


