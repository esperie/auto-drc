# User Flow: Receive Task Assignment

## Summary

This flow describes how a responder receives a task assignment from a commander or the AI routing system. The task appears as a notification, and the responder can accept, acknowledge, or decline with reason.

---

## Actor(s)

- **Primary**: Responder (medic, USAR, logistics)
- **Secondary**: Team Leader (also receives visibility)

---

## Trigger

- Task assigned by commander (manual)
- Task assigned by AI routing agent (automatic)
- Task reassigned from another responder

---

## Preconditions

1. Responder is connected to mesh network
2. Responder status is "available" or "engaged" (not "offline")
3. Device is not in Do Not Disturb mode (unless critical task)

---

## Main Flow

### 1. Task Notification

**Standard Priority (Medium/Low):**
```
┌────────────────────────────────────────┐
│ ADRC                           12:45   │
├────────────────────────────────────────┤
│                                        │
│  📋 New Task Assigned                  │
│                                        │
│  🔶 MEDIUM PRIORITY                    │
│                                        │
│  Type: Medical Assessment              │
│  Location: 350m NE                     │
│  From: SGT Tan (Sector A)              │
│                                        │
│     [View Details]  [Acknowledge]      │
│                                        │
└────────────────────────────────────────┘
```

**High/Critical Priority:**
```
┌────────────────────────────────────────┐
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
│░                                      ░│
│░  🔴 CRITICAL TASK                    ░│
│░                                      ░│
│░  Casualty Rescue                     ░│
│░  120m SW - Building collapse         ░│
│░                                      ░│
│░  Assigned by: CPT Lee                ░│
│░                                      ░│
│░      [ACKNOWLEDGE NOW]               ░│
│░                                      ░│
│░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░│
└────────────────────────────────────────┘
```
- Critical tasks take over screen
- Audio alert plays (configurable)
- Vibration pattern (configurable)
- Device screen wakes if locked

### 2. View Task Details
```
┌────────────────────────────────────────┐
│ ← Task Details                         │
├────────────────────────────────────────┤
│                                        │
│  🔶 Medical Assessment                 │
│     MEDIUM PRIORITY                    │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │       [MAP VIEW]                 │ │
│  │                                  │ │
│  │    📍 You        🎯 Task        │ │
│  │      ↓             ↓             │ │
│  │      •-------------•             │ │
│  │                    350m NE       │ │
│  └──────────────────────────────────┘ │
│                                        │
│  📝 Description:                       │
│  Elderly male, possible broken leg.    │
│  Conscious and responsive.             │
│  Building: Blk 123 #03-45              │
│                                        │
│  👤 Assigned by: SGT Tan Wei Ming     │
│  ⏱  12:45 (5 min ago)                 │
│                                        │
│  ┌────────────────────────────────┐   │
│  │       [Start Navigation]       │   │
│  └────────────────────────────────┘   │
│                                        │
│   [Accept]  [Decline with reason]     │
│                                        │
└────────────────────────────────────────┘
```
- Full task details displayed
- Map showing route to task location
- Description from assigner
- One-tap navigation option

### 3. Accept Task
```
┌────────────────────────────────────────┐
│                                        │
│  ✓ Task Accepted                       │
│                                        │
│  Your status: IN TRANSIT               │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │       [Start Navigation]         │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │       [Mark Arrived]             │ │
│  └──────────────────────────────────┘ │
│                                        │
└────────────────────────────────────────┘
```
- Confirmation shown
- Status auto-updates to "in_transit"
- Task appears in active tasks list
- Team leader/commander notified

### 4. Task In Progress

Once responder arrives and starts work:
```
┌────────────────────────────────────────┐
│ Active Task                    13:02   │
├────────────────────────────────────────┤
│                                        │
│  🔶 Medical Assessment                 │
│  ⏱ Duration: 12 min                   │
│                                        │
│  Quick Updates:                        │
│  ┌──────────────────────────────────┐ │
│  │ [On Scene] [Need Backup] [SOS]  │ │
│  └──────────────────────────────────┘ │
│                                        │
│  Notes:                                │
│  ┌──────────────────────────────────┐ │
│  │ Patient stable, splint applied  │ │
│  │ ┌──────┐                        │ │
│  │ │ 📷   │ [Add Note] [Voice]     │ │
│  │ └──────┘                        │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌────────────────────────────────────┐│
│  │         [COMPLETE TASK]           ││
│  └────────────────────────────────────┘│
│                                        │
│  [Request Resource] [Report Incident]  │
│                                        │
└────────────────────────────────────────┘
```
- Timer shows task duration
- Quick update buttons
- Note-taking capability (text, voice, photo)
- Complete task button prominent

---

## Alternative Flows

### A1: Decline Task
```
┌────────────────────────────────────────┐
│                                        │
│  Decline Task                          │
│                                        │
│  Please select a reason:               │
│                                        │
│  ○ Already on critical task            │
│  ○ Too far / inaccessible              │
│  ○ Lack required skills                │
│  ○ Equipment issue                     │
│  ○ Medical/personal emergency          │
│  ○ Other: ________________             │
│                                        │
│  Note (optional):                      │
│  ┌──────────────────────────────────┐ │
│  │                                  │ │
│  └──────────────────────────────────┘ │
│                                        │
│     [Cancel]  [Confirm Decline]        │
│                                        │
└────────────────────────────────────────┘
```
- Reason required for decline
- Task returns to AI routing for reassignment
- Commander notified of decline

### A2: Auto-Acknowledge Timeout
- If responder doesn't acknowledge within threshold:
  - Medium: 5 minutes
  - High: 2 minutes
  - Critical: 30 seconds
- Escalate to team leader
- Consider reassignment

### A3: Multiple Concurrent Tasks
```
┌────────────────────────────────────────┐
│                                        │
│  ⚠️ You have an active task            │
│                                        │
│  Current: Casualty Rescue (CRITICAL)   │
│                                        │
│  New task will be queued.              │
│                                        │
│     [Queue Task]  [View Both]          │
│                                        │
└────────────────────────────────────────┘
```
- Warn if already on task
- Option to queue new task
- Critical can preempt medium/low

---

## Exception Flows

### E1: Network Disconnection During Task
- Task data cached locally
- Status updates queued
- "Offline" indicator shown
- Auto-sync when reconnected

### E2: Task Cancelled by Commander
```
┌────────────────────────────────────────┐
│                                        │
│  ⚠️ Task Cancelled                     │
│                                        │
│  "Medical Assessment" has been         │
│  cancelled by SGT Tan.                 │
│                                        │
│  Reason: Duplicate report              │
│                                        │
│  Your status: Available                │
│                                        │
│           [OK]                         │
│                                        │
└────────────────────────────────────────┘
```
- Clear notification
- Status reverts to available
- Task removed from active list

### E3: Location Unavailable
- GPS failure notification
- Option to enter location manually
- Task can proceed with last known location

---

## Postconditions

1. Task assigned to responder
2. Responder status updated (in_transit → engaged)
3. Team leader has visibility
4. Commander has visibility
5. Task timeline begins

---

## Data Changes

### Task Record Updated
```json
{
  "id": "task-uuid-789",
  "status": "in_progress",
  "assigned_to": "responder-uuid-123",
  "assigned_at": "2024-01-15T12:45:00Z"
}
```

### Responder Status Updated
```json
{
  "id": "responder-uuid-123",
  "status": "engaged",
  "current_task_id": "task-uuid-789"
}
```

---

## Mesh Messages

### MSG: Task Assignment
```json
{
  "type": "task_assigned",
  "task_id": "task-uuid-789",
  "responder_id": "responder-uuid-123",
  "priority": "medium",
  "task_type": "medical",
  "location": {"lat": 1.3521, "lng": 103.8198},
  "description": "Elderly male, possible broken leg...",
  "assigned_by": "commander-uuid-456",
  "timestamp": "2024-01-15T12:45:00Z"
}
```

### MSG: Task Acknowledgment
```json
{
  "type": "task_acknowledged",
  "task_id": "task-uuid-789",
  "responder_id": "responder-uuid-123",
  "status": "accepted",
  "timestamp": "2024-01-15T12:47:00Z"
}
```

---

## AI Considerations

### Task Routing Agent Factors
When AI assigns tasks, it considers:
1. **Proximity**: Distance to task location
2. **Skills**: Match between task type and responder skills
3. **Workload**: Current task count and fatigue
4. **Team**: Keep teams together when possible
5. **History**: Past performance on similar tasks

### Override Capability
- Commanders can override AI assignments
- Override logged for analysis
- Feedback improves future routing
