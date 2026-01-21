# Security Analysis - CIA Triad

## Overview

This document analyzes security requirements for ADRC based on the CIA Triad (Confidentiality, Integrity, Availability) with specific focus on the unique challenges of offline, mesh-networked, multi-agency disaster response operations.

---

## 1. Threat Model

### 1.1 Threat Actors

| Actor | Motivation | Capability | Likelihood |
|-------|------------|------------|------------|
| Opportunistic criminal | Exploit chaos for theft | Low-Medium | Medium |
| Hostile state actor | Intelligence gathering | High | Low (during disaster) |
| Insider threat | Personal gain, ideology | Medium | Low |
| Accidental exposure | Human error | N/A | High |
| Technical failure | System bugs, corruption | N/A | Medium |

### 1.2 Attack Surfaces

| Surface | Risk Level | Exposure |
|---------|------------|----------|
| BLE Mesh communications | High | All messages in transit |
| Device physical access | High | Lost/stolen devices |
| Local database | Medium | Device compromise |
| AI model inference | Low | Adversarial inputs |
| User interface | Medium | Social engineering |

### 1.3 Assets to Protect

| Asset | Classification | Impact if Compromised |
|-------|---------------|----------------------|
| Responder locations | Sensitive | Safety risk, operational compromise |
| Casualty information | Sensitive (PII) | Privacy violation, distress |
| Tactical plans | Sensitive | Operational compromise |
| Resource locations | Unclassified | Theft risk |
| Communication logs | Mixed | Intelligence value |
| AI recommendations | Sensitive | Decision manipulation |

---

## 2. CONFIDENTIALITY

### 2.1 Data Classification Scheme

**Two-Tier Model (PoC Scope)**:

| Tier | Classification | Examples | Access |
|------|---------------|----------|--------|
| Tier 1 | Unclassified | Shelter locations, public advisories, general resource info | All users |
| Tier 2 | Sensitive | Responder locations, tactical plans, PII, medical data | Role-based |

### 2.2 Confidentiality Controls

#### A. Data at Rest Encryption

**Requirement**: All local databases encrypted with device-bound keys

**Implementation**:
```
┌────────────────────────────────────────────────────────┐
│              Device Data Encryption                    │
│                                                        │
│  ┌────────────────┐     ┌────────────────┐           │
│  │ SQLite DB      │ ←── │ SQLCipher      │           │
│  │ (encrypted)    │     │ Encryption     │           │
│  └────────────────┘     └────────────────┘           │
│           ↑                     ↑                     │
│  ┌────────────────┐     ┌────────────────┐           │
│  │ Device Key     │ ←── │ Hardware       │           │
│  │ (derived)      │     │ Keystore       │           │
│  └────────────────┘     └────────────────┘           │
│                                                        │
│  Key Derivation:                                       │
│  • Android: Keystore-backed AES-256-GCM               │
│  • iOS: Secure Enclave + Keychain                     │
└────────────────────────────────────────────────────────┘
```

#### B. Data in Transit Encryption

**Requirement**: All BLE mesh messages encrypted end-to-end

**Implementation**:
```
┌────────────────────────────────────────────────────────┐
│           BLE Mesh Message Encryption                  │
│                                                        │
│  Sender                              Receiver          │
│  ┌─────────────┐                ┌─────────────┐       │
│  │ Plaintext   │                │ Plaintext   │       │
│  └──────┬──────┘                └──────▲──────┘       │
│         │                               │              │
│         ▼                               │              │
│  ┌─────────────┐                ┌─────────────┐       │
│  │ Encrypt     │                │ Decrypt     │       │
│  │ (E2E key)   │                │ (E2E key)   │       │
│  └──────┬──────┘                └──────▲──────┘       │
│         │                               │              │
│         ▼                               │              │
│  ┌─────────────┐                ┌─────────────┐       │
│  │ Encrypt     │────────────────│ Decrypt     │       │
│  │ (Mesh key)  │   BLE Mesh     │ (Mesh key)  │       │
│  └─────────────┘                └─────────────┘       │
│                                                        │
│  Two-layer encryption:                                 │
│  1. Mesh layer: Network-wide key for routing          │
│  2. E2E layer: Pairwise/group keys for content        │
└────────────────────────────────────────────────────────┘
```

**Key Management**:
- Mesh network key: Pre-provisioned, rotated periodically
- E2E keys: Derived from peer trust attestation chain
- Group keys: For broadcast to specific roles/teams

#### C. Access Control

**Requirement**: Role-based access to sensitive data

**Implementation**:
```
┌────────────────────────────────────────────────────────┐
│              Role-Based Access Control                 │
│                                                        │
│  Role Hierarchy:                                       │
│                                                        │
│  ┌─────────────────────────────────────────────┐      │
│  │               Command                        │      │
│  │  (Full access to Tier 1 & Tier 2 data)      │      │
│  └─────────────────────────────────────────────┘      │
│                        │                               │
│        ┌───────────────┴───────────────┐              │
│        ▼                               ▼              │
│  ┌─────────────┐                ┌─────────────┐      │
│  │ Team Lead   │                │ Team Lead   │      │
│  │ (Team T1+T2)│                │ (Team T1+T2)│      │
│  └─────────────┘                └─────────────┘      │
│        │                               │              │
│        ▼                               ▼              │
│  ┌─────────────┐                ┌─────────────┐      │
│  │ Responder   │                │ Responder   │      │
│  │ (Own T2,    │                │ (Own T2,    │      │
│  │  Team T1)   │                │  Team T1)   │      │
│  └─────────────┘                └─────────────┘      │
│                                                        │
│  Data Access Matrix:                                   │
│  ─────────────────                                    │
│  │ Data            │ Cmd │ TL  │ Resp │ Civilian │   │
│  │─────────────────│─────│─────│──────│──────────│   │
│  │ Own location    │ R   │ R   │ RW   │ -        │   │
│  │ Team locations  │ R   │ R   │ R    │ -        │   │
│  │ All locations   │ RW  │ -   │ -    │ -        │   │
│  │ Tactical plans  │ RW  │ R   │ -    │ -        │   │
│  │ Public info     │ RW  │ RW  │ R    │ R        │   │
└────────────────────────────────────────────────────────┘
```

### 2.3 Confidentiality Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Lost/stolen device | Data exposure | Remote wipe capability, device encryption |
| BLE eavesdropping | Message interception | E2E encryption |
| Insider access abuse | Data leak | Audit logging, need-to-know access |
| AI inference from patterns | Intelligence leak | Metadata minimization |

---

## 3. INTEGRITY

### 3.1 Data Integrity Requirements

| Data Type | Integrity Need | Verification Method |
|-----------|---------------|---------------------|
| Task assignments | Critical | Digital signature by assigner |
| Status updates | High | Signed by device key |
| Incident reports | High | Signed, with corroboration scoring |
| Resource counts | Medium | Reconciliation process |
| AI recommendations | Medium | Explainability, override audit |

### 3.2 Integrity Controls

#### A. Message Authentication

**Requirement**: All messages authenticated to prevent tampering/spoofing

**Implementation**:
```
┌────────────────────────────────────────────────────────┐
│            Message Authentication                      │
│                                                        │
│  Message Structure:                                    │
│  ┌─────────────────────────────────────────────────┐  │
│  │ Header    │ Payload      │ Signature           │  │
│  │ (sender,  │ (encrypted   │ (Ed25519 over       │  │
│  │  seq#,    │  content)    │  header+payload)    │  │
│  │  time)    │              │                     │  │
│  └─────────────────────────────────────────────────┘  │
│                                                        │
│  Verification:                                         │
│  1. Check signature against sender's public key        │
│  2. Verify sender is in trust network                  │
│  3. Check sequence number for replay prevention        │
│  4. Validate timestamp within acceptable window        │
└────────────────────────────────────────────────────────┘
```

#### B. Peer Trust Authentication

**Requirement**: Verify identity without central authority (offline)

**Implementation**:
```
┌────────────────────────────────────────────────────────┐
│           Peer Trust Network                           │
│                                                        │
│  Pre-Deployment:                                       │
│  ┌─────────────┐     ┌─────────────┐                  │
│  │ Trust       │     │ Responder   │                  │
│  │ Authority   │────▶│ Certificate │                  │
│  │ (offline)   │     │ (signed)    │                  │
│  └─────────────┘     └─────────────┘                  │
│                                                        │
│  In-Field Verification:                                │
│  ┌─────────────┐                ┌─────────────┐       │
│  │ Alice       │◀───────────────│ Bob         │       │
│  │ (verified)  │  Challenge/    │ (new)       │       │
│  │             │  Response      │             │       │
│  └─────────────┘                └─────────────┘       │
│         │                               │              │
│         ▼                               ▼              │
│  ┌─────────────────────────────────────────────┐      │
│  │ Alice vouches for Bob → Attestation Chain   │      │
│  │ Bob can now participate with Alice's trust  │      │
│  └─────────────────────────────────────────────┘      │
│                                                        │
│  Trust Levels:                                         │
│  • Direct: Certified by Trust Authority               │
│  • 1-hop: Vouched by directly trusted peer            │
│  • 2-hop: Vouched by 1-hop peer (limited access)      │
└────────────────────────────────────────────────────────┘
```

#### C. CRDT Integrity

**Requirement**: Prevent malicious data corruption during sync

**Implementation**:
- Signed operations (each update signed by originator)
- Causal ordering (vector clocks prevent reordering attacks)
- Conflict resolution rules (deterministic, auditable)
- Reconciliation checkpoints (periodic state verification)

### 3.3 Integrity Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Message spoofing | False orders/reports | Cryptographic signatures |
| Data tampering | Corrupted decisions | Hash chains, signed updates |
| Replay attacks | Duplicate actions | Sequence numbers, timestamps |
| Sybil attacks | Fake responders | Peer trust with attestation |
| AI manipulation | Wrong recommendations | Input validation, explainability |

---

## 4. AVAILABILITY

### 4.1 Availability Requirements

| Service | Availability Target | Max Downtime |
|---------|--------------------| -------------|
| Local app operation | 99.9% | < 1 min/day |
| Mesh communication | 95% | Graceful degradation |
| Data sync | Best effort | Store-and-forward |
| AI services | 90% | Fallback to rules |
| Command dashboard | 99% | Critical for ops |

### 4.2 Availability Controls

#### A. Mesh Network Resilience

**Requirement**: Network continues functioning as nodes fail

**Implementation**:
```
┌────────────────────────────────────────────────────────┐
│           Mesh Resilience Architecture                 │
│                                                        │
│  Before Node Failure:          After Node Failure:     │
│  ┌───────────────────┐        ┌───────────────────┐   │
│  │  A ─── B ─── C    │        │  A ─────── C      │   │
│  │  │     │     │    │   →    │  │         │      │   │
│  │  D ─── E ─── F    │        │  D ─── E ─ F      │   │
│  │        ↑          │        │              ↑     │   │
│  │     (fails)       │        │   (reroutes)      │   │
│  └───────────────────┘        └───────────────────┘   │
│                                                        │
│  Resilience Mechanisms:                                │
│  • Multi-path routing (no single point of failure)    │
│  • Automatic path rediscovery on node loss            │
│  • Message queuing during temporary disconnection     │
│  • Graceful degradation to local-only mode            │
└────────────────────────────────────────────────────────┘
```

#### B. Local-First Architecture

**Requirement**: Core functionality works with zero network

**Implementation**:
```
┌────────────────────────────────────────────────────────┐
│           Connectivity Modes                           │
│                                                        │
│  Full Mesh:                                            │
│  • Real-time sync                                      │
│  • AI distributed processing                           │
│  • Full situational awareness                          │
│                        │                               │
│                        │ (connectivity degrades)       │
│                        ▼                               │
│  Partial Mesh:                                         │
│  • Store-and-forward sync                              │
│  • Local AI processing                                 │
│  • Partial situational awareness                       │
│                        │                               │
│                        │ (connectivity lost)           │
│                        ▼                               │
│  Isolated:                                             │
│  • Local data only                                     │
│  • Queue updates for later sync                        │
│  • Core task management functional                     │
│  • Last-known situational data                         │
│                                                        │
│  All modes: Core app functionality preserved           │
└────────────────────────────────────────────────────────┘
```

#### C. Redundancy Strategy

**Requirement**: No single point of failure for critical functions

**Implementation**:

| Component | Redundancy Strategy |
|-----------|---------------------|
| Data | Replicated across all mesh nodes |
| Command authority | Multiple designated commanders |
| Gateway bridges | Minimum 2 per cluster boundary |
| Sector nodes | Any team leader device can assume |
| Power | Backup battery packs pre-positioned |

#### D. Device Recovery

**Requirement**: Recover from device loss/failure quickly

**Implementation**:
- **Replacement provisioning**: New device joins via peer trust
- **State recovery**: Sync from mesh restores operational state
- **Role reassignment**: Automatic or manual role transfer
- **Audit continuity**: Actions logged even during recovery

### 4.3 Availability Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Mass device failure | Coordination collapse | Redundancy, manual fallback |
| Network partition | Split-brain operation | Partition-tolerant design |
| Battery depletion | Device offline | Power management, backups |
| Jamming/interference | Comms disruption | Frequency hopping, fallback |
| DDoS on mesh | Network overload | Rate limiting, priority queuing |

---

## 5. Security Architecture Summary

### 5.1 Security Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Security Stack                           │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ APPLICATION SECURITY                                │   │
│  │ • Role-based access control                         │   │
│  │ • Input validation                                  │   │
│  │ • Secure session management                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ DATA SECURITY                                       │   │
│  │ • Two-tier classification                           │   │
│  │ • SQLCipher database encryption                     │   │
│  │ • Signed CRDT operations                            │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ COMMUNICATION SECURITY                              │   │
│  │ • End-to-end encryption (E2E)                       │   │
│  │ • Message authentication (Ed25519)                  │   │
│  │ • BLE mesh security (network key)                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ IDENTITY & ACCESS                                   │   │
│  │ • Pre-provisioned certificates                      │   │
│  │ • Peer trust attestation chain                      │   │
│  │ • Trust level enforcement                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ DEVICE SECURITY                                     │   │
│  │ • Hardware-backed key storage                       │   │
│  │ • Remote wipe capability                            │   │
│  │ • Secure boot (device level)                        │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 PoC Security Scope

| Feature | PoC Implementation | Production Enhancement |
|---------|-------------------|----------------------|
| Authentication | Pre-shared certs + basic peer vouch | Full PKI + biometric |
| Encryption at rest | SQLCipher AES-256 | Hardware security module |
| Encryption in transit | BLE mesh + simple E2E | Full forward secrecy |
| Access control | Role-based (4 roles) | Fine-grained + temporal |
| Audit logging | Local logs | Tamper-proof distributed log |
| Compliance | Basic PDPA awareness | Full certification |

### 5.3 Security Testing Requirements

| Test Type | Scope | Timing |
|-----------|-------|--------|
| Penetration testing | BLE, API, device | Post-PoC |
| Code security review | All custom code | During development |
| Threat modeling | Full architecture | Before PoC |
| Compliance audit | Data handling | Before pilot |
