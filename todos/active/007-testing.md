# TODO-060 to TODO-066: Testing & Demo Phase (Week 11-12)

**Phase**: 7 - Testing & Demo
**Timeline**: Week 11-12
**Status**: PENDING

---

## TODO-060: Unit Test Suite

**Status**: PENDING
**Owner**: QA
**Estimated Effort**: 3 days
**Dependencies**: TODO-021 (Workflows), TODO-031 (Agents)

### Description

Create comprehensive unit test suite for all Python backend components following the Kailash SDK 3-tier testing strategy.

### Acceptance Criteria

- [ ] DataFlow model tests
- [ ] Workflow unit tests
- [ ] Agent unit tests (with mocking for LLM)
- [ ] CRDT type tests
- [ ] Sync engine tests
- [ ] Utility function tests
- [ ] 80%+ code coverage

### Test Organization (Tier 1)

```
tests/
├── unit/                      # Tier 1: Unit tests
│   ├── models/
│   │   ├── test_responder.py
│   │   ├── test_task.py
│   │   ├── test_incident.py
│   │   └── test_resource.py
│   ├── workflows/
│   │   ├── test_status_update.py
│   │   ├── test_task_assignment.py
│   │   ├── test_incident_report.py
│   │   └── test_resource_query.py
│   ├── agents/
│   │   ├── test_base_edge_agent.py
│   │   ├── test_task_router.py
│   │   ├── test_resource_matcher.py
│   │   └── test_situation_analyzer.py
│   ├── sync/
│   │   ├── test_crdt_types.py
│   │   └── test_sync_engine.py
│   └── security/
│       ├── test_classification.py
│       ├── test_rbac.py
│       └── test_audit.py
├── integration/               # Tier 2: Integration tests
└── e2e/                       # Tier 3: End-to-end tests
```

### Unit Test Examples

```python
# tests/unit/agents/test_task_router.py
import pytest
from unittest.mock import AsyncMock, MagicMock
from velus.agents.task_router import TaskRouterAgent, TaskRoutingSignature
from velus.agents.base_edge_agent import EdgeAgentConfig


class TestTaskRouterAgent:
    """Unit tests for TaskRouterAgent."""

    @pytest.fixture
    def mock_db(self):
        """Create mock DataFlow database."""
        db = MagicMock()
        db.express = MagicMock()
        db.express.list = AsyncMock(return_value=[
            {
                "id": "resp-001",
                "name": "Alice",
                "role": "medic",
                "status": "available",
                "location_lat": 1.3521,
                "location_lng": 103.8198,
                "skills": '["cpr", "first_aid"]'
            },
            {
                "id": "resp-002",
                "name": "Bob",
                "role": "usar",
                "status": "available",
                "location_lat": 1.3550,
                "location_lng": 103.8200,
                "skills": '["rescue", "hazmat"]'
            }
        ])
        return db

    @pytest.fixture
    def agent(self, mock_db):
        """Create agent in offline mode (rule-based only)."""
        config = EdgeAgentConfig(
            name="TestRouter",
            offline_mode=True  # Force rule-based for deterministic tests
        )
        return TaskRouterAgent(mock_db, config)

    @pytest.mark.asyncio
    async def test_route_to_closest_responder(self, agent):
        """Should route to closest available responder."""
        result = await agent.route_task(
            task_type="medical",
            priority="high",
            location=(1.3521, 103.8198),  # Same as resp-001
            required_skills=["cpr"]
        )

        assert result["selected_responder_id"] == "resp-001"
        assert result["confidence"] > 0

    @pytest.mark.asyncio
    async def test_route_by_skills(self, agent):
        """Should prefer responder with matching skills."""
        result = await agent.route_task(
            task_type="rescue",
            priority="high",
            location=(1.3535, 103.8199),  # Equidistant
            required_skills=["rescue", "hazmat"]
        )

        # Bob has rescue and hazmat skills
        assert result["selected_responder_id"] == "resp-002"

    @pytest.mark.asyncio
    async def test_route_no_responders(self, mock_db):
        """Should handle no available responders gracefully."""
        mock_db.express.list = AsyncMock(return_value=[])
        agent = TaskRouterAgent(mock_db, EdgeAgentConfig(offline_mode=True))

        result = await agent.route_task(
            task_type="medical",
            priority="critical",
            location=(1.3521, 103.8198)
        )

        assert result["selected_responder_id"] is None
        assert "no available" in result["reasoning"].lower()

    @pytest.mark.asyncio
    async def test_provides_alternatives(self, agent):
        """Should provide alternative responders."""
        result = await agent.route_task(
            task_type="medical",
            priority="medium",
            location=(1.3521, 103.8198)
        )

        assert "alternatives" in result
        assert len(result["alternatives"]) > 0


# tests/unit/sync/test_crdt_types.py
import pytest
from velus.sync.crdt_types import VectorClock, LWWRegister, GrowOnlySet, ORSet


class TestVectorClock:
    """Unit tests for VectorClock CRDT."""

    def test_increment(self):
        """Should increment clock for node."""
        clock = VectorClock()
        clock.increment("node-1")
        clock.increment("node-1")
        clock.increment("node-2")

        assert clock.clock["node-1"] == 2
        assert clock.clock["node-2"] == 1

    def test_merge(self):
        """Should merge clocks taking max values."""
        clock1 = VectorClock({"a": 2, "b": 1})
        clock2 = VectorClock({"a": 1, "b": 3, "c": 1})

        clock1.merge(clock2)

        assert clock1.clock["a"] == 2  # Max of 2, 1
        assert clock1.clock["b"] == 3  # Max of 1, 3
        assert clock1.clock["c"] == 1  # New from clock2

    def test_happens_before(self):
        """Should correctly detect causal ordering."""
        clock1 = VectorClock({"a": 1, "b": 1})
        clock2 = VectorClock({"a": 2, "b": 2})

        assert clock1.happens_before(clock2)
        assert not clock2.happens_before(clock1)

    def test_concurrent(self):
        """Should detect concurrent clocks."""
        clock1 = VectorClock({"a": 2, "b": 1})
        clock2 = VectorClock({"a": 1, "b": 2})

        assert clock1.concurrent(clock2)


class TestLWWRegister:
    """Unit tests for Last-Writer-Wins Register."""

    def test_update_newer_wins(self):
        """Newer timestamp should win."""
        register = LWWRegister(value="old", timestamp=1.0, node_id="a")
        updated = register.update("new", timestamp=2.0, node_id="b")

        assert updated is True
        assert register.value == "new"

    def test_update_older_loses(self):
        """Older timestamp should not update."""
        register = LWWRegister(value="current", timestamp=2.0, node_id="a")
        updated = register.update("old", timestamp=1.0, node_id="b")

        assert updated is False
        assert register.value == "current"

    def test_tie_breaker(self):
        """Same timestamp should use node_id as tie breaker."""
        register = LWWRegister(value="a_value", timestamp=1.0, node_id="a")
        updated = register.update("b_value", timestamp=1.0, node_id="b")

        assert updated is True  # "b" > "a"
        assert register.value == "b_value"


class TestORSet:
    """Unit tests for Observed-Remove Set."""

    def test_add_element(self):
        """Should add elements correctly."""
        orset = ORSet()
        orset.add("item1", "node-1")
        orset.add("item2", "node-1")

        assert orset.contains("item1")
        assert orset.contains("item2")
        assert not orset.contains("item3")

    def test_remove_element(self):
        """Should remove elements correctly."""
        orset = ORSet()
        orset.add("item1", "node-1")
        orset.remove("item1")

        assert not orset.contains("item1")

    def test_add_after_remove(self):
        """Should allow re-adding after remove."""
        orset = ORSet()
        orset.add("item1", "node-1")
        orset.remove("item1")
        orset.add("item1", "node-2")  # Different tag

        assert orset.contains("item1")

    def test_merge(self):
        """Should merge two ORSets correctly."""
        orset1 = ORSet()
        orset1.add("a", "node-1")
        orset1.add("b", "node-1")

        orset2 = ORSet()
        orset2.add("b", "node-2")
        orset2.add("c", "node-2")
        orset2.remove("b")  # Only removes node-2's add

        orset1.merge(orset2)

        assert orset1.contains("a")
        assert orset1.contains("b")  # node-1's add still valid
        assert orset1.contains("c")
```

### Tasks

1. [ ] Create test directory structure (1h)
2. [ ] Write DataFlow model tests (3h)
3. [ ] Write workflow unit tests (4h)
4. [ ] Write agent unit tests (4h)
5. [ ] Write CRDT unit tests (3h)
6. [ ] Write sync engine unit tests (3h)
7. [ ] Write security module tests (3h)
8. [ ] Achieve 80%+ coverage (ongoing)

### Testing Requirements

- [ ] All unit tests pass
- [ ] Coverage >= 80%
- [ ] Tests are deterministic (no flakiness)
- [ ] Tests run in < 30 seconds

### Definition of Done

- [ ] Complete unit test suite
- [ ] All tests pass
- [ ] Coverage target met

---

## TODO-061: Integration Test Suite

**Status**: PENDING
**Owner**: QA
**Estimated Effort**: 3 days
**Dependencies**: TODO-051 (E2E Integration)

### Description

Create integration test suite (Tier 2) that tests component interactions with real infrastructure. **NO MOCKING** of DataFlow or external services.

### Acceptance Criteria

- [ ] Workflow + DataFlow integration tests
- [ ] Agent + DataFlow integration tests
- [ ] Sync engine integration tests
- [ ] Mesh + Data layer integration tests
- [ ] All tests use real SQLite database
- [ ] **NO MOCKING in Tier 2** (Kailash policy)

### Test Examples (Tier 2)

```python
# tests/integration/test_task_assignment_flow.py
import pytest
from datetime import datetime, UTC
from dataflow import DataFlow
from velus.agents.orchestrator import AgentOrchestrator
from velus.agents.task_router import TaskRouterAgent
from velus.models import Responder, Task

# Use real database - NO MOCKING
TEST_DB_URL = "sqlite:///test_integration.db"


@pytest.fixture
async def db():
    """Create real test database."""
    db = DataFlow(TEST_DB_URL)
    db.register_model(Responder)
    db.register_model(Task)
    await db.create_tables_async()

    yield db

    # Cleanup
    await db.drop_tables_async()


@pytest.fixture
async def seeded_db(db):
    """Database with test responders."""
    responders = [
        {
            "id": "resp-001",
            "name": "Alice",
            "role": "medic",
            "team_id": "team-1",
            "agency": "scdf",
            "status": "available",
            "location_lat": 1.3521,
            "location_lng": 103.8198,
            "location_updated_at": datetime.now(UTC).isoformat(),
            "skills": '["cpr", "first_aid", "triage"]',
            "battery_level": 85,
            "last_seen_at": datetime.now(UTC).isoformat()
        },
        {
            "id": "resp-002",
            "name": "Bob",
            "role": "usar",
            "team_id": "team-2",
            "agency": "scdf",
            "status": "available",
            "location_lat": 1.3550,
            "location_lng": 103.8200,
            "location_updated_at": datetime.now(UTC).isoformat(),
            "skills": '["rescue", "hazmat"]',
            "battery_level": 90,
            "last_seen_at": datetime.now(UTC).isoformat()
        }
    ]

    for r in responders:
        await db.express.create("Responder", r)

    return db


class TestTaskAssignmentIntegration:
    """Integration tests for task assignment flow."""

    @pytest.mark.asyncio
    async def test_ai_task_assignment(self, seeded_db):
        """Test full AI-assisted task assignment flow."""
        orchestrator = AgentOrchestrator(seeded_db)

        # Create and assign task with AI
        result = await orchestrator.create_and_assign_task(
            task_id="task-001",
            task_type="medical",
            priority="high",
            location=(1.3521, 103.8198),  # Near Alice
            description="Medical emergency at sector A",
            commander_id="cmd-001",
            required_skills=["triage"]
        )

        # Verify success
        assert result["success"] is True
        assert result["assigned_to"] == "resp-001"  # Alice

        # Verify task in database
        task = await seeded_db.express.read("Task", "task-001")
        assert task is not None
        assert task["status"] == "assigned"
        assert task["assigned_to"] == "resp-001"

    @pytest.mark.asyncio
    async def test_task_assignment_updates_availability(self, seeded_db):
        """Verify responder status updates after assignment."""
        orchestrator = AgentOrchestrator(seeded_db)

        # Assign task
        result = await orchestrator.create_and_assign_task(
            task_id="task-002",
            task_type="medical",
            priority="high",
            location=(1.3521, 103.8198),
            description="Test task",
            commander_id="cmd-001"
        )

        assigned_id = result["assigned_to"]

        # In real system, responder would update own status
        # Here we simulate the status update
        await seeded_db.express.update("Responder", assigned_id, {
            "status": "engaged"
        })

        # Verify status updated
        responder = await seeded_db.express.read("Responder", assigned_id)
        assert responder["status"] == "engaged"


# tests/integration/test_sync_flow.py
@pytest.fixture
async def two_sync_engines(db):
    """Create two sync engines simulating two devices."""
    from velus.sync.sync_engine import SyncEngine

    engine1 = SyncEngine("device-1", db)
    engine2 = SyncEngine("device-2", db)

    return engine1, engine2


class TestSyncIntegration:
    """Integration tests for sync engine with real database."""

    @pytest.mark.asyncio
    async def test_sync_propagates_updates(self, seeded_db, two_sync_engines):
        """Test that updates sync between engines."""
        engine1, engine2 = two_sync_engines

        # Engine 1 makes a local update
        update = await engine1.local_update(
            model="Responder",
            record_id="resp-001",
            fields={"status": "engaged", "location_updated_at": datetime.now(UTC).isoformat()}
        )

        # Engine 2 receives the update
        sync_state = engine1.get_sync_state()
        from velus.sync.sync_engine import SyncUpdate
        updates = [
            SyncUpdate(**u) for u in sync_state["pending"]
        ]
        applied = await engine2.receive_updates(updates)

        assert applied == 1

        # Verify database updated
        responder = await seeded_db.express.read("Responder", "resp-001")
        assert responder["status"] == "engaged"

    @pytest.mark.asyncio
    async def test_conflict_resolution_lww(self, seeded_db, two_sync_engines):
        """Test LWW conflict resolution."""
        engine1, engine2 = two_sync_engines
        import time

        # Engine 1 updates first
        await engine1.local_update(
            model="Responder",
            record_id="resp-001",
            fields={"status": "engaged", "location_updated_at": "2024-01-01T10:00:00Z"}
        )

        time.sleep(0.1)  # Ensure different timestamp

        # Engine 2 updates later (should win)
        await engine2.local_update(
            model="Responder",
            record_id="resp-001",
            fields={"status": "available", "location_updated_at": "2024-01-01T10:01:00Z"}
        )

        # Cross-sync both
        state1 = engine1.get_sync_state()
        state2 = engine2.get_sync_state()

        from velus.sync.sync_engine import SyncUpdate
        await engine1.receive_updates([SyncUpdate(**u) for u in state2["pending"]])
        await engine2.receive_updates([SyncUpdate(**u) for u in state1["pending"]])

        # Both should have "available" (later timestamp wins)
        responder = await seeded_db.express.read("Responder", "resp-001")
        assert responder["status"] == "available"
```

### Tasks

1. [ ] Set up integration test infrastructure (2h)
2. [ ] Write workflow + DataFlow tests (4h)
3. [ ] Write agent + DataFlow tests (4h)
4. [ ] Write sync engine integration tests (4h)
5. [ ] Write mesh + data layer tests (4h)
6. [ ] Ensure NO MOCKING (ongoing)

### Testing Requirements

- [ ] All tests use real database
- [ ] No mocking of DataFlow
- [ ] Tests clean up after themselves
- [ ] Tests are isolated from each other

### Definition of Done

- [ ] Integration test suite complete
- [ ] All tests pass
- [ ] No mocking violations

---

## TODO-062: Load Testing (50-100 nodes)

**Status**: PENDING
**Owner**: QA
**Estimated Effort**: 3 days
**Dependencies**: TODO-061 (Integration Tests)

### Description

Perform load testing with 50-100 simulated nodes to validate system capacity.

### Acceptance Criteria

- [ ] 50 simulated nodes operational
- [ ] 100 simulated nodes tested
- [ ] Message delivery rate >= 90%
- [ ] System stable for 2+ hours
- [ ] Performance metrics documented

### Test Scenarios

#### Scenario 1: Node Density Test
- 50 nodes in single cluster
- All nodes broadcasting status every 30 seconds
- Measure message throughput and latency

#### Scenario 2: Task Storm Test
- 50 active nodes
- 100 tasks created in 5 minutes
- Measure task routing time and accuracy

#### Scenario 3: Incident Flood Test
- 100 nodes
- 50 incidents reported simultaneously
- Measure propagation time and delivery rate

#### Scenario 4: Sync Stress Test
- 100 nodes with 1000 records each
- Full sync between nodes
- Measure sync time and data consistency

### Load Test Framework

```python
# tests/load/test_mesh_capacity.py
import asyncio
import time
from dataclasses import dataclass
from typing import List
import statistics


@dataclass
class LoadTestResult:
    scenario: str
    node_count: int
    duration_seconds: float
    messages_sent: int
    messages_received: int
    delivery_rate: float
    avg_latency_ms: float
    p95_latency_ms: float
    p99_latency_ms: float
    errors: int


class MeshLoadTester:
    """Load testing framework for BLE mesh simulation."""

    def __init__(self, node_count: int):
        self.node_count = node_count
        self.nodes: List[SimulatedNode] = []
        self.latencies: List[float] = []
        self.sent = 0
        self.received = 0
        self.errors = 0

    async def setup(self):
        """Create simulated nodes."""
        for i in range(self.node_count):
            node = SimulatedNode(f"node-{i:03d}")
            node.on_message_received = self._record_receive
            self.nodes.append(node)

        # Connect nodes (simulate mesh topology)
        for node in self.nodes:
            # Each node connected to ~5-10 neighbors
            neighbors = self._get_random_neighbors(node, count=7)
            for neighbor in neighbors:
                node.connect(neighbor)

    async def run_broadcast_test(self, duration_seconds: int, interval_seconds: float):
        """Run broadcast load test."""
        start = time.time()
        tasks = []

        for node in self.nodes:
            task = asyncio.create_task(
                self._broadcast_loop(node, duration_seconds, interval_seconds)
            )
            tasks.append(task)

        await asyncio.gather(*tasks)
        end = time.time()

        return LoadTestResult(
            scenario="broadcast",
            node_count=self.node_count,
            duration_seconds=end - start,
            messages_sent=self.sent,
            messages_received=self.received,
            delivery_rate=self.received / (self.sent * (self.node_count - 1)) if self.sent > 0 else 0,
            avg_latency_ms=statistics.mean(self.latencies) if self.latencies else 0,
            p95_latency_ms=statistics.quantiles(self.latencies, n=20)[18] if len(self.latencies) > 20 else 0,
            p99_latency_ms=statistics.quantiles(self.latencies, n=100)[98] if len(self.latencies) > 100 else 0,
            errors=self.errors
        )

    async def _broadcast_loop(self, node, duration: int, interval: float):
        """Broadcast messages from a node."""
        end_time = time.time() + duration
        while time.time() < end_time:
            try:
                send_time = time.time()
                await node.broadcast({
                    "type": "status",
                    "send_time": send_time
                })
                self.sent += 1
            except Exception as e:
                self.errors += 1
            await asyncio.sleep(interval)

    def _record_receive(self, message):
        """Record message receipt."""
        self.received += 1
        if "send_time" in message:
            latency = (time.time() - message["send_time"]) * 1000
            self.latencies.append(latency)


# Run load tests
async def test_50_node_broadcast():
    """Test 50 nodes broadcasting."""
    tester = MeshLoadTester(50)
    await tester.setup()

    result = await tester.run_broadcast_test(
        duration_seconds=120,  # 2 minutes
        interval_seconds=30    # Every 30 seconds
    )

    assert result.delivery_rate >= 0.90, f"Delivery rate {result.delivery_rate} below 90%"
    assert result.avg_latency_ms < 1000, f"Average latency {result.avg_latency_ms}ms too high"
    print(f"50-node test: {result}")


async def test_100_node_broadcast():
    """Test 100 nodes broadcasting."""
    tester = MeshLoadTester(100)
    await tester.setup()

    result = await tester.run_broadcast_test(
        duration_seconds=120,
        interval_seconds=30
    )

    assert result.delivery_rate >= 0.85, f"Delivery rate {result.delivery_rate} below 85%"
    print(f"100-node test: {result}")
```

### Tasks

1. [ ] Create load test framework (4h)
2. [ ] Implement node simulation (4h)
3. [ ] Run 50-node tests (4h)
4. [ ] Run 100-node tests (4h)
5. [ ] Document results and recommendations (4h)

### Metrics to Collect

| Metric | Target (50 nodes) | Target (100 nodes) |
|--------|-------------------|---------------------|
| Delivery rate | >= 95% | >= 90% |
| Avg latency | < 500ms | < 1000ms |
| P95 latency | < 1000ms | < 2000ms |
| Memory per node | < 50MB | < 50MB |
| CPU utilization | < 30% | < 50% |

### Definition of Done

- [ ] Load tests implemented and run
- [ ] Results documented
- [ ] Performance meets targets

---

## TODO-063: Failure Mode Testing

**Status**: PENDING
**Owner**: QA
**Estimated Effort**: 2 days
**Dependencies**: TODO-062 (Load Testing)

### Description

Test system behavior under various failure conditions to ensure resilience.

### Acceptance Criteria

- [ ] Network partition recovery tested
- [ ] Node failure/restart tested
- [ ] Database corruption recovery tested
- [ ] Message loss handling tested
- [ ] Clock drift handling tested

### Failure Scenarios

#### F1: Network Partition
- Split mesh into 2 groups
- Both groups continue operating
- Reconnect and verify sync

#### F2: Node Crash and Recovery
- Kill node mid-operation
- Restart node
- Verify data consistency

#### F3: Message Loss
- Drop 20% of messages randomly
- Verify eventual consistency
- Measure time to converge

#### F4: Database Corruption
- Corrupt sync log
- Verify recovery mechanism
- No data loss

### Tasks

1. [ ] Implement partition simulation (3h)
2. [ ] Test partition recovery (3h)
3. [ ] Test node crash recovery (3h)
4. [ ] Test message loss resilience (3h)
5. [ ] Document failure handling (2h)

### Definition of Done

- [ ] All failure scenarios tested
- [ ] Recovery documented
- [ ] System resilient to failures

---

## TODO-064: Demo Scenario Development

**Status**: PENDING
**Owner**: Product
**Estimated Effort**: 2 days
**Dependencies**: TODO-051 (E2E Integration)

### Description

Develop the earthquake response simulation demo scenario for stakeholder presentation.

### Acceptance Criteria

- [ ] Demo script written
- [ ] All roles defined
- [ ] Timeline documented
- [ ] Success criteria defined
- [ ] Backup procedures documented

### Demo Script: Earthquake Response Simulation

```markdown
# ADRC PoC Demo: Earthquake Response Simulation
Duration: 30 minutes

## Setup (5 minutes)
- 50 devices deployed
- 5 teams (10 responders each)
- 2 sectors (A and B)
- 2 commanders (1 per sector)

## Phase 1: Deployment (5 minutes)
1. Teams power on devices
2. Devices join mesh network
3. Commander dashboards show all responders
4. Map displays all positions

**Demo Point**: Show mesh formation in real-time

## Phase 2: Initial Response (10 minutes)
1. Sector A: 3 incidents reported
   - Critical: Building collapse (needs USAR)
   - High: Multiple casualties (needs medics)
   - Medium: Trapped vehicle (needs logistics)

2. AI Task Routing Demo:
   - Commander creates tasks for incidents
   - AI recommends optimal responders
   - Show reasoning and alternatives
   - Commander approves (1-click)

3. Responders receive tasks:
   - Notification on device
   - Navigation to location
   - Accept and start task

**Demo Point**: AI routing speed and accuracy

## Phase 3: Network Partition (5 minutes)
1. Simulate partition (turn off bridge devices)
2. Sector A and B operate independently
3. Show both sectors continue working
4. New incidents handled locally

**Demo Point**: Offline resilience

## Phase 4: Recovery and Sync (5 minutes)
1. Reconnect partition
2. Data syncs automatically
3. Commanders see unified view
4. No data loss demonstrated

**Demo Point**: CRDT sync and consistency

## Debrief (5 minutes)
- Show situation summary (AI-generated)
- Display key metrics
- Q&A
```

### Tasks

1. [ ] Write detailed demo script (4h)
2. [ ] Define roles and responsibilities (2h)
3. [ ] Create demo data/incidents (3h)
4. [ ] Rehearse demo flow (4h)
5. [ ] Document backup procedures (2h)

### Definition of Done

- [ ] Demo script complete
- [ ] Successfully rehearsed
- [ ] Backup plans documented

---

## TODO-065: Demo Environment Setup

**Status**: PENDING
**Owner**: DevOps
**Estimated Effort**: 2 days
**Dependencies**: TODO-064 (Demo Scenario)

### Description

Set up the physical and software environment for the demo.

### Acceptance Criteria

- [ ] 50 test devices configured
- [ ] All devices have app installed
- [ ] Network topology planned
- [ ] Backup devices available
- [ ] Monitoring in place

### Environment Requirements

| Item | Quantity | Purpose |
|------|----------|---------|
| Android phones | 50 | Responder devices |
| Tablets | 5 | Commander devices |
| Backup devices | 10 | Failover |
| Charging stations | 5 | Keep devices charged |
| Large display | 2 | Show dashboards |
| WiFi router | 1 | For monitoring only |

### Tasks

1. [ ] Source and configure devices (4h)
2. [ ] Install app on all devices (4h)
3. [ ] Set up monitoring dashboard (3h)
4. [ ] Configure network topology (2h)
5. [ ] Test complete setup (4h)

### Definition of Done

- [ ] All devices configured
- [ ] Demo environment operational
- [ ] Monitoring working

---

## TODO-066: User Documentation

**Status**: PENDING
**Owner**: Tech Writer
**Estimated Effort**: 3 days
**Dependencies**: TODO-051 (E2E Integration)

### Description

Create user documentation for responders and commanders.

### Acceptance Criteria

- [ ] Responder quick start guide
- [ ] Commander quick start guide
- [ ] Troubleshooting guide
- [ ] FAQ document
- [ ] Video tutorials (optional)

### Documentation Structure

```
docs/user/
├── quick-start-responder.md
├── quick-start-commander.md
├── user-guide/
│   ├── 01-getting-started.md
│   ├── 02-status-updates.md
│   ├── 03-task-management.md
│   ├── 04-incident-reporting.md
│   └── 05-map-navigation.md
├── troubleshooting.md
└── faq.md
```

### Tasks

1. [ ] Write responder quick start (3h)
2. [ ] Write commander quick start (3h)
3. [ ] Write detailed user guide (8h)
4. [ ] Write troubleshooting guide (3h)
5. [ ] Write FAQ (2h)
6. [ ] Review and edit (3h)

### Definition of Done

- [ ] All documentation complete
- [ ] Reviewed by non-technical user
- [ ] Accessible from app

---

## Phase 7 Dependencies Graph

```
TODO-060 (Unit Tests)
    │
    └──> TODO-061 (Integration Tests)
              │
              └──> TODO-062 (Load Testing)
                        │
                        └──> TODO-063 (Failure Mode Testing)

TODO-051 (E2E Integration)
    │
    ├──> TODO-064 (Demo Scenario)
    │         │
    │         └──> TODO-065 (Demo Environment)
    │
    └──> TODO-066 (User Documentation)
```

## Phase 7 Completion Criteria

- [ ] Unit test coverage >= 80%
- [ ] All integration tests pass
- [ ] Load testing shows 50+ nodes working
- [ ] Failure modes handled gracefully
- [ ] Demo scenario ready
- [ ] Demo environment set up
- [ ] User documentation complete
- [ ] **PoC COMPLETE**: Ready for stakeholder demo

## PoC Success Criteria

| Criterion | Minimum | Target | Status |
|-----------|---------|--------|--------|
| Mesh nodes operational | 30 | 50-100 | |
| Message delivery rate | 90% | 95%+ | |
| Core features offline | 80% | 100% | |
| Task completion rate | 70% | 85%+ | |
| User satisfaction | Neutral | Positive | |

## References

- Roadmap Success Criteria: `/docs/02-plans/00-poc-roadmap.md` (Section 1.2)
- Testing Strategy: Kailash SDK 3-tier approach
