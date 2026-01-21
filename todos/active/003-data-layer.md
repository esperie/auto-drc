# TODO-020 to TODO-023: Data Layer Phase (Week 5-6)

**Phase**: 3 - Data Layer
**Timeline**: Week 5-6
**Status**: PENDING

**CRITICAL CORRECTIONS FROM DATAFLOW SPECIALIST**:
1. Template syntax: Use `${}` NOT `{{}}`
2. NEVER manually set `created_at` or `updated_at` (auto-managed)
3. Use `add_connection()` NOT `connect()`
4. Add `auto_discovery=False` when using Nexus + DataFlow together

---

## TODO-020: DataFlow Models Implementation

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-001 (Project Setup)

### Description

Implement DataFlow models for all ADRC entities. DataFlow automatically generates 11 CRUD nodes per model.

### Acceptance Criteria

- [ ] Responder model with all required fields
- [ ] Task model with all required fields
- [ ] Incident model with all required fields
- [ ] Resource model with all required fields
- [ ] SQLite database configured
- [ ] All models have proper validation
- [ ] **No `created_at`/`updated_at` in model definitions**

### Critical DataFlow Rules

```python
# CORRECT: DataFlow auto-manages timestamps
@db.model
class Responder:
    id: str                    # MUST be named 'id' (not user_id, responder_id)
    name: str
    # ... other fields
    # DO NOT include: created_at, updated_at

# WRONG: Never manually set timestamps
@db.model
class Responder:
    id: str
    created_at: str  # WRONG - Remove this!
    updated_at: str  # WRONG - Remove this!
```

### Model Definitions

```python
# src/velus/models/responder.py
from dataflow import DataFlow
from typing import Optional

db = DataFlow("sqlite:///adrc.db")

@db.model
class Responder:
    id: str                    # Device UUID (primary key)
    name: str
    role: str                  # medic, usar, logistics, command
    team_id: str
    agency: str                # scdf, saf, redcross, etc.
    status: str                # available, engaged, offline
    location_lat: float
    location_lng: float
    location_updated_at: str   # ISO timestamp (manual, not auto)
    skills: str                # JSON array ["cpr", "hazmat", ...]
    battery_level: int
    last_seen_at: str          # Manual timestamp


@db.model
class Task:
    id: str                    # UUID
    type: str                  # rescue, medical, logistics, recon
    priority: str              # critical, high, medium, low
    status: str                # pending, assigned, in_progress, completed, cancelled
    location_lat: float
    location_lng: float
    description: str
    assigned_to: Optional[str] = None    # Responder ID
    assigned_by: Optional[str] = None    # Commander ID
    assigned_at: Optional[str] = None    # Manual timestamp
    completed_at: Optional[str] = None   # Manual timestamp
    outcome: Optional[str] = None
    notes: Optional[str] = None


@db.model
class Incident:
    id: str                    # UUID
    type: str                  # casualty, hazard, structure_collapse, etc.
    severity: str              # critical, serious, moderate, minor
    status: str                # active, resolved, false_alarm
    location_lat: float
    location_lng: float
    description: str
    reported_by: str           # Responder ID
    reported_at: str           # Manual timestamp
    verified: bool = False
    verified_by: Optional[str] = None
    responder_count: int = 0   # Estimated needed


@db.model
class Resource:
    id: str                    # UUID
    type: str                  # vehicle, medical, equipment, personnel
    subtype: str               # ambulance, stretcher, radio, etc.
    status: str                # available, deployed, depleted, damaged
    location_lat: float
    location_lng: float
    quantity: int
    assigned_to: Optional[str] = None    # Task or team ID
    notes: Optional[str] = None
```

### Tasks

1. [ ] Create `src/velus/models/__init__.py` with DataFlow instance (1h)
2. [ ] Create `src/velus/models/responder.py` (2h)
3. [ ] Create `src/velus/models/task.py` (2h)
4. [ ] Create `src/velus/models/incident.py` (2h)
5. [ ] Create `src/velus/models/resource.py` (2h)
6. [ ] Write model tests (3h)
7. [ ] Test auto-generated nodes work correctly (2h)

### Auto-Generated Nodes (per model)

For `Responder`, DataFlow generates:
- `ResponderCreateNode` - Create responder
- `ResponderReadNode` - Read by ID
- `ResponderUpdateNode` - Update fields
- `ResponderDeleteNode` - Delete
- `ResponderListNode` - Query with filters
- `ResponderCountNode` - Count matching
- `ResponderUpsertNode` - Insert or update
- `ResponderBulkCreateNode` - Batch create
- `ResponderBulkUpdateNode` - Batch update
- `ResponderBulkDeleteNode` - Batch delete
- `ResponderBulkUpsertNode` - Batch upsert

### Testing Requirements

- [ ] All models defined without errors
- [ ] Auto-generated nodes accessible
- [ ] CRUD operations work via express API
- [ ] **No DF-104 errors** (timestamp conflicts)

### Risk Assessment

- **LOW**: DataFlow is well-documented
- **MEDIUM**: Timestamp handling errors if rules not followed

### Definition of Done

- [ ] All 4 models implemented
- [ ] All tests pass
- [ ] Express API works for all CRUD operations

---

## TODO-021: Core Workflows (Kailash SDK)

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 4 days
**Dependencies**: TODO-020 (DataFlow Models)

### Description

Implement core Kailash SDK workflows for ADRC operations. These workflows orchestrate DataFlow operations and prepare data for mesh broadcast.

### Acceptance Criteria

- [ ] Status Update Workflow
- [ ] Task Assignment Workflow
- [ ] Incident Report Workflow
- [ ] Resource Query Workflow
- [ ] All workflows use correct template syntax (`${}`)
- [ ] All workflows use `add_connection()` not `connect()`

### Critical Workflow Corrections

```python
# CORRECT: Use ${} template syntax
workflow.add_node("ResponderUpdateNode", "update_status", {
    "filter": {"id": "${inputs.responder_id}"},  # CORRECT
    "fields": {
        "status": "${inputs.status}",            # CORRECT
    }
})

# WRONG: Using {{}} template syntax
workflow.add_node("ResponderUpdateNode", "update_status", {
    "filter": {"id": "{{inputs.responder_id}}"},  # WRONG!
    "fields": {
        "status": "{{inputs.status}}",            # WRONG!
    }
})

# CORRECT: Use add_connection()
workflow.add_connection("update_status", "prepare_broadcast")  # CORRECT

# WRONG: Using connect()
workflow.connect("update_status", "prepare_broadcast")  # WRONG!
```

### Workflow Implementations

#### 1. Status Update Workflow

```python
# src/velus/workflows/status_update.py
from kailash.workflow.builder import WorkflowBuilder
from kailash.runtime import AsyncLocalRuntime
from datetime import datetime, UTC

def create_status_update_workflow():
    workflow = WorkflowBuilder()

    # Update responder status in database
    workflow.add_node("ResponderUpdateNode", "update_status", {
        "filter": {"id": "${inputs.responder_id}"},
        "fields": {
            "status": "${inputs.status}",
            "location_lat": "${inputs.location_lat}",
            "location_lng": "${inputs.location_lng}",
            "location_updated_at": "${inputs.timestamp}",
            "last_seen_at": "${inputs.timestamp}"
        }
    })

    # Prepare broadcast message
    workflow.add_node("PythonCode", "prepare_broadcast", {
        "code": """
output = {
    "type": "status_update",
    "responder_id": inputs["responder_id"],
    "status": inputs["status"],
    "location": {
        "lat": inputs["location_lat"],
        "lng": inputs["location_lng"]
    },
    "timestamp": inputs["timestamp"]
}
"""
    })
    workflow.add_connection("update_status", "prepare_broadcast")  # CORRECT!

    return workflow
```

#### 2. Task Assignment Workflow

```python
# src/velus/workflows/task_assignment.py
from kailash.workflow.builder import WorkflowBuilder

def create_task_assignment_workflow():
    workflow = WorkflowBuilder()

    # Create task (flat fields for CreateNode)
    workflow.add_node("TaskCreateNode", "create_task", {
        "id": "${inputs.task_id}",
        "type": "${inputs.task_type}",
        "priority": "${inputs.priority}",
        "status": "pending",
        "location_lat": "${inputs.location_lat}",
        "location_lng": "${inputs.location_lng}",
        "description": "${inputs.description}",
        "assigned_by": "${inputs.commander_id}"
    })

    # Note: AI agent called OUTSIDE workflow (see TODO-034)
    # Workflow receives responder_id from caller

    # Assign to responder (UpdateNode uses filter + fields)
    workflow.add_node("TaskUpdateNode", "assign_task", {
        "filter": {"id": "${inputs.task_id}"},
        "fields": {
            "status": "assigned",
            "assigned_to": "${inputs.responder_id}",
            "assigned_at": "${inputs.timestamp}"
        }
    })
    workflow.add_connection("create_task", "assign_task")

    # Prepare notification
    workflow.add_node("PythonCode", "prepare_notification", {
        "code": """
output = {
    "type": "task_assignment",
    "task_id": inputs["task_id"],
    "task_type": inputs["task_type"],
    "priority": inputs["priority"],
    "assigned_to": inputs["responder_id"],
    "location": {
        "lat": inputs["location_lat"],
        "lng": inputs["location_lng"]
    },
    "description": inputs["description"]
}
"""
    })
    workflow.add_connection("assign_task", "prepare_notification")

    return workflow
```

#### 3. Incident Report Workflow

```python
# src/velus/workflows/incident_report.py
from kailash.workflow.builder import WorkflowBuilder

def create_incident_report_workflow():
    workflow = WorkflowBuilder()

    # Create incident
    workflow.add_node("IncidentCreateNode", "create_incident", {
        "id": "${inputs.incident_id}",
        "type": "${inputs.incident_type}",
        "severity": "${inputs.severity}",
        "status": "active",
        "location_lat": "${inputs.location_lat}",
        "location_lng": "${inputs.location_lng}",
        "description": "${inputs.description}",
        "reported_by": "${inputs.reporter_id}",
        "reported_at": "${inputs.timestamp}",
        "responder_count": "${inputs.responder_count}"
    })

    # Prepare broadcast
    workflow.add_node("PythonCode", "prepare_broadcast", {
        "code": """
output = {
    "type": "incident_report",
    "incident_id": inputs["incident_id"],
    "incident_type": inputs["incident_type"],
    "severity": inputs["severity"],
    "location": {
        "lat": inputs["location_lat"],
        "lng": inputs["location_lng"]
    },
    "description": inputs["description"],
    "responders_needed": inputs["responder_count"]
}
"""
    })
    workflow.add_connection("create_incident", "prepare_broadcast")

    return workflow
```

#### 4. Resource Query Workflow

```python
# src/velus/workflows/resource_query.py
from kailash.workflow.builder import WorkflowBuilder

def create_resource_query_workflow():
    workflow = WorkflowBuilder()

    # Query available resources
    workflow.add_node("ResourceListNode", "list_resources", {
        "filter": {
            "type": "${inputs.resource_type}",
            "status": "available"
        }
    })

    # Sort by proximity
    workflow.add_node("PythonCode", "sort_by_proximity", {
        "code": """
from math import radians, cos, sin, asin, sqrt

def haversine(lat1, lon1, lat2, lon2):
    lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])
    dlat = lat2 - lat1
    dlon = lon2 - lon1
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    return 2 * 6371 * asin(sqrt(a))

resources = inputs["list_resources"]["records"]
target_lat = inputs["location_lat"]
target_lng = inputs["location_lng"]

for r in resources:
    r["distance_km"] = haversine(
        target_lat, target_lng,
        r["location_lat"], r["location_lng"]
    )

resources.sort(key=lambda x: x["distance_km"])
output = {"resources": resources}
"""
    })
    workflow.add_connection("list_resources", "sort_by_proximity")

    return workflow
```

### Tasks

1. [ ] Create `src/velus/workflows/__init__.py` (30m)
2. [ ] Implement `status_update.py` (2h)
3. [ ] Implement `task_assignment.py` (3h)
4. [ ] Implement `incident_report.py` (2h)
5. [ ] Implement `resource_query.py` (2h)
6. [ ] Write workflow tests (4h)
7. [ ] Integration test with DataFlow (2h)

### Testing Requirements

- [ ] All workflows execute without errors
- [ ] Template parameters resolve correctly
- [ ] Connections work as expected
- [ ] Database updated correctly
- [ ] Broadcast messages formatted correctly

### Risk Assessment

- **MEDIUM**: Template syntax errors can cause runtime failures
- **Mitigation**: Thorough testing of each workflow

### Definition of Done

- [ ] All 4 workflows implemented
- [ ] All tests pass
- [ ] Integration with DataFlow verified

---

## TODO-022: CRDT Sync Engine

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 4 days
**Dependencies**: TODO-020 (DataFlow Models)

### Description

Implement CRDT-based synchronization engine for conflict-free data replication across mesh nodes.

### Acceptance Criteria

- [ ] Vector clock implementation
- [ ] Last-Writer-Wins (LWW) Register for simple values
- [ ] Grow-only Set for append-only data (incidents)
- [ ] OR-Set for add/remove operations (task assignments)
- [ ] Sync protocol for state exchange
- [ ] Conflict resolution rules documented

### CRDT Types

```python
# src/velus/sync/crdt_types.py
from dataclasses import dataclass, field
from typing import Dict, Any, Set
import time

@dataclass
class VectorClock:
    """Vector clock for causal ordering."""
    clock: Dict[str, int] = field(default_factory=dict)

    def increment(self, node_id: str):
        self.clock[node_id] = self.clock.get(node_id, 0) + 1

    def merge(self, other: 'VectorClock'):
        for node_id, count in other.clock.items():
            self.clock[node_id] = max(self.clock.get(node_id, 0), count)

    def happens_before(self, other: 'VectorClock') -> bool:
        """Returns True if self causally precedes other."""
        all_leq = all(
            self.clock.get(k, 0) <= other.clock.get(k, 0)
            for k in set(self.clock) | set(other.clock)
        )
        any_lt = any(
            self.clock.get(k, 0) < other.clock.get(k, 0)
            for k in set(self.clock) | set(other.clock)
        )
        return all_leq and any_lt

    def concurrent(self, other: 'VectorClock') -> bool:
        """Returns True if neither happens before the other."""
        return not self.happens_before(other) and not other.happens_before(self)


@dataclass
class LWWRegister:
    """Last-Writer-Wins Register for simple values."""
    value: Any = None
    timestamp: float = 0.0
    node_id: str = ""

    def update(self, new_value: Any, timestamp: float, node_id: str) -> bool:
        """Update if timestamp is newer. Returns True if updated."""
        if timestamp > self.timestamp or (
            timestamp == self.timestamp and node_id > self.node_id
        ):
            self.value = new_value
            self.timestamp = timestamp
            self.node_id = node_id
            return True
        return False

    def merge(self, other: 'LWWRegister') -> bool:
        """Merge with another register. Returns True if changed."""
        return self.update(other.value, other.timestamp, other.node_id)


@dataclass
class GrowOnlySet:
    """Grow-only set (G-Set) for append-only data."""
    items: Set[str] = field(default_factory=set)

    def add(self, item: str):
        self.items.add(item)

    def contains(self, item: str) -> bool:
        return item in self.items

    def merge(self, other: 'GrowOnlySet'):
        self.items |= other.items

    def elements(self) -> Set[str]:
        return self.items.copy()


@dataclass
class ORSet:
    """Observed-Remove Set for add/remove operations."""
    adds: Dict[str, Set[str]] = field(default_factory=dict)
    removes: Dict[str, Set[str]] = field(default_factory=dict)

    def add(self, element: str, node_id: str):
        tag = f"{node_id}:{time.time()}"
        if element not in self.adds:
            self.adds[element] = set()
        self.adds[element].add(tag)

    def remove(self, element: str):
        if element in self.adds:
            if element not in self.removes:
                self.removes[element] = set()
            self.removes[element] |= self.adds[element]

    def contains(self, element: str) -> bool:
        if element not in self.adds:
            return False
        add_tags = self.adds[element]
        remove_tags = self.removes.get(element, set())
        return bool(add_tags - remove_tags)

    def elements(self) -> Set[str]:
        result = set()
        for element, add_tags in self.adds.items():
            remove_tags = self.removes.get(element, set())
            if add_tags - remove_tags:
                result.add(element)
        return result

    def merge(self, other: 'ORSet'):
        for element, tags in other.adds.items():
            if element not in self.adds:
                self.adds[element] = set()
            self.adds[element] |= tags
        for element, tags in other.removes.items():
            if element not in self.removes:
                self.removes[element] = set()
            self.removes[element] |= tags
```

### Sync Engine

```python
# src/velus/sync/sync_engine.py
from dataclasses import dataclass, field
from typing import Dict, List, Optional
import time
from .crdt_types import VectorClock, LWWRegister

@dataclass
class SyncUpdate:
    """Represents a single update to sync."""
    model: str
    record_id: str
    fields: Dict[str, Any]
    clock: Dict[str, int]
    timestamp: float
    node_id: str


class SyncEngine:
    """CRDT-based synchronization engine."""

    def __init__(self, node_id: str, db):
        self.node_id = node_id
        self.db = db
        self.vector_clock = VectorClock()
        self.pending_updates: List[SyncUpdate] = []
        self.seen_updates: Dict[str, float] = {}  # update_key -> timestamp
        self.max_pending = 1000

    async def local_update(
        self,
        model: str,
        record_id: str,
        fields: Dict[str, Any]
    ) -> SyncUpdate:
        """Record a local update for sync."""
        self.vector_clock.increment(self.node_id)
        timestamp = time.time()

        update = SyncUpdate(
            model=model,
            record_id=record_id,
            fields=fields,
            clock=dict(self.vector_clock.clock),
            timestamp=timestamp,
            node_id=self.node_id
        )

        self.pending_updates.append(update)
        self._trim_pending()

        return update

    async def receive_updates(self, updates: List[SyncUpdate]) -> int:
        """Process updates from other nodes. Returns count applied."""
        applied = 0
        for update in updates:
            update_key = f"{update.model}:{update.record_id}:{update.timestamp}"

            # Skip if already seen
            if update_key in self.seen_updates:
                continue

            # Apply update
            if await self._apply_update(update):
                applied += 1

            # Merge vector clock
            remote_clock = VectorClock(update.clock)
            self.vector_clock.merge(remote_clock)

            # Mark as seen
            self.seen_updates[update_key] = time.time()
            self._trim_seen()

        return applied

    async def _apply_update(self, update: SyncUpdate) -> bool:
        """Apply a remote update using LWW conflict resolution."""
        try:
            # Read current record
            current = await self.db.express.read(update.model, update.record_id)

            if current is None:
                # Record doesn't exist, create it
                await self.db.express.create(update.model, {
                    "id": update.record_id,
                    **update.fields
                })
                return True

            # LWW: Only update if remote is newer
            # Use location_updated_at or last_seen_at as version
            current_ts = current.get("location_updated_at") or current.get("last_seen_at") or "0"
            update_ts = update.fields.get("location_updated_at") or update.fields.get("last_seen_at") or "0"

            if update_ts > current_ts:
                await self.db.express.update(update.model, update.record_id, update.fields)
                return True

            return False
        except Exception as e:
            print(f"Sync apply error: {e}")
            return False

    def get_sync_state(self) -> Dict:
        """Get current sync state for exchange."""
        return {
            "clock": dict(self.vector_clock.clock),
            "pending": [
                {
                    "model": u.model,
                    "record_id": u.record_id,
                    "fields": u.fields,
                    "clock": u.clock,
                    "timestamp": u.timestamp,
                    "node_id": u.node_id
                }
                for u in self.pending_updates[-100:]
            ]
        }

    def clear_pending(self, before_timestamp: Optional[float] = None):
        """Clear pending updates, optionally before a timestamp."""
        if before_timestamp:
            self.pending_updates = [
                u for u in self.pending_updates
                if u.timestamp >= before_timestamp
            ]
        else:
            self.pending_updates = []

    def _trim_pending(self):
        """Keep pending list bounded."""
        if len(self.pending_updates) > self.max_pending:
            self.pending_updates = self.pending_updates[-self.max_pending:]

    def _trim_seen(self):
        """Remove old entries from seen cache."""
        cutoff = time.time() - 300  # 5 minutes
        self.seen_updates = {
            k: v for k, v in self.seen_updates.items()
            if v > cutoff
        }
```

### Tasks

1. [ ] Implement CRDT types in `crdt_types.py` (4h)
2. [ ] Implement `SyncEngine` class (6h)
3. [ ] Write CRDT unit tests (4h)
4. [ ] Write sync engine tests (4h)
5. [ ] Test conflict resolution scenarios (4h)
6. [ ] Document conflict resolution rules (2h)

### Conflict Resolution Rules

| Data Type | Strategy | Example |
|-----------|----------|---------|
| Responder status | LWW by timestamp | Later status wins |
| Responder location | LWW by timestamp | Later location wins |
| Task assignment | LWW by assigned_at | Later assignment wins |
| Incident reports | G-Set (append only) | All reports kept |
| Resource allocation | OR-Set | Can add and remove |

### Testing Requirements

- [ ] Vector clock increment/merge works
- [ ] LWW register resolves concurrent updates
- [ ] G-Set union is correct
- [ ] OR-Set add/remove works correctly
- [ ] Sync engine applies updates correctly
- [ ] Duplicate updates are ignored

### Risk Assessment

- **MEDIUM**: CRDT implementation is nuanced
- **Mitigation**: Use well-tested patterns, thorough testing

### Definition of Done

- [ ] All CRDT types implemented and tested
- [ ] Sync engine functional
- [ ] Conflict resolution documented and tested

---

## TODO-023: Mesh-to-DataFlow Bridge

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-021 (Workflows), TODO-012 (Mesh Protocol)

### Description

Create the bridge layer that connects BLE mesh messages to DataFlow operations and sync engine.

### Acceptance Criteria

- [ ] Receive mesh messages and route to appropriate handler
- [ ] Convert mesh messages to DataFlow operations
- [ ] Trigger sync when relevant messages received
- [ ] Queue operations for batch processing
- [ ] Handle message types: status, task, incident, resource, sync

### Message Handler

```python
# src/velus/mesh/message_handler.py
from typing import Dict, Any, Callable
from velus.sync.sync_engine import SyncEngine
from velus.workflows import (
    create_status_update_workflow,
    create_task_assignment_workflow,
    create_incident_report_workflow
)
from kailash.runtime import AsyncLocalRuntime

class MeshMessageHandler:
    """Handles incoming mesh messages and routes to appropriate operations."""

    def __init__(self, db, sync_engine: SyncEngine):
        self.db = db
        self.sync_engine = sync_engine
        self.runtime = AsyncLocalRuntime()
        self.handlers: Dict[str, Callable] = {
            "status_update": self._handle_status_update,
            "task_assignment": self._handle_task_assignment,
            "incident_report": self._handle_incident_report,
            "resource_update": self._handle_resource_update,
            "sync_request": self._handle_sync_request,
            "sync_response": self._handle_sync_response,
        }

    async def handle_message(self, message: Dict[str, Any]) -> Dict[str, Any]:
        """Route message to appropriate handler."""
        msg_type = message.get("type")
        handler = self.handlers.get(msg_type)

        if not handler:
            return {"error": f"Unknown message type: {msg_type}"}

        try:
            return await handler(message)
        except Exception as e:
            return {"error": str(e)}

    async def _handle_status_update(self, message: Dict) -> Dict:
        """Handle responder status update from mesh."""
        payload = message.get("payload", {})

        # Record in sync engine
        await self.sync_engine.local_update(
            model="Responder",
            record_id=payload["responder_id"],
            fields={
                "status": payload["status"],
                "location_lat": payload["location"]["lat"],
                "location_lng": payload["location"]["lng"],
                "location_updated_at": payload["timestamp"],
                "last_seen_at": payload["timestamp"]
            }
        )

        # Update local database
        await self.db.express.update(
            "Responder",
            payload["responder_id"],
            {
                "status": payload["status"],
                "location_lat": payload["location"]["lat"],
                "location_lng": payload["location"]["lng"],
                "location_updated_at": payload["timestamp"],
                "last_seen_at": payload["timestamp"]
            }
        )

        return {"status": "ok", "type": "status_update"}

    async def _handle_task_assignment(self, message: Dict) -> Dict:
        """Handle task assignment from mesh."""
        payload = message.get("payload", {})

        # Upsert task
        await self.db.express.upsert("Task", {
            "id": payload["task_id"],
            "type": payload["task_type"],
            "priority": payload["priority"],
            "status": "assigned",
            "location_lat": payload["location"]["lat"],
            "location_lng": payload["location"]["lng"],
            "description": payload["description"],
            "assigned_to": payload["assigned_to"]
        })

        return {"status": "ok", "type": "task_assignment"}

    async def _handle_incident_report(self, message: Dict) -> Dict:
        """Handle incident report from mesh."""
        payload = message.get("payload", {})

        # Create incident (G-Set - append only)
        try:
            await self.db.express.create("Incident", {
                "id": payload["incident_id"],
                "type": payload["incident_type"],
                "severity": payload["severity"],
                "status": "active",
                "location_lat": payload["location"]["lat"],
                "location_lng": payload["location"]["lng"],
                "description": payload["description"],
                "reported_by": message.get("source_id"),
                "reported_at": payload.get("timestamp"),
                "responder_count": payload.get("responders_needed", 0)
            })
        except Exception:
            # Already exists (G-Set semantics - ignore duplicates)
            pass

        return {"status": "ok", "type": "incident_report"}

    async def _handle_resource_update(self, message: Dict) -> Dict:
        """Handle resource update from mesh."""
        payload = message.get("payload", {})

        await self.db.express.upsert("Resource", payload)

        return {"status": "ok", "type": "resource_update"}

    async def _handle_sync_request(self, message: Dict) -> Dict:
        """Handle sync state request."""
        return {
            "status": "ok",
            "type": "sync_response",
            "sync_state": self.sync_engine.get_sync_state()
        }

    async def _handle_sync_response(self, message: Dict) -> Dict:
        """Handle sync state response."""
        sync_state = message.get("sync_state", {})
        pending = sync_state.get("pending", [])

        # Convert to SyncUpdate objects and apply
        from velus.sync.sync_engine import SyncUpdate
        updates = [
            SyncUpdate(
                model=u["model"],
                record_id=u["record_id"],
                fields=u["fields"],
                clock=u["clock"],
                timestamp=u["timestamp"],
                node_id=u["node_id"]
            )
            for u in pending
        ]

        applied = await self.sync_engine.receive_updates(updates)

        return {"status": "ok", "type": "sync_response", "applied": applied}
```

### Tasks

1. [ ] Create `src/velus/mesh/__init__.py` (30m)
2. [ ] Implement `MeshMessageHandler` class (8h)
3. [ ] Implement message validation (2h)
4. [ ] Write handler tests (4h)
5. [ ] Integration test with sync engine (4h)
6. [ ] Document message flow (2h)

### Testing Requirements

- [ ] All message types handled correctly
- [ ] Invalid messages rejected gracefully
- [ ] Sync operations trigger correctly
- [ ] Database updated appropriately
- [ ] Error cases handled

### Risk Assessment

- **MEDIUM**: Integration complexity
- **Mitigation**: Clear interfaces, thorough testing

### Definition of Done

- [ ] All message handlers implemented
- [ ] All tests pass
- [ ] Integration with sync engine verified

---

## Phase 3 Dependencies Graph

```
TODO-020 (DataFlow Models)
    │
    ├──> TODO-021 (Core Workflows)
    │         │
    │         └──> TODO-023 (Mesh Bridge) <── TODO-012 (Mesh Protocol)
    │
    └──> TODO-022 (CRDT Sync Engine)
              │
              └──> TODO-023 (Mesh Bridge)
```

## Phase 3 Completion Criteria

- [ ] All DataFlow models implemented
- [ ] All core workflows functional
- [ ] CRDT sync engine operational
- [ ] Mesh-to-DataFlow bridge working
- [ ] **GO/NO-GO GATE**: Core Features functional (CRUD + sync)

## References

- Architecture Plan: `/docs/02-plans/01-architecture-plan.md` (Sections 3, 4, 6)
- DataFlow Documentation: CLAUDE.md DataFlow rules
