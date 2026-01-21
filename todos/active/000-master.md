# ADRC PoC - Master Todo List

**Project**: Autonomous Disaster Response Coordination (ADRC)
**Timeline**: 3 months accelerated (12 weeks)
**Goal**: Demonstrate 50-100 node BLE mesh with AI coordination

---

## Phase 1: Foundation (Week 1-2)

- [ ] TODO-001: Project Setup & Environment
  - Status: PENDING
  - Owner: Tech Lead
  - Dependencies: None
  - Estimated Effort: 2 days
  - Details: [001-foundation.md](./001-foundation.md)

- [ ] TODO-002: Architecture Decisions & ADRs
  - Status: PENDING
  - Owner: Tech Lead
  - Dependencies: TODO-001
  - Estimated Effort: 3 days
  - Details: [001-foundation.md](./001-foundation.md)

- [ ] TODO-003: Flutter Mobile Project Setup
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-001
  - Estimated Effort: 2 days
  - Details: [001-foundation.md](./001-foundation.md)

- [ ] TODO-004: Design System Foundation
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-003
  - Estimated Effort: 3 days
  - Details: [005-frontend.md](./005-frontend.md)

---

## Phase 2: BLE Mesh Network (Week 3-4)

- [ ] TODO-010: BLE Library Integration
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-003
  - Estimated Effort: 2 days
  - Details: [002-network.md](./002-network.md)

- [ ] TODO-011: Device Discovery Service
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-010
  - Estimated Effort: 2 days
  - Details: [002-network.md](./002-network.md)

- [ ] TODO-012: Custom Mesh Protocol
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-011
  - Estimated Effort: 4 days
  - Details: [002-network.md](./002-network.md)

- [ ] TODO-013: Message Serialization
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-002
  - Estimated Effort: 2 days
  - Details: [002-network.md](./002-network.md)

- [ ] TODO-014: BLE Range Testing
  - Status: PENDING
  - Owner: QA
  - Dependencies: TODO-012
  - Estimated Effort: 2 days
  - Details: [002-network.md](./002-network.md)

**RISK GATE**: Week 4 - BLE mesh viability check (5+ nodes communicating)

---

## Phase 3: Data Layer (Week 5-6)

- [ ] TODO-020: DataFlow Models Implementation
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-001
  - Estimated Effort: 3 days
  - Details: [003-data-layer.md](./003-data-layer.md)
  - **CRITICAL**: Use `${}` template syntax, NOT `{{}}`

- [ ] TODO-021: Core Workflows (Kailash SDK)
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-020
  - Estimated Effort: 4 days
  - Details: [003-data-layer.md](./003-data-layer.md)
  - **CRITICAL**: Use `add_connection()`, NOT `connect()`

- [ ] TODO-022: CRDT Sync Engine
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-020
  - Estimated Effort: 4 days
  - Details: [003-data-layer.md](./003-data-layer.md)

- [ ] TODO-023: Mesh-to-DataFlow Bridge
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-021, TODO-012
  - Estimated Effort: 3 days
  - Details: [003-data-layer.md](./003-data-layer.md)

---

## Phase 4: AI Agents (Week 7-8)

- [ ] TODO-030: BaseEdgeAgent Pattern
  - Status: PENDING
  - Owner: AI Dev
  - Dependencies: TODO-020
  - Estimated Effort: 2 days
  - Details: [004-ai-agents.md](./004-ai-agents.md)
  - **CRITICAL**: Offline-first design, rule-based fallbacks

- [ ] TODO-031: Task Routing Agent
  - Status: PENDING
  - Owner: AI Dev
  - Dependencies: TODO-030
  - Estimated Effort: 3 days
  - Details: [004-ai-agents.md](./004-ai-agents.md)

- [ ] TODO-032: Resource Matching Agent
  - Status: PENDING
  - Owner: AI Dev
  - Dependencies: TODO-030
  - Estimated Effort: 2 days
  - Details: [004-ai-agents.md](./004-ai-agents.md)

- [ ] TODO-033: Situation Summary Agent
  - Status: PENDING
  - Owner: AI Dev
  - Dependencies: TODO-030
  - Estimated Effort: 2 days
  - Details: [004-ai-agents.md](./004-ai-agents.md)

- [ ] TODO-034: Agent-Workflow Integration
  - Status: PENDING
  - Owner: AI Dev
  - Dependencies: TODO-031, TODO-021
  - Estimated Effort: 2 days
  - Details: [004-ai-agents.md](./004-ai-agents.md)
  - **CRITICAL**: Call agents outside workflows (PythonCode cannot await)

---

## Phase 5: Frontend (Ongoing with Week 7-8 focus)

- [ ] TODO-040: Core UI Components
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-004
  - Estimated Effort: 4 days
  - Details: [005-frontend.md](./005-frontend.md)
  - **CRITICAL**: 56dp minimum touch targets

- [ ] TODO-041: Responder App Screens
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-040
  - Estimated Effort: 4 days
  - Details: [005-frontend.md](./005-frontend.md)

- [ ] TODO-042: Commander Dashboard
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-040
  - Estimated Effort: 3 days
  - Details: [005-frontend.md](./005-frontend.md)

- [ ] TODO-043: Map Integration
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-041
  - Estimated Effort: 3 days
  - Details: [005-frontend.md](./005-frontend.md)

- [ ] TODO-044: Offline-First Data Layer (Flutter)
  - Status: PENDING
  - Owner: Mobile Dev
  - Dependencies: TODO-003
  - Estimated Effort: 3 days
  - Details: [005-frontend.md](./005-frontend.md)

---

## Phase 6: Integration (Week 9-10)

- [ ] TODO-050: Optional Command Post Backend
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-021
  - Estimated Effort: 2 days
  - Details: [006-integration.md](./006-integration.md)
  - **NOTE**: Only if Command Post has server capability

- [ ] TODO-051: End-to-End Workflow Integration
  - Status: PENDING
  - Owner: Full Team
  - Dependencies: TODO-034, TODO-044
  - Estimated Effort: 4 days
  - Details: [006-integration.md](./006-integration.md)

- [ ] TODO-052: Peer Trust Authentication
  - Status: PENDING
  - Owner: Security Dev
  - Dependencies: TODO-012
  - Estimated Effort: 3 days
  - Details: [006-integration.md](./006-integration.md)

- [ ] TODO-053: Two-Tier Data Classification
  - Status: PENDING
  - Owner: Security Dev
  - Dependencies: TODO-020
  - Estimated Effort: 2 days
  - Details: [006-integration.md](./006-integration.md)

- [ ] TODO-054: Role-Based Access Control
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-052
  - Estimated Effort: 2 days
  - Details: [006-integration.md](./006-integration.md)

- [ ] TODO-055: Audit Logging
  - Status: PENDING
  - Owner: Backend Dev
  - Dependencies: TODO-020
  - Estimated Effort: 2 days
  - Details: [006-integration.md](./006-integration.md)

---

## Phase 7: Testing & Demo (Week 11-12)

- [ ] TODO-060: Unit Test Suite
  - Status: PENDING
  - Owner: QA
  - Dependencies: TODO-021, TODO-031
  - Estimated Effort: 3 days
  - Details: [007-testing.md](./007-testing.md)

- [ ] TODO-061: Integration Test Suite
  - Status: PENDING
  - Owner: QA
  - Dependencies: TODO-051
  - Estimated Effort: 3 days
  - Details: [007-testing.md](./007-testing.md)

- [ ] TODO-062: Load Testing (50-100 nodes)
  - Status: PENDING
  - Owner: QA
  - Dependencies: TODO-061
  - Estimated Effort: 3 days
  - Details: [007-testing.md](./007-testing.md)

- [ ] TODO-063: Failure Mode Testing
  - Status: PENDING
  - Owner: QA
  - Dependencies: TODO-062
  - Estimated Effort: 2 days
  - Details: [007-testing.md](./007-testing.md)

- [ ] TODO-064: Demo Scenario Development
  - Status: PENDING
  - Owner: Product
  - Dependencies: TODO-051
  - Estimated Effort: 2 days
  - Details: [007-testing.md](./007-testing.md)

- [ ] TODO-065: Demo Environment Setup
  - Status: PENDING
  - Owner: DevOps
  - Dependencies: TODO-064
  - Estimated Effort: 2 days
  - Details: [007-testing.md](./007-testing.md)

- [ ] TODO-066: User Documentation
  - Status: PENDING
  - Owner: Tech Writer
  - Dependencies: TODO-051
  - Estimated Effort: 3 days
  - Details: [007-testing.md](./007-testing.md)

---

## Critical Specialist Corrections Applied

These corrections from framework specialists are incorporated throughout:

### DataFlow Corrections (see 003-data-layer.md)
1. Template syntax: Use `${}` NOT `{{}}`
2. No `created_at`/`updated_at` in model definitions (auto-managed)
3. Use `add_connection()` NOT `connect()`
4. Add `auto_discovery=False` when using Nexus + DataFlow together

### Kaizen Corrections (see 004-ai-agents.md)
1. BaseAgent init requires `config=` and `signature=` parameters
2. Create BaseEdgeAgent pattern for offline-first design
3. Call agents OUTSIDE workflows (PythonCode cannot await)
4. Use simple coordinator, not full multi-agent orchestration

### Nexus Corrections (see 006-integration.md)
1. Use `register()` NOT `register_workflow()`
2. Use `start()` NOT `run()`
3. Cannot run Python on mobile devices
4. Pure Flutter recommended for responder devices
5. Command Post backend is OPTIONAL

### Frontend Design (see 005-frontend.md)
1. Create design system FIRST (colors, typography, components)
2. 56dp minimum touch targets for gloved hands
3. Dark mode as default for outdoor visibility
4. Multi-modal feedback (visual + haptic + audio)

---

## Go/No-Go Gates

| Gate | Week | Criteria | Fallback |
|------|------|----------|----------|
| BLE Viability | 4 | 5+ nodes mesh working | Evaluate WiFi Direct |
| Core Features | 6 | CRUD + sync functional | Simplify data model |
| AI Integration | 8 | Task routing working | Deploy rule-based only |
| System Integration | 10 | E2E flows working | Focus on subset of flows |

---

## Success Criteria

| Criterion | Minimum | Target |
|-----------|---------|--------|
| Mesh nodes operational | 30 | 50-100 |
| Message delivery rate | 90% | 95%+ |
| Core features offline | 80% | 100% |
| Task completion rate | 70% | 85%+ |
| User satisfaction | Neutral | Positive |

---

## References

- Roadmap: `/docs/02-plans/00-poc-roadmap.md`
- Architecture: `/docs/02-plans/01-architecture-plan.md`
- Requirements: `/docs/01-analysis/00-requirements-summary.md`
