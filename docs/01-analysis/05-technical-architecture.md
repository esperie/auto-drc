# Technical Architecture Analysis

## Overview

This document analyzes the technical architecture required for ADRC, focusing on the unique constraints of offline operation, BLE mesh networking, and edge AI processing.

---

## 1. Architecture Constraints

### 1.1 Hard Constraints

| Constraint | Requirement | Impact |
|------------|-------------|--------|
| No internet | Zero external connectivity | All processing local, no cloud |
| BLE Mesh | Primary communication | Limited bandwidth, range constraints |
| Mixed devices | Tablets + phones | Heterogeneous compute capability |
| Battery-powered | Extended operation | Power optimization critical |
| Mixed classification | Security levels | Data compartmentalization |

### 1.2 BLE Mesh Limitations

| Parameter | BLE Mesh Typical | ADRC Requirement | Gap Analysis |
|-----------|-----------------|------------------|--------------|
| Range per node | 10-100m (varies) | Building-scale | OK with relay |
| Nodes per network | 200-500 practical | 50-100 (PoC) | OK |
| Throughput | ~1 Mbps shared | Low data volume | OK |
| Latency | 100ms-2s | < 5s for status | OK |
| Power consumption | Low | Critical | Needs optimization |
| Penetration | Moderate | Rubble/structures | Challenge |

---

## 2. High-Level Architecture

### 2.1 System Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                     APPLICATION LAYER                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ Responder   │  │ Commander   │  │ Civilian    │            │
│  │ App         │  │ App         │  │ App         │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                      AI/ML LAYER                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ Task        │  │ Resource    │  │ Situation   │            │
│  │ Routing     │  │ Optimization│  │ Analysis    │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                    COORDINATION LAYER                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ Workflow    │  │ Data        │  │ Event       │            │
│  │ Engine      │  │ Sync        │  │ Bus         │            │
│  │ (Kailash)   │  │ (CRDTs)     │  │             │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                    DATA LAYER                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ Local       │  │ Mesh        │  │ Audit       │            │
│  │ Database    │  │ State       │  │ Log         │            │
│  │ (DataFlow)  │  │             │  │             │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                    NETWORK LAYER                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ BLE Mesh    │  │ Gateway     │  │ Sync        │            │
│  │ Protocol    │  │ Bridge      │  │ Protocol    │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                    SECURITY LAYER                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ Peer Trust  │  │ Encryption  │  │ Access      │            │
│  │ Auth        │  │             │  │ Control     │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Node Types

| Node Type | Hardware | Role | Count (PoC) |
|-----------|----------|------|-------------|
| Responder Node | Rugged phone/tablet | Field operations | 40-80 |
| Team Leader Node | Rugged tablet | Team coordination | 5-10 |
| Sector Node | Tablet + external antenna | Sector coordination, relay | 3-5 |
| Command Node | Tablet/laptop | Operations center | 1-3 |
| Sensor Node | IoT device | Data collection | 5-10 |
| Gateway Node | Dedicated device | Mesh bridging | 2-3 |

---

## 3. Component Architecture

### 3.1 Coordination Engine (Kailash SDK)

**Purpose**: Workflow orchestration, task routing, resource matching

**Architecture**:
```
┌─────────────────────────────────────────────────┐
│               Coordination Engine               │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │           Workflow Builder              │   │
│  │  • Task assignment workflows            │   │
│  │  • Resource allocation workflows        │   │
│  │  • Status aggregation workflows         │   │
│  └─────────────────────────────────────────┘   │
│                      │                         │
│  ┌─────────────────────────────────────────┐   │
│  │         AsyncLocalRuntime               │   │
│  │  • Async-first execution                │   │
│  │  • No external dependencies             │   │
│  │  • Cycle support for iterative tasks    │   │
│  └─────────────────────────────────────────┘   │
│                      │                         │
│  ┌─────────────────────────────────────────┐   │
│  │              Node Library               │   │
│  │  • PythonCode nodes for custom logic    │   │
│  │  • Data transform nodes                 │   │
│  │  • AI inference nodes                   │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

### 3.2 Data Layer (DataFlow)

**Purpose**: Local data persistence, offline-capable sync

**Architecture**:
```
┌─────────────────────────────────────────────────┐
│                   DataFlow                      │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │              Models                     │   │
│  │  • Responder (location, status, skills) │   │
│  │  • Task (assignment, progress, outcome) │   │
│  │  • Resource (type, location, quantity)  │   │
│  │  • Incident (location, severity, status)│   │
│  │  • Message (content, sender, timestamp) │   │
│  └─────────────────────────────────────────┘   │
│                      │                         │
│  ┌─────────────────────────────────────────┐   │
│  │         SQLite Local Database           │   │
│  │  • Per-device persistent storage        │   │
│  │  • Automatic node generation            │   │
│  │  • Express API for fast CRUD            │   │
│  └─────────────────────────────────────────┘   │
│                      │                         │
│  ┌─────────────────────────────────────────┐   │
│  │           Sync Engine (CRDT)            │   │
│  │  • Conflict-free replicated data        │   │
│  │  • Eventual consistency across mesh     │   │
│  │  • Causal ordering preservation         │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

### 3.3 AI Agent Layer (Kaizen)

**Purpose**: Intelligent decision support, task routing, situation analysis

**Architecture**:
```
┌─────────────────────────────────────────────────┐
│                  Kaizen Agents                  │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │         Task Routing Agent              │   │
│  │  • Match tasks to responder skills      │   │
│  │  • Optimize for proximity and load      │   │
│  │  • Priority-based queuing               │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │       Resource Allocation Agent         │   │
│  │  • Predict resource needs               │   │
│  │  • Optimize distribution across sites   │   │
│  │  • Alert on critical shortages          │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │       Situation Analysis Agent          │   │
│  │  • Aggregate multi-source data          │   │
│  │  • Detect anomalies and trends          │   │
│  │  • Generate situation summaries         │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │     Local LLM Runtime (Ollama-style)    │   │
│  │  • Quantized models (4-bit)             │   │
│  │  • Mobile-optimized inference           │   │
│  │  • Fallback to rule-based systems       │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

### 3.4 BLE Mesh Network Layer

**Architecture**:
```
┌─────────────────────────────────────────────────────────────────┐
│                      BLE Mesh Network                           │
│                                                                 │
│   Cluster A (Building 1)          Cluster B (Building 2)       │
│  ┌─────────────────────┐         ┌─────────────────────┐       │
│  │  ○ ─── ○ ─── ○     │   GW    │      ○ ─── ○        │       │
│  │  │     │     │     │ ═══════ │      │     │        │       │
│  │  ○ ─── ● ─── ○     │ Bridge  │      ○ ─── ●        │       │
│  │  │     │     │     │         │      │     │        │       │
│  │  ○ ─── ○ ─── ○     │         │      ○ ─── ○        │       │
│  └─────────────────────┘         └─────────────────────┘       │
│                                                                 │
│  ○ = Responder Node              ● = Sector/Relay Node         │
│  GW = Gateway Bridge             ═══ = Cross-cluster link      │
│                                                                 │
│  Protocol Stack:                                                │
│  ┌─────────────────────────────────────────────────┐           │
│  │ Application: ADRC Messages (JSON/Protobuf)      │           │
│  │ Transport: BLE Mesh Transport                   │           │
│  │ Network: BLE Mesh Routing                       │           │
│  │ Link: BLE Connection                            │           │
│  │ Physical: 2.4 GHz BLE Radio                     │           │
│  └─────────────────────────────────────────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Data Architecture

### 4.1 Core Data Models

```python
# Responder Model
@db.model
class Responder:
    id: str                    # Unique device/responder ID
    name: str                  # Responder name
    role: str                  # medic, usar, logistics, etc.
    team_id: str               # Team assignment
    status: str                # available, engaged, offline
    location_lat: float        # Last known latitude
    location_lng: float        # Last known longitude
    location_updated: str      # ISO timestamp
    skills: str                # JSON array of skill tags
    battery_level: int         # Device battery percentage

# Task Model
@db.model
class Task:
    id: str                    # Unique task ID
    type: str                  # rescue, medical, logistics, etc.
    priority: str              # critical, high, medium, low
    status: str                # pending, assigned, in_progress, completed
    location_lat: float
    location_lng: float
    description: str
    assigned_to: str           # Responder ID
    assigned_by: str           # Commander ID
    created_at: str
    updated_at: str
    completed_at: str
    outcome: str               # Result description

# Resource Model
@db.model
class Resource:
    id: str
    type: str                  # vehicle, medical, equipment, etc.
    subtype: str               # ambulance, stretcher, radio, etc.
    status: str                # available, deployed, depleted
    location_lat: float
    location_lng: float
    quantity: int
    assigned_to: str           # Task or responder ID

# Incident Model
@db.model
class Incident:
    id: str
    type: str                  # casualty, hazard, access_blocked, etc.
    severity: str              # critical, serious, moderate, minor
    status: str                # active, resolved, false_alarm
    location_lat: float
    location_lng: float
    description: str
    reported_by: str
    verified: bool
    created_at: str
    resolved_at: str
```

### 4.2 Data Synchronization Strategy

**Approach**: Conflict-free Replicated Data Types (CRDTs)

```
┌──────────────────────────────────────────────────────────────┐
│                    CRDT Sync Strategy                        │
│                                                              │
│  Node A                      Node B                         │
│  ┌─────────┐                ┌─────────┐                     │
│  │ Local   │ ─── Merge ───▶ │ Local   │                     │
│  │ State   │ ◀─── Merge ─── │ State   │                     │
│  └─────────┘                └─────────┘                     │
│                                                              │
│  Conflict Resolution Rules:                                  │
│  1. Last-Writer-Wins for simple values (with vector clock)   │
│  2. Union for set membership (responders in team)            │
│  3. Max for counters (incident count)                        │
│  4. Causal order for messages (happens-before)               │
│                                                              │
│  Sync Priorities:                                            │
│  1. Status updates (high frequency, small size)              │
│  2. Task assignments (medium frequency, critical)            │
│  3. Incident reports (event-driven, important)               │
│  4. Resource updates (medium frequency)                      │
│  5. Messages (variable, lower priority)                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Edge AI Architecture

### 5.1 Model Deployment Strategy

| Model Type | Size | Runtime | Purpose |
|------------|------|---------|---------|
| Task Router | 50-100MB | ONNX Mobile | Task-responder matching |
| Anomaly Detector | 10-20MB | TensorFlow Lite | Pattern detection |
| Text Classifier | 20-50MB | ONNX Mobile | Message categorization |
| Summarizer | 100-200MB | Quantized LLM | Report generation |

### 5.2 Compute Distribution

```
                   AI Compute Distribution

┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Responder Device        Team Leader         Command Post  │
│  ┌────────────────┐     ┌────────────────┐  ┌────────────┐│
│  │ • Classification│     │ • Task routing │  │ • Full LLM ││
│  │ • Local search  │     │ • Team summary │  │ • Analysis ││
│  │ • Quick actions │     │ • Aggregation  │  │ • Planning ││
│  │                 │     │ • Relay        │  │            ││
│  │ CPU: Light     │     │ CPU: Medium    │  │ CPU: Heavy ││
│  │ Memory: 2GB    │     │ Memory: 4GB    │  │ Memory: 8GB││
│  └────────────────┘     └────────────────┘  └────────────┘│
│                                                            │
│  Model Distribution:                                       │
│  ──────────────────                                       │
│  • All nodes: Classifier, Anomaly detector                │
│  • Team leader+: Task router, Summarizer (light)          │
│  • Command only: Full LLM, Planning models                │
└────────────────────────────────────────────────────────────┘
```

### 5.3 Fallback Strategy

When AI models cannot run (low battery, limited device):

1. **Rule-based fallback**: Pre-coded decision trees for common scenarios
2. **Human escalation**: Flag for commander decision
3. **Degraded mode**: Basic functionality without AI enhancement

---

## 6. Security Architecture

(Detailed in 06-security-cia-triad.md)

**Key Components**:
- Peer trust authentication
- End-to-end encryption for mesh messages
- Role-based access control
- Two-tier data classification
- Audit logging

---

## 7. API Design (Nexus)

### 7.1 Multi-Channel Deployment

```
┌─────────────────────────────────────────────────────────────┐
│                        Nexus Platform                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Unified Session Management             │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                    │                    │        │
│         ▼                    ▼                    ▼        │
│  ┌─────────────┐      ┌─────────────┐      ┌──────────┐   │
│  │ REST API    │      │ CLI         │      │ MCP      │   │
│  │ (Mobile)    │      │ (Desktop)   │      │ (AI)     │   │
│  └─────────────┘      └─────────────┘      └──────────┘   │
│                                                             │
│  Workflows registered once, exposed via all channels        │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Key API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /responders/status | POST | Update responder status |
| /responders/nearby | GET | Find nearby responders |
| /tasks | POST | Create new task |
| /tasks/{id}/assign | POST | Assign task to responder |
| /tasks/{id}/complete | POST | Mark task complete |
| /incidents | POST | Report new incident |
| /incidents/nearby | GET | Get nearby incidents |
| /resources | GET | List available resources |
| /sync | POST | Synchronize local state |

---

## 8. Technology Stack Summary

### 8.1 Core Technologies

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Framework | Kailash SDK | Workflow orchestration, offline-capable |
| Database | DataFlow + SQLite | Zero-config, mobile-friendly |
| AI Agents | Kaizen | Signature-based AI, multi-agent |
| API | Nexus | Multi-channel deployment |
| Mesh Network | Android BLE Mesh / iOS Core BT | Native platform support |
| Edge AI | ONNX Runtime / TF Lite | Optimized mobile inference |
| Sync | Custom CRDT library | Offline-first sync |

### 8.2 Development Languages

| Component | Language | Framework |
|-----------|----------|-----------|
| Backend/Agents | Python 3.11+ | Kailash SDK |
| Mobile (Android) | Kotlin | Jetpack Compose |
| Mobile (iOS) | Swift | SwiftUI |
| Cross-platform option | Flutter | Dart |
| AI Models | Python | PyTorch → ONNX |

---

## 9. PoC Scope Technical Boundaries

### 9.1 In Scope (PoC)

- [ ] BLE mesh networking (single cluster, 50-100 nodes)
- [ ] Core data models (Responder, Task, Incident, Resource)
- [ ] Basic CRDT sync for offline operation
- [ ] Task assignment workflow with simple routing
- [ ] Status tracking and situational display
- [ ] Two-tier security (unclassified + sensitive)
- [ ] Mobile app (Android or cross-platform)
- [ ] Simulated AI (rule-based with AI model stubs)

### 9.2 Out of Scope (PoC)

- Multi-cluster federation (gateway bridging)
- Full AI model deployment (use rule-based fallbacks)
- iOS native app (use cross-platform or Android-only)
- Integration with external systems (AHA Centre, etc.)
- Production security hardening
- Multi-language support (English only for PoC)
- Civilian-facing interface
- Post-connectivity sync to cloud
