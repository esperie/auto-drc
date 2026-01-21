# Requirements Summary - Autonomous Disaster Response Coordination (ADRC)

## Overview

The ADRC platform is an AI-coordinated disaster response system designed for deployment in Southeast Asia (SEA) from the Regional HADR Centre in Singapore. The system operates on local Bluetooth Low Energy (BLE) mesh networks in disconnected environments where traditional communications infrastructure is unavailable.

## Core Requirements

### 1. System Context

| Attribute | Value |
|-----------|-------|
| Base Location | Regional HADR Centre, Singapore |
| Coverage Area | Southeast Asia (SEA) |
| Deployment Type | Multi-agency coalition (military, government civil defense, humanitarian NGOs) |
| Network Technology | Bluetooth Low Energy (BLE) Mesh |
| Hardware | Rugged tablets + smartphones |

### 2. Key Constraints

1. **No Internet/Comms**: Disaster areas have no connectivity; system must operate fully offline
2. **Local BLE Distribution**: All communication via BLE mesh network
3. **Security (CIA Triad)**: Mixed data classification levels, peer trust authentication
4. **ASEAN Compliance**: Must integrate with ASEAN AHA Centre protocols

### 3. AI Autonomy Model

**Tiered by Urgency**: AI autonomy level varies based on time-sensitivity and impact:

| Urgency Level | AI Role | Human Role |
|---------------|---------|------------|
| Critical (seconds) | Autonomous decision with logging | Post-hoc review |
| High (minutes) | Strong recommendation | Quick approval/override |
| Medium (hours) | Multiple options presented | Deliberate decision |
| Low (days) | Analysis and suggestions | Full planning cycle |

### 4. Platform Model

**Producers** (Value Creators):
- Field responders: Ground-truth situational data, status updates
- Command centers: Decisions, resource allocation, operational orders
- Sensor networks: Automated IoT/drone data streams

**Consumers** (Value Users):
- Affected population: Evacuation routes, shelter locations, assistance requests
- Operational responders: Task assignments, coordination info, logistics
- Command leadership: Situational awareness, decision support

**Core Transaction**: Combined orchestration integrating:
- Information exchange (situational awareness)
- Task coordination (assignments, tracking, completion)
- Resource matching (needs vs. capabilities)

### 5. Scale & Scope

| Parameter | Target | PoC Scope |
|-----------|--------|-----------|
| Responder capacity | 5000+ (mega disaster) | 50-100 nodes (cluster demo) |
| Disaster types | All-hazards approach | Single scenario validation |
| Duration offline | 0-12 hours | 2-4 hour exercise |
| Security levels | Mixed classification | Two-tier (unclassified + sensitive) |

### 6. Timeline

- **MVP Type**: Proof of Concept
- **Timeline**: 3 months (accelerated)
- **Goal**: Demonstrate BLE mesh + AI coordination technically works

### 7. Integration Requirements

| Framework | Requirement |
|-----------|-------------|
| ASEAN AHA Centre | Primary - must integrate with protocols |
| UN OCHA | Future consideration |
| NATO STANAG | Future consideration |

## Technical Architecture Implications

### BLE Mesh Considerations

Standard BLE Mesh supports ~200-500 nodes reliably per network. For 5000+ scale:
- **Federated architecture**: Multiple mesh clusters with gateway bridges
- **Hierarchical topology**: Local clusters → sector gateways → command mesh
- **PoC Focus**: Single cluster (50-100 nodes), document federation design

### Security Architecture

For two-tier demo:
- **Tier 1 (Unclassified)**: Public safety info, resource locations, evacuation routes
- **Tier 2 (Sensitive)**: Responder locations, tactical movements, PII

Authentication via peer trust network:
- Pre-provisioned trust anchors
- Verified responders vouch for newcomers
- Cryptographic attestation chain

### Data Synchronization

Sync-and-merge strategy when connectivity restored:
- Conflict resolution protocols (vector clocks, CRDTs)
- Audit trails for all offline decisions
- Authoritative source determination rules

## Next Steps

1. Document Value Propositions and USPs (see 01-value-propositions.md)
2. AAA Framework Evaluation (see 02-aaa-framework.md)
3. Network Effects Analysis (see 03-network-effects.md)
4. Stakeholder Analysis (see 04-stakeholder-analysis.md)
5. Technical Architecture (see 05-technical-architecture.md)
6. Security Analysis (see 06-security-cia-triad.md)
7. Scenario Critique (see 07-scenario-critique.md)
