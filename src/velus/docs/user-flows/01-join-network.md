# User Flow: Join Network

## Summary

This flow describes how a responder joins the ADRC BLE mesh network upon arriving at the disaster site. The process establishes identity, authenticates via peer trust, and enables mesh communication.

---

## Actor(s)

- **Primary**: Any responder (medic, USAR, logistics, command)
- **Secondary**: Existing mesh peer (for trust attestation)

---

## Trigger

- App launched in proximity to active mesh network
- OR Manual "Join Network" action

---

## Preconditions

1. Device has ADRC app installed
2. Bluetooth is enabled
3. Device has pre-loaded credentials (pre-deployment provisioning)
4. At least one mesh node is within BLE range

---

## Main Flow

### 1. App Launch
```
┌────────────────────────────────────────┐
│                                        │
│           ADRC                         │
│     Disaster Response                  │
│                                        │
│     ┌──────────────────────┐          │
│     │   🔵 Scanning...     │          │
│     │                      │          │
│     │   Looking for        │          │
│     │   nearby networks    │          │
│     └──────────────────────┘          │
│                                        │
└────────────────────────────────────────┘
```
- App opens to scanning screen
- BLE scan initiates automatically
- Shows "Scanning for networks..."

### 2. Network Discovery
```
┌────────────────────────────────────────┐
│                                        │
│  Networks Found                        │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ 📡 ADRC-Sector-A                 │ │
│  │    12 nodes | Signal: Strong     │ │
│  │    [Join]                        │ │
│  └──────────────────────────────────┘ │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ 📡 ADRC-Sector-B                 │ │
│  │    5 nodes | Signal: Weak        │ │
│  │    [Join]                        │ │
│  └──────────────────────────────────┘ │
│                                        │
│            [Create New Network]        │
│                                        │
└────────────────────────────────────────┘
```
- Display discovered mesh networks
- Show node count and signal strength
- User taps "Join" on appropriate network

### 3. Credential Presentation
```
┌────────────────────────────────────────┐
│                                        │
│  Joining ADRC-Sector-A                 │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │                                  │ │
│  │    🔐 Presenting credentials...  │ │
│  │                                  │ │
│  │    ████████░░░░░░░░ 50%         │ │
│  │                                  │ │
│  └──────────────────────────────────┘ │
│                                        │
│  Your pre-loaded certificate is       │
│  being verified by the network.       │
│                                        │
└────────────────────────────────────────┘
```
- System presents pre-loaded certificate to nearest node
- Certificate contains: device ID, agency, pre-signed attestation

### 4. Peer Trust Verification

**If certificate is directly trusted (pre-provisioned):**
- Automatic acceptance
- Skip to step 5

**If certificate requires peer vouching:**
```
┌────────────────────────────────────────┐
│                                        │
│  Peer Verification Required            │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │                                  │ │
│  │  👤 SGT Tan Wei Ming             │ │
│  │     SCDF - Sector A Lead         │ │
│  │                                  │ │
│  │  Can verify your identity to     │ │
│  │  join the network.               │ │
│  │                                  │ │
│  │     [Request Verification]       │ │
│  │                                  │ │
│  └──────────────────────────────────┘ │
│                                        │
│         [Show other verifiers]         │
│                                        │
└────────────────────────────────────────┘
```
- Show available peers who can vouch
- User requests verification from peer
- Peer receives verification request on their device
- Peer confirms (face-to-face visual confirmation recommended)

### 5. Role Selection
```
┌────────────────────────────────────────┐
│                                        │
│  ✓ Connected to ADRC-Sector-A         │
│                                        │
│  Select your role:                     │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ 🚑 Medic                         │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🔧 USAR                          │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 📦 Logistics                     │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 📋 Team Leader                   │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ 🎖 Sector Commander              │ │
│  └──────────────────────────────────┘ │
│                                        │
└────────────────────────────────────────┘
```
- User selects operational role
- Role determines UI mode and permissions

### 6. Team Assignment
```
┌────────────────────────────────────────┐
│                                        │
│  Select your team:                     │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │ Alpha Team                       │ │
│  │   TL: CPT Lee | 8 members        │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ Bravo Team                       │ │
│  │   TL: SGT Tan | 6 members        │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ Charlie Team                     │ │
│  │   TL: LT Wong | 4 members        │ │
│  └──────────────────────────────────┘ │
│                                        │
│         [No team / Unassigned]        │
│                                        │
└────────────────────────────────────────┘
```
- Display available teams
- User selects team OR remains unassigned (for command roles)

### 7. Join Complete
```
┌────────────────────────────────────────┐
│                                        │
│  ✓ You're Connected!                   │
│                                        │
│  ┌──────────────────────────────────┐ │
│  │                                  │ │
│  │  📡 ADRC-Sector-A               │ │
│  │  👤 LCP Ahmad (Medic)           │ │
│  │  👥 Alpha Team                  │ │
│  │  📍 Location sharing: ON        │ │
│  │                                  │ │
│  └──────────────────────────────────┘ │
│                                        │
│         [Go to Dashboard →]           │
│                                        │
└────────────────────────────────────────┘
```
- Confirmation screen
- Summary of connection details
- Proceed to main dashboard

---

## Alternative Flows

### A1: No Networks Found
- Display "No networks in range"
- Offer "Create New Network" option
- Show troubleshooting tips (enable Bluetooth, move closer)

### A2: Pre-Assigned Team
- If device has pre-configured team assignment, skip team selection
- Show confirmation of pre-assigned team

### A3: Commander Joining
- Additional confirmation for commander roles
- Option to claim/release sector command

---

## Exception Flows

### E1: Credential Rejected
```
┌────────────────────────────────────────┐
│                                        │
│  ❌ Connection Failed                  │
│                                        │
│  Your credentials were not accepted    │
│  by the network.                       │
│                                        │
│  This may happen if:                   │
│  • Credentials expired                 │
│  • Wrong deployment area               │
│  • Certificate revoked                 │
│                                        │
│     [Contact Command]  [Retry]         │
│                                        │
└────────────────────────────────────────┘
```
- Clear error message
- Guidance on resolution
- Option to contact command or retry

### E2: Peer Verification Timeout
- If peer doesn't respond in 60 seconds
- Show timeout message
- Option to request from different peer

### E3: Network Disconnected During Join
- If mesh connection lost mid-join
- Auto-retry connection
- Show "Reconnecting..." status

---

## Postconditions

1. Device is connected to mesh network
2. Responder identity established in network
3. Role and team assigned
4. Location sharing enabled
5. Ready to receive tasks and report status

---

## UI Mockup Reference

See: `designs/mobile/01-join-network-flow.fig` (future)

---

## Data Changes

### Responder Record Created
```json
{
  "id": "device-uuid-123",
  "name": "LCP Ahmad bin Hassan",
  "role": "medic",
  "team_id": "alpha",
  "agency": "scdf",
  "status": "available",
  "location_lat": 1.3521,
  "location_lng": 103.8198,
  "skills": ["cpr", "trauma", "triage"],
  "battery_level": 85,
  "last_seen_at": "2024-01-15T09:30:00Z"
}
```

---

## Mesh Messages

### MSG: Join Announcement
```json
{
  "type": "responder_joined",
  "responder_id": "device-uuid-123",
  "name": "LCP Ahmad bin Hassan",
  "role": "medic",
  "team_id": "alpha",
  "agency": "scdf",
  "timestamp": "2024-01-15T09:30:00Z"
}
```
- Broadcast to all mesh nodes
- Team leader and commander dashboards update

### MSG: Trust Attestation (if peer vouching)
```json
{
  "type": "trust_attestation",
  "voucher_id": "device-uuid-456",
  "subject_id": "device-uuid-123",
  "trust_level": "verified",
  "signature": "ed25519-signature...",
  "timestamp": "2024-01-15T09:29:55Z"
}
```
- Recorded for audit trail
- Enables subject's network participation

---

## Security Considerations

1. **Pre-provisioned credentials**: Reduces risk of impersonation
2. **Peer trust chain**: Allows field verification without central authority
3. **Trust level tracking**: 1-hop vs 2-hop trust recorded
4. **Audit logging**: All join events logged for post-incident review
