# PoC Roadmap - 3 Month Accelerated Timeline

## Overview

This roadmap outlines the development plan for the ADRC Proof of Concept over a 3-month accelerated timeline. The goal is to demonstrate technical feasibility of AI-coordinated disaster response over BLE mesh networks.

---

## 1. PoC Objectives

### 1.1 Primary Objectives

1. **Validate BLE Mesh Viability**: Demonstrate 50-100 node cluster operation
2. **Prove AI Coordination Concept**: Show task routing and situational awareness
3. **Demonstrate Offline Operation**: Core functionality without any connectivity
4. **Test Multi-Agency Model**: Simulate multi-agency coordination

### 1.2 Success Criteria

| Criterion | Minimum | Target |
|-----------|---------|--------|
| Mesh nodes operational | 30 | 50-100 |
| Message delivery rate | 90% | 95%+ |
| Core features offline | 80% | 100% |
| Task completion rate | 70% | 85%+ |
| User satisfaction | Neutral | Positive |

### 1.3 Non-Goals (Explicitly Out of Scope)

- Production security hardening
- Multi-cluster federation
- Full AI model deployment (use rules + stubs)
- Multi-language support
- Civilian-facing interface
- External system integration
- iOS native app (Android/Flutter focus)

---

## 2. Timeline Overview

```
Month 1                    Month 2                    Month 3
├────────────────────────┼────────────────────────────┼────────────────────────────┤
│ Week 1-2: Foundation   │ Week 5-6: Core Features    │ Week 9-10: Integration     │
│ Week 3-4: Network      │ Week 7-8: AI + UI          │ Week 11-12: Testing        │
├────────────────────────┴────────────────────────────┴────────────────────────────┤
│                            PoC Demo & Review                                    │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Month 1: Foundation & Network

### Week 1-2: Project Setup & Architecture

**Goals**:
- Development environment setup
- Architecture finalization
- BLE mesh library selection

**Deliverables**:
| Deliverable | Owner | Status |
|-------------|-------|--------|
| Dev environment (Python, Kailash SDK) | Tech Lead | |
| Mobile project setup (Flutter/Android) | Mobile Dev | |
| Architecture Decision Records (ADRs) | Tech Lead | |
| Data model design | Backend Dev | |
| BLE mesh library evaluation | Mobile Dev | |

**Key Decisions**:
- [ ] Mobile platform: Flutter vs Android native
- [ ] BLE mesh library: flutter_blue_plus, android BLE mesh
- [ ] Database: SQLite via DataFlow
- [ ] State sync: CRDT library selection

### Week 3-4: BLE Mesh Foundation

**Goals**:
- BLE mesh communication working
- Basic device discovery
- Message passing validated

**Deliverables**:
| Deliverable | Owner | Status |
|-------------|-------|--------|
| BLE mesh prototype (5-10 nodes) | Mobile Dev | |
| Device discovery service | Mobile Dev | |
| Message serialization format | Backend Dev | |
| Basic peer communication | Mobile Dev | |
| Range/reliability testing | QA | |

**Key Milestones**:
- [ ] Two devices communicate via BLE
- [ ] Five devices form mesh network
- [ ] Messages relay through intermediate nodes
- [ ] Basic reliability metrics captured

**Risk Checkpoint**: If BLE mesh doesn't work reliably by end of Week 4, escalate for architectural alternatives.

---

## 4. Month 2: Core Features

### Week 5-6: Data Layer & Core Workflows

**Goals**:
- DataFlow models implemented
- Core workflows operational
- CRDT sync functional

**Deliverables**:
| Deliverable | Owner | Status |
|-------------|-------|--------|
| Responder model + CRUD | Backend Dev | |
| Task model + CRUD | Backend Dev | |
| Incident model + CRUD | Backend Dev | |
| Resource model + CRUD | Backend Dev | |
| CRDT sync library integration | Backend Dev | |
| Mesh-to-DataFlow bridge | Backend Dev | |

**Core Workflows (Kailash)**:
```
1. Responder Status Update Workflow
   Input: status, location
   Output: Updated local DB + mesh broadcast

2. Task Assignment Workflow
   Input: task details, target responder
   Output: Task created, assigned, notified

3. Incident Report Workflow
   Input: incident details
   Output: Incident logged, nearby responders alerted

4. Resource Query Workflow
   Input: resource type, location
   Output: Available resources sorted by proximity
```

### Week 7-8: AI Agents & User Interface

**Goals**:
- Task routing agent functional
- Situation awareness display
- Basic mobile UI operational

**Deliverables**:
| Deliverable | Owner | Status |
|-------------|-------|--------|
| Task Routing Agent (Kaizen) | AI Dev | |
| Resource Matching Agent | AI Dev | |
| Situation Summary Agent | AI Dev | |
| Responder app UI (Flutter) | Mobile Dev | |
| Commander dashboard UI | Mobile Dev | |
| Map-based situational display | Mobile Dev | |

**AI Agent Specifications (Kaizen)**:
```
1. Task Routing Agent
   - Input: New task, available responders
   - Logic: Score by proximity, skills, workload
   - Output: Ranked responder recommendations
   - Fallback: Round-robin assignment

2. Resource Matching Agent
   - Input: Resource need, current inventory
   - Logic: Match type, check availability, proximity
   - Output: Resource allocation recommendation
   - Fallback: List available resources

3. Situation Summary Agent
   - Input: Recent events, status updates
   - Logic: Aggregate, detect patterns, summarize
   - Output: Textual situation summary
   - Fallback: Raw event list
```

---

## 5. Month 3: Integration & Testing

### Week 9-10: Integration & Security

**Goals**:
- All components integrated
- Two-tier security implemented
- End-to-end workflows working

**Deliverables**:
| Deliverable | Owner | Status |
|-------------|-------|--------|
| Mobile-Backend integration | Full Team | |
| BLE-AI-DataFlow integration | Tech Lead | |
| Peer trust authentication (basic) | Security Dev | |
| Two-tier data classification | Security Dev | |
| Role-based access control | Backend Dev | |
| Audit logging | Backend Dev | |

**Integration Test Scenarios**:
1. Responder joins network and receives tasks
2. Commander creates task, AI routes to responder
3. Responder completes task, status updates propagate
4. Network partitions, nodes continue operating
5. Partition heals, data syncs correctly

### Week 11-12: Testing & Demo Prep

**Goals**:
- System tested with 50+ nodes
- Demo scenario prepared
- Documentation complete

**Deliverables**:
| Deliverable | Owner | Status |
|-------------|-------|--------|
| Load testing (50-100 nodes) | QA | |
| Failure mode testing | QA | |
| Demo scenario script | Product | |
| Demo environment setup | DevOps | |
| User documentation | Tech Writer | |
| Lessons learned document | Tech Lead | |

**Demo Scenario**:
```
Earthquake Response Simulation (30-minute demo)
1. Setup: 50 devices, 5 teams, 2 sectors
2. Phase 1: Teams deploy, join mesh (5 min)
3. Phase 2: Incidents reported, tasks created (10 min)
4. Phase 3: AI routing demonstrated (5 min)
5. Phase 4: Network partition simulated (5 min)
6. Phase 5: Recovery and sync (5 min)
```

---

## 6. Resource Requirements

### 6.1 Team Structure

| Role | FTE | Responsibilities |
|------|-----|-----------------|
| Tech Lead | 1.0 | Architecture, Kailash SDK integration |
| Backend Developer | 1.0 | DataFlow, workflows, API |
| Mobile Developer | 1.5 | Flutter/Android, BLE mesh, UI |
| AI Developer | 0.5 | Kaizen agents, rule-based fallbacks |
| QA Engineer | 0.5 | Testing, validation |
| Product Owner | 0.25 | Requirements, stakeholder liaison |

### 6.2 Hardware Requirements

| Item | Quantity | Purpose |
|------|----------|---------|
| Android phones/tablets | 20-30 | Primary test devices |
| Rugged tablets | 5-10 | Realistic device testing |
| Development laptops | 4 | Developer workstations |
| Test area space | 1 | Indoor/outdoor mesh testing |

### 6.3 Software/Services

| Item | Purpose | Cost |
|------|---------|------|
| Kailash SDK | Core framework | Licensed |
| Flutter SDK | Mobile development | Free |
| Android Studio | IDE | Free |
| GitHub | Source control | Free/Paid |
| Firebase (optional) | Analytics only | Free tier |

---

## 7. Risk Management

### 7.1 Technical Risks

| Risk | Probability | Impact | Mitigation | Trigger |
|------|-------------|--------|------------|---------|
| BLE range insufficient | Medium | High | Test early, plan relay nodes | Week 4 |
| Battery drain excessive | Medium | Medium | Power profiling, optimize | Week 6 |
| AI too slow on device | Medium | Medium | Rule-based fallbacks | Week 8 |
| CRDT sync issues | Low | High | Use proven library | Week 6 |

### 7.2 Schedule Risks

| Risk | Probability | Impact | Mitigation | Trigger |
|------|-------------|--------|------------|---------|
| BLE library issues | Medium | High | Early library selection | Week 2 |
| Integration complexity | Medium | Medium | Continuous integration | Week 8 |
| Testing bottleneck | Medium | Medium | Parallel test dev | Week 10 |
| Scope creep | High | Medium | Strict scope control | Ongoing |

### 7.3 Go/No-Go Gates

| Gate | Timing | Criteria | Fallback |
|------|--------|----------|----------|
| BLE Viability | Week 4 | 5+ nodes mesh working | Evaluate WiFi Direct |
| Core Features | Week 6 | CRUD + sync functional | Simplify data model |
| AI Integration | Week 8 | Task routing working | Deploy rule-based only |
| System Integration | Week 10 | E2E flows working | Focus on subset of flows |

---

## 8. Stakeholder Engagement

### 8.1 Engagement Schedule

| Week | Activity | Stakeholders |
|------|----------|--------------|
| 2 | Architecture review | SCDF tech, SAF tech |
| 4 | BLE demo | Tech stakeholders |
| 6 | Mid-point review | All sponsors |
| 8 | Feature demo | All stakeholders |
| 10 | Pre-final demo | All stakeholders |
| 12 | Final demo | All stakeholders + leadership |

### 8.2 Feedback Integration

- Weekly stakeholder feedback collection
- Bi-weekly backlog refinement
- Change request process for scope changes
- Demo feedback → immediate backlog updates

---

## 9. Definition of Done

### 9.1 PoC Complete When:

- [ ] 50+ node mesh operates for 2+ hours
- [ ] All core workflows execute successfully
- [ ] AI task routing demonstrates value
- [ ] Offline operation fully functional
- [ ] Two-tier security implemented
- [ ] Demo scenario runs successfully
- [ ] Documentation complete
- [ ] Stakeholder feedback collected
- [ ] Lessons learned documented
- [ ] Roadmap for pilot phase proposed

### 9.2 Acceptance Criteria Sign-off

| Stakeholder | Role | Sign-off Date |
|-------------|------|---------------|
| Project Sponsor | Funding/Strategic | |
| SCDF Representative | Operational | |
| SAF Representative | Operational | |
| Technical Lead | Technical | |

---

## 10. Post-PoC Considerations

### 10.1 Decision Point After PoC

Based on PoC results, decide:
1. **Proceed to Pilot**: PoC successful, proceed with expanded pilot
2. **Pivot**: PoC revealed issues, address before pilot
3. **Stop**: PoC shows fundamental infeasibility

### 10.2 Pilot Phase Preview (If Proceeding)

**Scope**:
- 100-300 nodes
- Multi-cluster federation
- Real exercise deployment
- Enhanced security
- External system integration

**Duration**: 6-12 months

**Resources**: 2x PoC team
