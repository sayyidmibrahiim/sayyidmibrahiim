# Deployment Workflow Tracker

> A sanitized engineering case study of a Windows desktop application I designed to reduce manual tracking, coordination, and uncertainty in deployment workflows.

> **Privacy note:** This case study intentionally excludes company identity, internal URLs, credentials, infrastructure details, proprietary workflow specifics, employee information, production configuration, and internal screenshots. The implementation itself is not linked from this public case study.

---

## Why I Built This

The project started from a recurring operational problem.

Deployment work involved many moving parts:

- lifecycle stages
- deadlines
- approvals
- supporting files and evidence
- status changes
- communication through desktop productivity tools
- repetitive manual tracking

The information existed, but keeping everything synchronized required too much human memory and repetitive work.

The question that pushed the project forward was simple:

> **Can the workflow itself become a more reliable system instead of depending on people remembering every next action?**

That question gradually became a desktop application.

---

## My Role

I was responsible for shaping the project from problem to working system.

My contribution included:

- observing the existing workflow
- identifying repetitive and failure-prone steps
- defining expected application behavior
- mapping lifecycle states and transitions
- defining edge cases and failure conditions
- deciding which operations should and should not be automated
- designing the application workflow
- reviewing AI-generated implementations
- testing behavior against realistic operational scenarios
- debugging and refining the system iteratively

I used AI heavily during implementation.

I do not position myself as a traditional software developer who manually wrote every line of the application.

My strongest contribution was in:

**problem framing, workflow design, system behavior, constraints, verification, and iterative engineering.**

---

## What the System Became

At a high level, the application evolved into a Windows desktop workflow-management system.

Its architecture uses:

- **Python** for domain logic, services, persistence, and desktop-side orchestration
- **Svelte + TypeScript** for the user interface
- **pywebview** as the single frontend/backend bridge
- the **local filesystem as the canonical source of truth for project state**
- **SQLite for two distinct purposes:** rebuildable read/cache data and durable operational ledger data
- desktop integrations for supporting operational workflows

The application does not model a project as one flat status. Instead, it separates several related lifecycles so that transitions, guards, and side effects can be reasoned about explicitly.

For example, project-folder state, request state, sub-ticket state, and non-request project state are treated as separate concerns rather than collapsed into one generic status.

---

## Architecture Direction

As the application grew, one of the major changes was separating responsibilities that had accumulated inside a large application module.

The backend evolved toward a one-way dependency structure:

```text
Domain
   ↓
Infrastructure
   ↓
Services
   ↓
Adapters
   ↓
Application Composition
   ↓
Frontend
```

The purpose was practical rather than theoretical.

The separation made it easier to:

- understand ownership of behavior
- change one area without unintentionally affecting another
- test domain and service behavior independently
- isolate operating-system and integration concerns
- reason about failures and recovery

An earlier composition module had grown to roughly 2,600 lines and contained multiple adapter implementations. Those adapters were later extracted into dedicated modules, reducing the composition module to roughly 500 lines and making responsibilities more explicit.

The frontend does not call desktop APIs directly across the codebase. Calls are funneled through one bridge layer, and bridge responses use a consistent result/error shape.

---

## Filesystem as Operational Truth

A core design choice is that project state remains inspectable outside the application.

For request-based projects, folder location itself represents a lifecycle state. Moving a project to another state therefore means moving its folder, while project metadata is stored alongside the project.

That gives the system a useful property:

> the operational state is still visible in the filesystem even when the application is not open.

Metadata writes are performed atomically so an interrupted write does not silently replace a valid file with a half-written one.

SQLite is deliberately not treated as one undifferentiated database. Disposable dashboard/cache projections can be rebuilt from canonical sources, while operational records that must survive rebuilds are classified separately as durable ledger data.

---

## Reliability Matters

One of the most important lessons from this project was that automation is not only about reducing clicks.

It is also about controlling uncertainty.

Several design decisions were driven by that idea:

- explicit state-transition rules
- validation before critical actions
- keeping canonical state separate from rebuildable derived state
- protecting durable automation records from cache rebuilds
- avoiding blind retries when an external action may already have partially completed
- distinguishing confirmed failure from uncertain outcome
- rebuilding derived state from canonical sources when necessary
- preserving evidence before state-changing automation where the workflow requires it

A key principle became:

> **Automation should make failure easier to reason about, not harder.**

---

## Handling Uncertain External Actions

Desktop automation introduced a specific reliability problem: sometimes a call can time out even though the external application may still complete the action.

The application therefore distinguishes between two broad outcomes:

```text
failed
→ the action is known not to have completed

unknown
→ the caller timed out, but the external action may still complete
```

That distinction matters because automatically retrying an `unknown` result can duplicate a real-world side effect.

For desktop email integration, work is serialized through a bounded worker queue rather than allowing multiple concurrent COM operations to run freely.

This is a good example of how the project moved from simple automation toward explicit failure semantics.

---

## Backend-to-Frontend Events

The application also needs to push state changes from the backend toward the frontend.

Rather than relying on a fire-and-forget callback, the implementation uses a bounded, sequenced event buffer.

The frontend drains events using a cursor. If it falls so far behind that required events are no longer retained, the backend tells it to request a fresh snapshot rather than pretending the incomplete event history is valid.

This turns a potentially fragile UI synchronization mechanism into something with an explicit recovery path.

---

## Testing & Verification

As the project became more complex, the verification strategy also evolved.

The repository contains automated backend tests covering areas such as:

- application lifecycle behavior
- state transitions
- automation integrity
- approval and polling behavior
- cache/projection consistency
- migrations
- supporting services and bootstrap behavior

The frontend also has automated checks/tests, while visual behavior and some operating-system integrations still require manual verification because they depend on the desktop environment.

The project documentation itself has also been reviewed against the code. That process exposed stale or conflicting assumptions in older documents, which reinforced another rule I use when working with AI-assisted systems:

> **documentation, generated explanations, and old design decisions are evidence to verify, not authority to trust blindly.**

---

## How AI Was Used

AI was deeply involved in implementation, but I treated generated code as a proposed implementation rather than a finished answer.

My workflow usually looked like this:

```text
Observe the problem
        ↓
Map the current workflow
        ↓
Define desired behavior
        ↓
Define constraints and failure cases
        ↓
Plan the system
        ↓
Use AI to help implement
        ↓
Review the implementation
        ↓
Test the behavior
        ↓
Find incorrect assumptions
        ↓
Refine the requirements
        ↓
Fix / re-implement
        ↓
Verify again
```

As the project became larger, specification and verification became more important than simply generating more code.

This project also taught me that AI can accelerate implementation faster than architecture and operational reasoning can keep up. Once that happens, the important work becomes defining boundaries, finding contradictions, testing failure modes, and keeping the system understandable.

---

## What This Project Represents

For me, this project is less about a particular programming language or framework.

It represents the way I naturally approach work:

**I see a repetitive or fragile process, map how it actually works, identify where it can fail, and try to turn it into a more explicit and reliable system.**

AI makes it possible for me to implement ideas that would previously have been beyond my traditional programming ability.

But the part I care most about remains the same:

> **understanding the problem well enough to design a system that actually works.**

---

## Public / Private Boundary

This document is the public portfolio artifact for the project.

The original repository and implementation remain private because they contain company-specific operational context.

The public version intentionally avoids:

- company identity
- internal terminology that can identify systems or teams
- internal URLs
- credentials or secrets
- production infrastructure details
- proprietary workflow rules
- employee information
- internal screenshots
- production configuration

The goal is to show the engineering thinking without publishing company-specific implementation details.
