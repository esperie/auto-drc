# User Flow: Create Task (Commander)

## Summary

This flow describes how a team leader, sector commander, or operations center creates and assigns a new task. The flow includes AI-assisted responder recommendation and manual override capability.

---

## Actor(s)

- **Primary**: Team Leader, Sector Commander, Operations Center
- **Secondary**: AI Task Routing Agent

---

## Trigger

- Incident reported requiring response
- Proactive tasking based on situational assessment
- Resource redistribution decision
- Follow-up task from previous activity

---

## Preconditions

1. Commander is connected to mesh network
2. Commander has task creation privileges (role-based)
3. Available responders exist in network (for assignment)

---

## Main Flow

### 1. Initiate Task Creation

**From Dashboard:**
```
┌────────────────────────────────────────┐
│ Sector A Dashboard             13:15   │
├────────────────────────────────────────┤
│                                        │
│  [+ NEW TASK]  [📋 Tasks] [👥 Teams]  │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │         [MAP VIEW]               │ │
│  │                                  │ │
│  │    📍 Incident markers           │ │
│  │    👤 Responder locations        │ │
│  │                                  │ │
│  └──────────────────────────────────┘ │
│                                        │
│  Active Incidents: 5                   │
│  Available Responders: 12              │
│                                        │
└────────────────────────────────────────┘
```
- Tap "+ NEW TASK" button
- OR long-press on map to create task at location

### 2. Select Task Type
```
┌────────────────────────────────────────┐
│ ← Create Task                          │
├────────────────────────────────────────┤
│                                        │
│  Select Task Type:                     │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ 🚨 Rescue                        │ │
│  │    Extract casualties from site  │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🏥 Medical                       │ │
│  │    Medical assessment or care    │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🔍 Reconnaissance                │ │
│  │    Survey area or structure      │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 📦 Logistics                     │ │
│  │    Move or deliver resources     │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🛡️ Security                     │ │
│  │    Secure area or escort         │ │
│  └──────────────────────────────────┘ │
│                                        │
└────────────────────────────────────────┘
```
- Pre-defined task types with descriptions
- Selection filters available responders by skill match

### 3. Set Priority
```
┌────────────────────────────────────────┐
│ ← Create Task                          │
├────────────────────────────────────────┤
│                                        │
│  Task Type: 🏥 Medical                 │
│                                        │
│  Select Priority:                      │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ 🔴 CRITICAL                      │ │
│  │    Life-threatening, immediate   │ │
│  │    Response: < 5 min             │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🟠 HIGH                          │ │
│  │    Urgent, significant impact    │ │
│  │    Response: < 15 min            │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🟡 MEDIUM                        │ │
│  │    Important, can wait briefly   │ │
│  │    Response: < 30 min            │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🟢 LOW                           │ │
│  │    Routine, schedule when able   │ │
│  │    Response: < 2 hours           │ │
│  └──────────────────────────────────┘ │
│                                        │
└────────────────────────────────────────┘
```
- Clear priority definitions
- Response time expectations shown
- Priority affects AI routing urgency

### 4. Set Location
```
┌────────────────────────────────────────┐
│ ← Create Task                          │
├────────────────────────────────────────┤
│                                        │
│  Task: 🏥 Medical | 🟠 HIGH            │
│                                        │
│  Set Location:                         │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │         [MAP VIEW]               │ │
│  │                                  │ │
│  │    Tap to place marker           │ │
│  │                                  │ │
│  │         📍                       │ │
│  │                                  │ │
│  │                                  │ │
│  └──────────────────────────────────┘ │
│                                        │
│  OR enter address/landmark:            │
│  ┌──────────────────────────────────┐ │
│  │ Blk 123 Ang Mo Kio Ave 3         │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ○ Use current location                │
│  ○ Use incident location (if linked)   │
│                                        │
│              [Next →]                  │
│                                        │
└────────────────────────────────────────┘
```
- Interactive map for precise location
- Text entry for address/landmark
- Quick options for common locations
- Link to existing incident if applicable

### 5. Add Description
```
┌────────────────────────────────────────┐
│ ← Create Task                          │
├────────────────────────────────────────┤
│                                        │
│  Task: 🏥 Medical | 🟠 HIGH            │
│  📍 Blk 123 Ang Mo Kio Ave 3          │
│                                        │
│  Description:                          │
│  ┌──────────────────────────────────┐ │
│  │ Elderly female, ~70 years old.   │ │
│  │ Found conscious but disoriented. │ │
│  │ Possible head injury from fall.  │ │
│  │ Located at level 3, unit 45.     │ │
│  │                                  │ │
│  │                                  │ │
│  └──────────────────────────────────┘ │
│                                        │
│  🎤 [Voice Input]                     │
│                                        │
│  Required Skills: (optional)           │
│  ┌──────────────────────────────────┐ │
│  │ [✓ Trauma] [ First Aid] [ CPR]  │ │
│  └──────────────────────────────────┘ │
│                                        │
│              [Next →]                  │
│                                        │
└────────────────────────────────────────┘
```
- Free text description
- Voice input option for hands-free
- Optional skill tags for better AI matching
- Structured data capture where possible

### 6. AI Responder Recommendation
```
┌────────────────────────────────────────┐
│ ← Create Task                          │
├────────────────────────────────────────┤
│                                        │
│  🤖 AI Recommendation                  │
│                                        │
│  Based on skills, proximity, and       │
│  workload, recommended responder:      │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ ⭐ RECOMMENDED                   │ │
│  │                                  │ │
│  │ 👤 SGT Lim Mei Ling             │ │
│  │    Medic | Alpha Team            │ │
│  │    📍 180m away | Available      │ │
│  │    Skills: Trauma ✓, CPR ✓       │ │
│  │                                  │ │
│  │    [SELECT]                      │ │
│  └──────────────────────────────────┘ │
│                                        │
│  Other options:                        │
│  ┌──────────────────────────────────┐ │
│  │ 👤 CPL Tan Wei | 320m | Avail   │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 👤 PTE Chen Yu | 450m | Engaged │ │
│  └──────────────────────────────────┘ │
│                                        │
│  [Manual Selection] [Assign to Team]   │
│                                        │
└────────────────────────────────────────┘
```
- AI recommendation prominently displayed
- Reasoning visible (proximity, skills)
- Alternative options shown
- Manual override available

### 7. Confirm & Create
```
┌────────────────────────────────────────┐
│ ← Create Task                          │
├────────────────────────────────────────┤
│                                        │
│  Review Task:                          │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ Type: 🏥 Medical                 │ │
│  │ Priority: 🟠 HIGH                │ │
│  │ Location: Blk 123 AMK Ave 3      │ │
│  │                                  │ │
│  │ Description:                     │ │
│  │ Elderly female, ~70 years old... │ │
│  │                                  │ │
│  │ Assigned to:                     │ │
│  │ 👤 SGT Lim Mei Ling             │ │
│  │    (AI Recommended)              │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌────────────────────────────────────┐│
│  │       [CREATE & ASSIGN TASK]      ││
│  └────────────────────────────────────┘│
│                                        │
│            [Save as Draft]             │
│                                        │
└────────────────────────────────────────┘
```
- Full summary for review
- Single tap to create and assign
- Draft option for later completion

### 8. Task Created Confirmation
```
┌────────────────────────────────────────┐
│                                        │
│  ✓ Task Created                        │
│                                        │
│  Task ID: T-2024-0115-042             │
│                                        │
│  SGT Lim Mei Ling has been notified.  │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │    [Track Task]  [Create Another]│ │
│  └──────────────────────────────────┘ │
│                                        │
│            [Return to Dashboard]       │
│                                        │
└────────────────────────────────────────┘
```
- Confirmation with task ID
- Notification sent to responder
- Quick actions available

---

## Alternative Flows

### A1: Manual Responder Selection
```
┌────────────────────────────────────────┐
│ Select Responder                       │
├────────────────────────────────────────┤
│                                        │
│  Filter: [All] [Medic] [USAR] [Logi]  │
│  Sort: [Nearest] [Available] [Team]    │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ 👤 SGT Lim Mei Ling             │ │
│  │    Medic | 180m | Available ✓    │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 👤 CPL Tan Wei Hong              │ │
│  │    Medic | 320m | Available ✓    │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 👤 PTE Chen Yu Wei               │ │
│  │    USAR | 450m | Engaged ⚠️      │ │
│  └──────────────────────────────────┘ │
│                                        │
└────────────────────────────────────────┘
```
- Override AI recommendation
- Filter and sort responders
- See all available personnel
- Log override reason (optional)

### A2: Assign to Team (Not Individual)
- Select team instead of individual
- Team leader assigns within team
- Useful when commander lacks granular visibility

### A3: Quick Task (Minimal Input)
```
┌────────────────────────────────────────┐
│                                        │
│  ⚡ Quick Task                         │
│                                        │
│  Type: [Rescue ▼]                     │
│  Priority: [🔴 Critical]              │
│  To: [Nearest Available]              │
│                                        │
│  📍 Using your current location       │
│                                        │
│        [CREATE NOW]                    │
│                                        │
└────────────────────────────────────────┘
```
- For urgent situations
- Minimal input required
- Uses defaults and current location
- AI selects best responder automatically

### A4: Task from Incident
- Link task to existing incident
- Pre-populate location and context
- Track all tasks related to incident

---

## Exception Flows

### E1: No Available Responders
```
┌────────────────────────────────────────┐
│                                        │
│  ⚠️ No Responders Available            │
│                                        │
│  All responders are currently          │
│  engaged or offline.                   │
│                                        │
│  Options:                              │
│  • Create task as unassigned           │
│  • Request resources from other sector │
│  • Reassign from lower priority task   │
│                                        │
│  [Create Unassigned] [Request Aid]     │
│                                        │
└────────────────────────────────────────┘
```
- Clear explanation
- Actionable alternatives
- Escalation path

### E2: Network Disconnection
- Task saved locally
- Queued for mesh broadcast
- Confirmation shows "Pending sync"
- Auto-sends when reconnected

### E3: Responder Declines
- Commander notified immediately
- Decline reason shown
- Option to reassign or modify task

---

## Postconditions

1. Task record created in system
2. Task assigned to responder
3. Responder notified via mesh
4. Task visible on commander dashboard
5. Team leader has visibility (if team member assigned)
6. Audit trail created

---

## Data Changes

### Task Record Created
```json
{
  "id": "task-uuid-999",
  "type": "medical",
  "priority": "high",
  "status": "assigned",
  "location_lat": 1.3689,
  "location_lng": 103.8492,
  "description": "Elderly female, ~70 years old. Found conscious but disoriented...",
  "assigned_to": "responder-uuid-456",
  "assigned_by": "commander-uuid-123",
  "assigned_at": "2024-01-15T13:20:00Z",
  "ai_recommended": true,
  "skills_required": ["trauma", "cpr"]
}
```

---

## Mesh Messages

### MSG: Task Created
```json
{
  "type": "task_created",
  "task_id": "task-uuid-999",
  "task_type": "medical",
  "priority": "high",
  "location": {"lat": 1.3689, "lng": 103.8492},
  "assigned_to": "responder-uuid-456",
  "assigned_by": "commander-uuid-123",
  "timestamp": "2024-01-15T13:20:00Z"
}
```
- Broadcast to all relevant parties
- Updates situational displays

### MSG: Task Assignment Notification
```json
{
  "type": "task_assigned",
  "task_id": "task-uuid-999",
  "responder_id": "responder-uuid-456",
  "priority": "high",
  "task_type": "medical",
  "location": {"lat": 1.3689, "lng": 103.8492},
  "description": "Elderly female, ~70 years old...",
  "assigned_by": "commander-uuid-123",
  "timestamp": "2024-01-15T13:20:00Z"
}
```
- Targeted to assigned responder
- Triggers notification on device

---

## AI Agent Interaction

### Task Router Agent Called
When task is created with AI assignment:

```python
# Simplified agent call
recommendation = await task_router.route_task(
    task_type="medical",
    priority="high",
    location=(1.3689, 103.8492),
    required_skills=["trauma", "cpr"]
)

# Returns:
{
    "responder_id": "responder-uuid-456",
    "reason": "Closest available medic with trauma skills",
    "confidence": 0.92,
    "alternatives": [
        {"id": "responder-uuid-789", "reason": "Farther but more experience"},
        {"id": "responder-uuid-012", "reason": "Available soon (current task completing)"}
    ]
}
```

### Learning from Overrides
When commander overrides AI:
- Log override with reason
- Feed back to improve model
- Track override patterns
