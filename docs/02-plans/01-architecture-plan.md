# Architecture Implementation Plan

## Overview

This document details the technical architecture implementation plan for ADRC PoC, mapping the architecture design to concrete implementation using Kailash SDK frameworks.

---

## 1. Technology Stack Decisions

### 1.1 Core Framework: Kailash SDK

| Component | Kailash Module | Purpose |
|-----------|---------------|---------|
| Workflow Engine | Core SDK (WorkflowBuilder, AsyncLocalRuntime) | Orchestrate operations |
| Database | DataFlow | Local data persistence |
| AI Agents | Kaizen (BaseAgent) | Intelligent decision support |
| API Layer | Nexus | Multi-channel exposure |

### 1.2 Mobile Platform: Flutter

**Decision**: Flutter (cross-platform) over native Android

**Rationale**:
- Single codebase for future iOS expansion
- Faster development with hot reload
- Good BLE library support (flutter_blue_plus)
- Modern UI framework

**Trade-off**: Slightly larger app size, potential BLE edge cases

### 1.3 BLE Mesh: Custom Protocol on flutter_blue_plus

**Decision**: Build custom mesh protocol on flutter_blue_plus

**Rationale**:
- Bluetooth Mesh standard too complex for PoC timeline
- Custom protocol allows optimized message format
- Full control over routing and reliability
- Simpler debugging

**Implementation**:
```
┌────────────────────────────────────────────────────┐
│              ADRC Mesh Protocol Stack              │
│                                                    │
│  ┌────────────────────────────────────────────┐   │
│  │ Application: ADRC Messages (JSON/Protobuf) │   │
│  └────────────────────────────────────────────┘   │
│  ┌────────────────────────────────────────────┐   │
│  │ Mesh: Custom flooding + gossip protocol    │   │
│  └────────────────────────────────────────────┘   │
│  ┌────────────────────────────────────────────┐   │
│  │ Transport: BLE GATT characteristics        │   │
│  └────────────────────────────────────────────┘   │
│  ┌────────────────────────────────────────────┐   │
│  │ Physical: flutter_blue_plus / BLE radio    │   │
│  └────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────┘
```

### 1.4 Database: SQLite via DataFlow

**Decision**: SQLite with DataFlow auto-generated nodes

**Rationale**:
- Zero external dependencies
- Works offline natively
- DataFlow generates CRUD nodes automatically
- Proven reliability

---

## 2. Module Architecture

### 2.1 Project Structure

```
auto-drc/
├── src/
│   └── velus/                    # Backend (Python/Kailash)
│       ├── models/               # DataFlow models
│       │   ├── responder.py
│       │   ├── task.py
│       │   ├── incident.py
│       │   └── resource.py
│       ├── workflows/            # Kailash workflows
│       │   ├── status_update.py
│       │   ├── task_assignment.py
│       │   ├── incident_report.py
│       │   └── resource_query.py
│       ├── agents/               # Kaizen AI agents
│       │   ├── task_router.py
│       │   ├── resource_matcher.py
│       │   └── situation_analyzer.py
│       ├── api/                  # Nexus API
│       │   └── platform.py
│       ├── sync/                 # CRDT sync engine
│       │   ├── crdt_types.py
│       │   └── sync_engine.py
│       └── docs/
│           └── user-flows/       # User flow documentation
├── apps/
│   └── mobile/                   # Flutter mobile app
│       ├── lib/
│       │   ├── core/             # Core utilities
│       │   ├── data/             # Data layer
│       │   ├── domain/           # Business logic
│       │   ├── presentation/     # UI layer
│       │   ├── mesh/             # BLE mesh implementation
│       │   └── sync/             # Sync integration
│       └── test/
├── docs/
│   ├── 01-analysis/
│   └── 02-plans/
└── tests/                        # Python tests
```

### 2.2 Backend Module Dependencies

```
┌─────────────────────────────────────────────────────────────┐
│                     Backend Architecture                     │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                  Nexus API Platform                   │ │
│  │  • REST API for mobile                               │ │
│  │  • Local-only (no external exposure)                 │ │
│  └───────────────────────────────────────────────────────┘ │
│                            │                                 │
│  ┌─────────────┬───────────┴───────────┬─────────────────┐ │
│  │             │                       │                 │ │
│  ▼             ▼                       ▼                 │ │
│  ┌─────────┐  ┌─────────────┐  ┌─────────────────────┐  │ │
│  │Workflows│  │   Agents    │  │    Sync Engine      │  │ │
│  │(Kailash)│  │  (Kaizen)   │  │    (CRDT)          │  │ │
│  └────┬────┘  └──────┬──────┘  └──────────┬──────────┘  │ │
│       │              │                     │              │ │
│       └──────────────┴─────────────────────┘              │ │
│                            │                               │ │
│                            ▼                               │ │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                   DataFlow Models                     │ │
│  │  • Responder, Task, Incident, Resource                │ │
│  │  • Auto-generated CRUD nodes                          │ │
│  │  • SQLite local storage                               │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 Mobile Module Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Mobile Architecture                       │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                  Presentation Layer                   │ │
│  │  • Responder UI                                       │ │
│  │  • Commander UI                                       │ │
│  │  • Map View                                           │ │
│  └───────────────────────────────────────────────────────┘ │
│                            │                                 │
│  ┌───────────────────────────────────────────────────────┐ │
│  │                   Domain Layer                        │ │
│  │  • Use cases (UpdateStatus, CreateTask, etc.)        │ │
│  │  • Business logic                                     │ │
│  └───────────────────────────────────────────────────────┘ │
│                            │                                 │
│  ┌────────────────┬───────┴────────┬─────────────────────┐ │
│  │                │                │                     │ │
│  ▼                ▼                ▼                     │ │
│  ┌──────────┐  ┌──────────┐  ┌─────────────────────┐    │ │
│  │ Local DB │  │  BLE     │  │  Backend Client     │    │ │
│  │ (SQLite) │  │  Mesh    │  │  (Local Nexus API)  │    │ │
│  └──────────┘  └──────────┘  └─────────────────────┘    │ │
│                     │                                     │ │
│                     ▼                                     │ │
│  ┌───────────────────────────────────────────────────────┐ │
│  │               BLE Mesh Engine                         │ │
│  │  • Device discovery                                   │ │
│  │  • Message routing                                    │ │
│  │  • Sync coordination                                  │ │
│  └───────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. DataFlow Models Implementation

### 3.1 Model Definitions

```python
# src/velus/models/responder.py
from dataflow import DataFlow

db = DataFlow("sqlite:///adrc.db")

@db.model
class Responder:
    id: str                    # Device UUID
    name: str
    role: str                  # medic, usar, logistics, command
    team_id: str
    agency: str                # scdf, saf, redcross, etc.
    status: str                # available, engaged, offline
    location_lat: float
    location_lng: float
    location_updated_at: str   # ISO timestamp
    skills: str                # JSON array ["cpr", "hazmat", ...]
    battery_level: int
    last_seen_at: str


@db.model
class Task:
    id: str
    type: str                  # rescue, medical, logistics, recon
    priority: str              # critical, high, medium, low
    status: str                # pending, assigned, in_progress, completed, cancelled
    location_lat: float
    location_lng: float
    description: str
    assigned_to: str           # Responder ID
    assigned_by: str           # Commander ID
    assigned_at: str
    completed_at: str
    outcome: str
    notes: str


@db.model
class Incident:
    id: str
    type: str                  # casualty, hazard, structure_collapse, etc.
    severity: str              # critical, serious, moderate, minor
    status: str                # active, resolved, false_alarm
    location_lat: float
    location_lng: float
    description: str
    reported_by: str           # Responder ID
    verified: bool
    verified_by: str
    responder_count: int       # Estimated needed


@db.model
class Resource:
    id: str
    type: str                  # vehicle, medical, equipment, personnel
    subtype: str               # ambulance, stretcher, radio, etc.
    status: str                # available, deployed, depleted, damaged
    location_lat: float
    location_lng: float
    quantity: int
    assigned_to: str           # Task or team ID
    notes: str
```

### 3.2 Auto-Generated Nodes

For each model, DataFlow generates:
- `ResponderCreateNode` - Create new responder
- `ResponderReadNode` - Read by ID
- `ResponderUpdateNode` - Update fields
- `ResponderDeleteNode` - Delete (soft)
- `ResponderListNode` - Query with filters
- `ResponderCountNode` - Count matching
- `ResponderUpsertNode` - Insert or update
- Bulk variants for batch operations

---

## 4. Workflow Implementations

### 4.1 Status Update Workflow

```python
# src/velus/workflows/status_update.py
from kailash.workflow.builder import WorkflowBuilder
from kailash.runtime import AsyncLocalRuntime

def create_status_update_workflow():
    workflow = WorkflowBuilder()

    # Input: responder_id, status, location_lat, location_lng
    workflow.add_node("ResponderUpdateNode", "update_status", {
        "filter": {"id": "{{inputs.responder_id}}"},
        "fields": {
            "status": "{{inputs.status}}",
            "location_lat": "{{inputs.location_lat}}",
            "location_lng": "{{inputs.location_lng}}",
            "location_updated_at": "{{inputs.timestamp}}"
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
    workflow.connect("update_status", "prepare_broadcast")

    return workflow


async def execute_status_update(responder_id: str, status: str, lat: float, lng: float):
    workflow = create_status_update_workflow()
    runtime = AsyncLocalRuntime()

    results, run_id = await runtime.execute_workflow_async(
        workflow.build(),
        inputs={
            "responder_id": responder_id,
            "status": status,
            "location_lat": lat,
            "location_lng": lng,
            "timestamp": datetime.now(UTC).isoformat()
        }
    )

    # Return broadcast message for mesh
    return results["prepare_broadcast"]["output"]
```

### 4.2 Task Assignment Workflow

```python
# src/velus/workflows/task_assignment.py
from kailash.workflow.builder import WorkflowBuilder
from kailash.runtime import AsyncLocalRuntime

def create_task_assignment_workflow():
    workflow = WorkflowBuilder()

    # Create task
    workflow.add_node("TaskCreateNode", "create_task", {
        "id": "{{inputs.task_id}}",
        "type": "{{inputs.task_type}}",
        "priority": "{{inputs.priority}}",
        "status": "pending",
        "location_lat": "{{inputs.location_lat}}",
        "location_lng": "{{inputs.location_lng}}",
        "description": "{{inputs.description}}",
        "assigned_by": "{{inputs.commander_id}}"
    })

    # Find best responder (calls AI agent)
    workflow.add_node("PythonCode", "find_responder", {
        "code": """
from velus.agents.task_router import TaskRouterAgent

agent = TaskRouterAgent()
recommendation = await agent.route_task(
    task_type=inputs["task_type"],
    priority=inputs["priority"],
    location=(inputs["location_lat"], inputs["location_lng"]),
    required_skills=inputs.get("required_skills", [])
)
output = {"recommended_responder": recommendation["responder_id"]}
"""
    })
    workflow.connect("create_task", "find_responder")

    # Assign to responder
    workflow.add_node("TaskUpdateNode", "assign_task", {
        "filter": {"id": "{{inputs.task_id}}"},
        "fields": {
            "status": "assigned",
            "assigned_to": "{{find_responder.output.recommended_responder}}",
            "assigned_at": "{{inputs.timestamp}}"
        }
    })
    workflow.connect("find_responder", "assign_task")

    return workflow
```

---

## 5. AI Agent Implementations

### 5.1 Task Router Agent

```python
# src/velus/agents/task_router.py
from kaizen.agent import BaseAgent
from kaizen.signatures import Signature, InputField, OutputField
from dataflow import DataFlow

class TaskRoutingSignature(Signature):
    """Route a task to the best available responder."""

    task_type: str = InputField(desc="Type of task (rescue, medical, logistics)")
    priority: str = InputField(desc="Task priority level")
    location: tuple = InputField(desc="Task location (lat, lng)")
    required_skills: list = InputField(desc="Skills needed for task")
    available_responders: list = InputField(desc="List of available responders")

    selected_responder: str = OutputField(desc="ID of selected responder")
    reasoning: str = OutputField(desc="Why this responder was selected")


class TaskRouterAgent(BaseAgent):
    """AI agent for intelligent task routing."""

    def __init__(self, db: DataFlow):
        super().__init__()
        self.db = db
        self.signature = TaskRoutingSignature

    async def route_task(
        self,
        task_type: str,
        priority: str,
        location: tuple,
        required_skills: list = None
    ) -> dict:
        # Get available responders
        responders = await self.db.express.list(
            "Responder",
            filter={"status": "available"}
        )

        if not responders:
            return {"responder_id": None, "reason": "No available responders"}

        # Score each responder
        scored = []
        for r in responders:
            score = self._calculate_score(r, task_type, location, required_skills)
            scored.append((r, score))

        # Sort by score (highest first)
        scored.sort(key=lambda x: x[1], reverse=True)

        best = scored[0]
        return {
            "responder_id": best[0]["id"],
            "reason": f"Best match: score={best[1]:.2f}",
            "alternatives": [s[0]["id"] for s in scored[1:3]]
        }

    def _calculate_score(self, responder, task_type, location, required_skills):
        score = 0.0

        # Distance factor (closer is better)
        dist = self._haversine_distance(
            (responder["location_lat"], responder["location_lng"]),
            location
        )
        score += max(0, 100 - dist) / 100  # 0-1, 1km radius normalization

        # Skills match
        if required_skills:
            responder_skills = set(json.loads(responder.get("skills", "[]")))
            skill_match = len(set(required_skills) & responder_skills) / len(required_skills)
            score += skill_match * 0.5

        # Role appropriateness
        role_match = {
            "medical": {"medic": 1.0, "usar": 0.3, "logistics": 0.1},
            "rescue": {"usar": 1.0, "medic": 0.5, "logistics": 0.2},
            "logistics": {"logistics": 1.0, "usar": 0.3, "medic": 0.2}
        }
        score += role_match.get(task_type, {}).get(responder["role"], 0.2)

        return score

    def _haversine_distance(self, coord1, coord2):
        # Simplified distance calculation
        from math import radians, cos, sin, asin, sqrt
        lat1, lon1 = radians(coord1[0]), radians(coord1[1])
        lat2, lon2 = radians(coord2[0]), radians(coord2[1])
        dlat = lat2 - lat1
        dlon = lon2 - lon1
        a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
        return 2 * 6371 * asin(sqrt(a))  # km
```

---

## 6. Sync Engine Design

### 6.1 CRDT Types

```python
# src/velus/sync/crdt_types.py
from dataclasses import dataclass
from typing import Dict, Any
import time

@dataclass
class VectorClock:
    """Vector clock for causal ordering."""
    clock: Dict[str, int]

    def increment(self, node_id: str):
        self.clock[node_id] = self.clock.get(node_id, 0) + 1

    def merge(self, other: 'VectorClock'):
        for node_id, count in other.clock.items():
            self.clock[node_id] = max(self.clock.get(node_id, 0), count)

    def happens_before(self, other: 'VectorClock') -> bool:
        return all(
            self.clock.get(k, 0) <= other.clock.get(k, 0)
            for k in set(self.clock) | set(other.clock)
        ) and self.clock != other.clock


@dataclass
class LWWRegister:
    """Last-Writer-Wins Register for simple values."""
    value: Any
    timestamp: float
    node_id: str

    def update(self, new_value: Any, timestamp: float, node_id: str):
        if timestamp > self.timestamp:
            self.value = new_value
            self.timestamp = timestamp
            self.node_id = node_id

    def merge(self, other: 'LWWRegister'):
        if other.timestamp > self.timestamp:
            self.value = other.value
            self.timestamp = other.timestamp
            self.node_id = other.node_id


@dataclass
class GrowOnlySet:
    """Grow-only set for additions (e.g., incident reports)."""
    items: set

    def add(self, item):
        self.items.add(item)

    def merge(self, other: 'GrowOnlySet'):
        self.items |= other.items


@dataclass
class ORSet:
    """Observed-Remove Set for add/remove operations."""
    adds: Dict[str, set]      # element -> set of unique tags
    removes: Dict[str, set]   # element -> set of removed tags

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

    def elements(self) -> set:
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

### 6.2 Sync Protocol

```python
# src/velus/sync/sync_engine.py
from typing import Dict, List
from .crdt_types import VectorClock, LWWRegister

class SyncEngine:
    """CRDT-based synchronization engine."""

    def __init__(self, node_id: str, db: DataFlow):
        self.node_id = node_id
        self.db = db
        self.vector_clock = VectorClock({})
        self.pending_updates: List[dict] = []

    async def local_update(self, model: str, record_id: str, fields: dict):
        """Record a local update for sync."""
        self.vector_clock.increment(self.node_id)

        update = {
            "type": "update",
            "model": model,
            "record_id": record_id,
            "fields": fields,
            "clock": dict(self.vector_clock.clock),
            "timestamp": time.time(),
            "node_id": self.node_id
        }
        self.pending_updates.append(update)
        return update

    async def receive_updates(self, updates: List[dict]):
        """Process updates from other nodes."""
        for update in updates:
            remote_clock = VectorClock(update["clock"])

            # Check if we've already seen this update
            if not self.vector_clock.happens_before(remote_clock):
                continue  # Already have this or newer

            # Apply update
            await self._apply_update(update)

            # Merge clocks
            self.vector_clock.merge(remote_clock)

    async def _apply_update(self, update: dict):
        """Apply a remote update to local database."""
        model = update["model"]
        record_id = update["record_id"]
        fields = update["fields"]

        # Use LWW for conflict resolution
        await self.db.express.update(model, record_id, fields)

    def get_sync_state(self) -> dict:
        """Get current sync state for exchange."""
        return {
            "clock": dict(self.vector_clock.clock),
            "pending": self.pending_updates[-100:]  # Last 100 updates
        }

    def clear_pending(self):
        """Clear pending updates after successful sync."""
        self.pending_updates = []
```

---

## 7. Mobile-Backend Integration

### 7.1 Local API Server

Each mobile device runs a local Nexus API for internal coordination:

```python
# src/velus/api/platform.py
from nexus import Nexus
from velus.workflows import (
    create_status_update_workflow,
    create_task_assignment_workflow,
    create_incident_report_workflow
)

def create_adrc_platform(db: DataFlow):
    platform = Nexus("ADRC Local Platform")

    # Register workflows
    platform.register_workflow(
        "status_update",
        create_status_update_workflow(),
        description="Update responder status and location"
    )

    platform.register_workflow(
        "task_assign",
        create_task_assignment_workflow(),
        description="Create and assign a task"
    )

    platform.register_workflow(
        "incident_report",
        create_incident_report_workflow(),
        description="Report a new incident"
    )

    return platform


# Run on device
if __name__ == "__main__":
    from velus.models import db
    platform = create_adrc_platform(db)
    platform.run(port=8080, host="127.0.0.1")  # Local only
```

### 7.2 Flutter-Python Bridge

```dart
// apps/mobile/lib/core/backend_client.dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class BackendClient {
  static const String baseUrl = 'http://127.0.0.1:8080';

  Future<Map<String, dynamic>> updateStatus({
    required String responderId,
    required String status,
    required double lat,
    required double lng,
  }) async {
    final response = await http.post(
      Uri.parse('$baseUrl/workflows/status_update'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({
        'responder_id': responderId,
        'status': status,
        'location_lat': lat,
        'location_lng': lng,
      }),
    );
    return jsonDecode(response.body);
  }

  Future<Map<String, dynamic>> createTask({
    required String taskType,
    required String priority,
    required double lat,
    required double lng,
    required String description,
    required String commanderId,
  }) async {
    final response = await http.post(
      Uri.parse('$baseUrl/workflows/task_assign'),
      body: jsonEncode({
        'task_type': taskType,
        'priority': priority,
        'location_lat': lat,
        'location_lng': lng,
        'description': description,
        'commander_id': commanderId,
      }),
    );
    return jsonDecode(response.body);
  }
}
```

---

## 8. Implementation Priorities

### 8.1 Week-by-Week Implementation

| Week | Backend Focus | Mobile Focus |
|------|---------------|--------------|
| 1-2 | DataFlow models, basic workflows | BLE discovery, basic UI |
| 3-4 | Sync engine, more workflows | Mesh protocol, message passing |
| 5-6 | AI agents (rule-based) | Local DB, backend integration |
| 7-8 | Nexus API, refinement | Full UI, map integration |
| 9-10 | Security, hardening | Security, polish |
| 11-12 | Testing, docs | Testing, demo prep |

### 8.2 Critical Path Items

1. **BLE Mesh Viability** (Week 3-4): Must validate before proceeding
2. **DataFlow Integration** (Week 5): Foundation for all data operations
3. **Mobile-Backend Bridge** (Week 6): Required for AI integration
4. **End-to-End Flow** (Week 8): Proof of concept validation
