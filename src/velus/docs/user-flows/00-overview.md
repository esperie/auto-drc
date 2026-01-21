# User Flows Overview - ADRC Platform

## Overview

This document provides an overview of the user flows in the ADRC (Autonomous Disaster Response Coordination) platform. User flows are organized by actor type and operational phase.

---

## 1. Actor Types

### 1.1 Responder

**Role**: Field personnel (medic, USAR, logistics) executing ground operations

**Primary Goals**:
- Receive and execute task assignments
- Report status and location
- Report incidents
- Request resources

**Device**: Rugged smartphone or tablet with ADRC mobile app

### 1.2 Team Leader

**Role**: First-line supervisor managing a small team (5-10 responders)

**Primary Goals**:
- Monitor team status and location
- Coordinate task execution
- Escalate issues
- Report team-level summaries

**Device**: Rugged tablet with ADRC mobile app (Team Leader mode)

### 1.3 Sector Commander

**Role**: Mid-level commander overseeing multiple teams in a geographic sector

**Primary Goals**:
- Maintain sector situational awareness
- Allocate resources across teams
- Create and assign tasks
- Report to operations center

**Device**: Tablet with ADRC mobile app (Commander mode)

### 1.4 Operations Center

**Role**: Central coordination hub managing overall response

**Primary Goals**:
- Full situational awareness
- Resource management
- Strategic decision-making
- External liaison

**Device**: Tablet/laptop with ADRC dashboard

---

## 2. User Flow Categories

### 2.1 Onboarding Flows
- [01-join-network.md](./01-join-network.md) - How users join the mesh network
- [02-role-selection.md](./02-role-selection.md) - Selecting operational role

### 2.2 Status Flows
- [03-status-update.md](./03-status-update.md) - Updating responder status
- [04-location-tracking.md](./04-location-tracking.md) - Automatic location updates

### 2.3 Task Flows
- [05-task-receive.md](./05-task-receive.md) - Receiving a task assignment
- [06-task-execute.md](./06-task-execute.md) - Executing and completing a task
- [07-task-create.md](./07-task-create.md) - Creating a new task (commander)
- [08-task-reassign.md](./08-task-reassign.md) - Reassigning a task

### 2.4 Incident Flows
- [09-incident-report.md](./09-incident-report.md) - Reporting a new incident
- [10-incident-verify.md](./10-incident-verify.md) - Verifying/updating an incident

### 2.5 Resource Flows
- [11-resource-request.md](./11-resource-request.md) - Requesting resources
- [12-resource-allocate.md](./12-resource-allocate.md) - Allocating resources

### 2.6 Situational Awareness Flows
- [13-situation-view.md](./13-situation-view.md) - Viewing situational dashboard
- [14-team-view.md](./14-team-view.md) - Viewing team status

---

## 3. Flow Matrix by Actor

| Flow | Responder | Team Lead | Sector Cmd | Ops Center |
|------|-----------|-----------|------------|------------|
| Join Network | ✓ | ✓ | ✓ | ✓ |
| Status Update | ✓ | ✓ | ✓ | - |
| Location Track | ✓ | ✓ | ✓ | - |
| Task Receive | ✓ | ✓ | - | - |
| Task Execute | ✓ | - | - | - |
| Task Create | - | ✓ | ✓ | ✓ |
| Task Reassign | - | ✓ | ✓ | ✓ |
| Incident Report | ✓ | ✓ | ✓ | - |
| Incident Verify | - | ✓ | ✓ | ✓ |
| Resource Request | ✓ | ✓ | - | - |
| Resource Allocate | - | - | ✓ | ✓ |
| Situation View | - | ✓ | ✓ | ✓ |
| Team View | - | ✓ | ✓ | ✓ |

---

## 4. Flow Notation

Each user flow document uses the following structure:

```markdown
# Flow Title

## Summary
Brief description of what the flow accomplishes.

## Actor(s)
Who performs this flow.

## Trigger
What initiates this flow.

## Preconditions
What must be true before the flow starts.

## Main Flow
1. Step 1
2. Step 2
   - Substep if needed
3. Step 3

## Alternative Flows
- Alternative A: Description
- Alternative B: Description

## Exception Flows
- Exception 1: What happens when X fails
- Exception 2: What happens when Y fails

## Postconditions
What is true after the flow completes.

## UI Mockup
[Link to mockup or ASCII diagram]

## Data Changes
- Model: field changes

## Mesh Messages
- Message type: description
```

---

## 5. Key Design Principles

### 5.1 One-Tap Primary Actions

The most common actions should require minimal taps:
- Status OK: 1 tap
- Task complete: 1 tap + confirmation
- SOS/Emergency: 1 tap (prominent)

### 5.2 Offline-First Design

All flows work without connectivity:
- Actions queue locally
- UI reflects pending state
- Sync happens automatically when mesh available

### 5.3 Minimal Input Required

- Pre-filled defaults based on context
- Voice input for hands-free
- Quick-select options over free text

### 5.4 Clear Feedback

- Immediate visual feedback on actions
- Delivery confirmation for critical messages
- Error states with recovery actions

---

## 6. PoC Scope

### 6.1 Flows Included in PoC

| Flow | Priority | Complexity |
|------|----------|------------|
| Join Network | P0 | Medium |
| Status Update | P0 | Low |
| Task Receive | P0 | Low |
| Task Execute | P0 | Medium |
| Task Create | P0 | Medium |
| Incident Report | P1 | Medium |
| Situation View | P1 | High |
| Team View | P1 | Medium |

### 6.2 Flows Deferred (Post-PoC)

- Resource allocation
- Incident verification workflow
- Task reassignment
- Advanced filtering/search
- Historical analysis
