# TODO-050 to TODO-055: Integration Phase (Week 9-10)

**Phase**: 6 - Integration
**Timeline**: Week 9-10
**Status**: PENDING

**CRITICAL CORRECTIONS FROM NEXUS SPECIALIST**:
1. Use `register()` NOT `register_workflow()`
2. Use `start()` NOT `run()`
3. Cannot run Python on mobile devices
4. Pure Flutter recommended for responder devices
5. Command Post backend is OPTIONAL (only if server available)
6. Add `auto_discovery=False` when using Nexus + DataFlow together

---

## TODO-050: Optional Command Post Backend

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-021 (Core Workflows)

### Description

Implement optional backend server for Command Post that provides centralized APIs. This is ONLY deployed if Command Post has a server capability. Responder devices use pure Flutter without Python.

### Architecture Decision

```
┌─────────────────────────────────────────────────────────────────┐
│                    ADRC Architecture                             │
│                                                                  │
│  ┌─────────────────────┐         ┌─────────────────────┐       │
│  │  Command Post       │◄───────►│  Responder Devices  │       │
│  │  (Optional Server)  │   BLE   │  (Pure Flutter)     │       │
│  │                     │   Mesh  │                     │       │
│  │  • Nexus API        │         │  • NO Python        │       │
│  │  • Kaizen Agents    │         │  • Local SQLite     │       │
│  │  • DataFlow DB      │         │  • BLE Mesh         │       │
│  └─────────────────────┘         └─────────────────────┘       │
│                                                                  │
│  NOTE: Command Post backend is OPTIONAL.                        │
│  System works fully with mesh-only (peer-to-peer).             │
└─────────────────────────────────────────────────────────────────┘
```

### Acceptance Criteria

- [ ] Nexus platform configured correctly
- [ ] DataFlow integration with `auto_discovery=False`
- [ ] All workflows registered with `register()`
- [ ] Server starts with `start()` not `run()`
- [ ] Health check endpoint
- [ ] Graceful shutdown handling
- [ ] Local-only binding (127.0.0.1) for security

### Corrected Nexus Implementation

```python
# src/velus/api/platform.py
from nexus import Nexus
from dataflow import DataFlow
from velus.workflows import (
    create_status_update_workflow,
    create_task_assignment_workflow,
    create_incident_report_workflow,
    create_resource_query_workflow
)
from velus.agents.orchestrator import AgentOrchestrator
import logging

logger = logging.getLogger(__name__)


def create_command_post_platform():
    """
    Create Nexus platform for Command Post backend.

    CRITICAL: This is OPTIONAL. Only deploy if Command Post has server.
    Responder devices do NOT need this - they use pure Flutter + BLE mesh.
    """

    # Initialize DataFlow with auto_discovery=False for Nexus integration
    # CRITICAL: auto_discovery=False prevents conflicts
    db = DataFlow("sqlite:///command_post.db", auto_discovery=False)

    # Create models (must be done before Nexus registration)
    from velus.models import Responder, Task, Incident, Resource
    db.register_model(Responder)
    db.register_model(Task)
    db.register_model(Incident)
    db.register_model(Resource)

    # Initialize Nexus platform
    platform = Nexus("ADRC Command Post")

    # Initialize agent orchestrator
    orchestrator = AgentOrchestrator(db)

    # Register workflows using register() NOT register_workflow()
    # CORRECT:
    platform.register(
        "status_update",
        create_status_update_workflow(),
        description="Update responder status and location"
    )

    platform.register(
        "task_assign",
        create_task_assignment_workflow(),
        description="Create and assign a task"
    )

    platform.register(
        "incident_report",
        create_incident_report_workflow(),
        description="Report a new incident"
    )

    platform.register(
        "resource_query",
        create_resource_query_workflow(),
        description="Query available resources"
    )

    # Add AI-enhanced endpoints
    @platform.endpoint("/ai/assign-task", methods=["POST"])
    async def ai_assign_task(request):
        """AI-assisted task assignment."""
        data = await request.json()
        result = await orchestrator.create_and_assign_task(
            task_id=data["task_id"],
            task_type=data["type"],
            priority=data["priority"],
            location=(data["lat"], data["lng"]),
            description=data["description"],
            commander_id=data["commander_id"],
            required_skills=data.get("required_skills", [])
        )
        return result

    @platform.endpoint("/ai/situation-summary", methods=["GET"])
    async def ai_situation_summary(request):
        """AI-generated situation summary."""
        time_window = request.query_params.get("minutes", 60)
        result = await orchestrator.get_situation_summary(int(time_window))
        return result

    @platform.endpoint("/health", methods=["GET"])
    async def health_check(request):
        """Health check endpoint."""
        return {"status": "healthy", "service": "ADRC Command Post"}

    return platform, db


def run_command_post(host: str = "127.0.0.1", port: int = 8080):
    """
    Run Command Post backend server.

    CRITICAL: Use start() NOT run()
    """
    platform, db = create_command_post_platform()

    logger.info(f"Starting ADRC Command Post on {host}:{port}")

    # CORRECT: Use start() not run()
    platform.start(host=host, port=port)


if __name__ == "__main__":
    run_command_post()
```

### Tasks

1. [ ] Create `platform.py` with corrected Nexus setup (4h)
   - Use `register()` not `register_workflow()`
   - Use `start()` not `run()`
   - Use `auto_discovery=False` with DataFlow
2. [ ] Add AI-enhanced endpoints (2h)
3. [ ] Add health check endpoint (30m)
4. [ ] Add graceful shutdown handling (1h)
5. [ ] Write platform tests (2h)
6. [ ] Document deployment (1h)

### Nexus Correction Checklist

- [ ] **register()** used instead of ~~register_workflow()~~
- [ ] **start()** used instead of ~~run()~~
- [ ] **auto_discovery=False** with DataFlow
- [ ] Local binding (127.0.0.1) for security
- [ ] Proper error handling

### Testing Requirements

- [ ] Platform starts without errors
- [ ] All endpoints respond correctly
- [ ] Health check works
- [ ] Workflows execute via API
- [ ] Graceful shutdown works

### Risk Assessment

- **LOW**: Optional component
- **NOTE**: System works without this via mesh-only

### Definition of Done

- [ ] Command Post backend deployable
- [ ] All Nexus corrections applied
- [ ] Tests pass

---

## TODO-051: End-to-End Workflow Integration

**Status**: PENDING
**Owner**: Full Team
**Estimated Effort**: 4 days
**Dependencies**: TODO-034 (Agent-Workflow), TODO-044 (Flutter Data Layer)

### Description

Integrate all components for end-to-end functionality: BLE mesh, Flutter app, DataFlow, and AI agents.

### Acceptance Criteria

- [ ] Responder joins mesh and appears in network
- [ ] Status updates propagate across mesh
- [ ] Task assignment flows end-to-end
- [ ] Incident reports broadcast to all nodes
- [ ] Data syncs correctly between devices
- [ ] AI routing works when available
- [ ] Offline fallback works correctly

### Integration Test Scenarios

#### Scenario 1: Responder Joins Network

```
1. Responder starts app
2. App starts BLE advertising
3. Nearby devices discover responder
4. Responder appears in device registry
5. Initial sync happens (if other devices have data)
6. Commander sees responder in dashboard
```

#### Scenario 2: Task Assignment with AI

```
1. Commander creates task in dashboard
2. AI agent recommends best responder
3. Task assignment workflow executes
4. Task assigned notification broadcast via mesh
5. Target responder receives notification
6. Task appears in responder's task list
7. Responder accepts/starts task
8. Status update propagates to commander
```

#### Scenario 3: Incident Report

```
1. Responder reports incident via app
2. Incident creation workflow executes
3. Incident broadcast via mesh
4. All nearby responders receive alert
5. Commander sees incident on map
6. Nearby responders can self-assign
```

#### Scenario 4: Network Partition

```
1. 10 devices in mesh
2. Partition: Group A (5 devices) | Group B (5 devices)
3. Both groups continue operating independently
4. New tasks/incidents created in each group
5. Partition heals (devices reconnect)
6. CRDT sync merges data correctly
7. No data loss, conflicts resolved
```

### Integration Points

```
┌──────────────────────────────────────────────────────────────────┐
│                    Integration Architecture                       │
│                                                                   │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐        │
│  │   Flutter   │     │   BLE       │     │   Other     │        │
│  │   App UI    │◄───►│   Mesh      │◄───►│   Devices   │        │
│  └──────┬──────┘     └──────┬──────┘     └─────────────┘        │
│         │                   │                                     │
│         ▼                   ▼                                     │
│  ┌─────────────┐     ┌─────────────┐                             │
│  │   Local     │◄───►│   Message   │                             │
│  │   SQLite    │     │   Handler   │                             │
│  └──────┬──────┘     └──────┬──────┘                             │
│         │                   │                                     │
│         └─────────┬─────────┘                                     │
│                   │                                               │
│                   ▼                                               │
│            ┌─────────────┐                                        │
│            │   Sync      │                                        │
│            │   Engine    │                                        │
│            └─────────────┘                                        │
└──────────────────────────────────────────────────────────────────┘
```

### Tasks

1. [ ] Create integration test framework (4h)
2. [ ] Implement Scenario 1: Responder Join (4h)
3. [ ] Implement Scenario 2: Task Assignment (4h)
4. [ ] Implement Scenario 3: Incident Report (3h)
5. [ ] Implement Scenario 4: Network Partition (4h)
6. [ ] End-to-end testing with multiple devices (8h)
7. [ ] Bug fixing and refinement (8h)

### Testing Requirements

- [ ] All scenarios pass on 5+ devices
- [ ] Data consistency verified after sync
- [ ] Offline operation confirmed
- [ ] Performance acceptable

### Risk Assessment

- **HIGH**: Integration is complex
- **Mitigation**: Clear interfaces, incremental testing

### Definition of Done

- [ ] All scenarios pass
- [ ] 5+ device testing complete
- [ ] No data loss in partition scenario

---

## TODO-052: Peer Trust Authentication

**Status**: PENDING
**Owner**: Security Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-012 (Mesh Protocol)

### Description

Implement basic peer trust authentication for the BLE mesh network. Devices vouch for newcomers based on pre-provisioned trust anchors.

### Acceptance Criteria

- [ ] Pre-provisioned trust anchors on devices
- [ ] New device verification via trusted peer
- [ ] Trust chain tracking
- [ ] Device revocation capability
- [ ] Cryptographic attestation

### Trust Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    Peer Trust Model                              │
│                                                                  │
│  1. Pre-Provisioning (Before Deployment)                        │
│     - Each device gets unique key pair                          │
│     - Root trust anchor distributed                             │
│     - Initial trusted devices list                              │
│                                                                  │
│  2. Runtime Trust (During Operation)                            │
│     - New device requests to join                               │
│     - Trusted peer verifies credentials                         │
│     - Peer vouches for new device                               │
│     - Trust propagates to network                               │
│                                                                  │
│  3. Revocation                                                  │
│     - Command posts can revoke devices                          │
│     - Revocation list propagates via mesh                       │
│     - Revoked devices rejected by all peers                     │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation

```dart
// lib/mesh/trust_manager.dart
import 'dart:typed_data';
import 'package:pointycastle/pointycastle.dart';

class TrustManager {
  final String deviceId;
  final AsymmetricKeyPair _keyPair;
  final Set<String> _trustedDevices = {};
  final Set<String> _revokedDevices = {};
  final Map<String, TrustChain> _trustChains = {};

  TrustManager({
    required this.deviceId,
    required AsymmetricKeyPair keyPair,
    required List<String> initialTrustedDevices,
  }) : _keyPair = keyPair {
    _trustedDevices.addAll(initialTrustedDevices);
  }

  /// Check if a device is trusted
  bool isTrusted(String deviceId) {
    if (_revokedDevices.contains(deviceId)) return false;
    return _trustedDevices.contains(deviceId);
  }

  /// Verify a device's claim with signature
  Future<bool> verifyDevice(String deviceId, Uint8List signature, Uint8List data) async {
    if (_revokedDevices.contains(deviceId)) return false;

    // Verify signature using device's public key
    final publicKey = await _getDevicePublicKey(deviceId);
    if (publicKey == null) return false;

    return _verifySignature(publicKey, signature, data);
  }

  /// Vouch for a new device
  Future<VouchMessage> vouchForDevice(String newDeviceId, Uint8List newDevicePublicKey) async {
    final vouch = VouchMessage(
      voucherId: deviceId,
      vouchedDeviceId: newDeviceId,
      vouchedPublicKey: newDevicePublicKey,
      timestamp: DateTime.now(),
    );

    // Sign the vouch
    vouch.signature = await _sign(vouch.toBytes());

    // Add to local trusted set
    _trustedDevices.add(newDeviceId);

    return vouch;
  }

  /// Process a vouch from another trusted device
  void processVouch(VouchMessage vouch) {
    // Verify voucher is trusted
    if (!isTrusted(vouch.voucherId)) return;

    // Verify signature
    if (!_verifyVouchSignature(vouch)) return;

    // Add new device to trusted set
    _trustedDevices.add(vouch.vouchedDeviceId);

    // Track trust chain
    _trustChains[vouch.vouchedDeviceId] = TrustChain(
      deviceId: vouch.vouchedDeviceId,
      vouchedBy: vouch.voucherId,
      timestamp: vouch.timestamp,
    );
  }

  /// Process revocation
  void processRevocation(RevocationMessage revocation) {
    // Verify revocation is from command post
    if (!_isCommandPost(revocation.issuerId)) return;

    // Add to revoked set
    _revokedDevices.add(revocation.deviceId);
    _trustedDevices.remove(revocation.deviceId);
  }
}

class VouchMessage {
  final String voucherId;
  final String vouchedDeviceId;
  final Uint8List vouchedPublicKey;
  final DateTime timestamp;
  Uint8List? signature;

  VouchMessage({
    required this.voucherId,
    required this.vouchedDeviceId,
    required this.vouchedPublicKey,
    required this.timestamp,
  });

  Uint8List toBytes() {
    // Serialize for signing
    return Uint8List.fromList([
      ...utf8.encode(voucherId),
      ...utf8.encode(vouchedDeviceId),
      ...vouchedPublicKey,
      ...utf8.encode(timestamp.toIso8601String()),
    ]);
  }
}
```

### Tasks

1. [ ] Design trust model and key management (4h)
2. [ ] Implement `TrustManager` class (6h)
3. [ ] Implement vouch protocol (4h)
4. [ ] Implement revocation protocol (3h)
5. [ ] Integrate with mesh protocol (3h)
6. [ ] Write security tests (4h)

### Testing Requirements

- [ ] Trusted devices can communicate
- [ ] Untrusted devices are rejected
- [ ] Vouch mechanism works
- [ ] Revocation propagates
- [ ] Cannot impersonate trusted device

### Risk Assessment

- **MEDIUM**: Security is critical but PoC scope allows basic implementation
- **NOTE**: Production will need hardening

### Definition of Done

- [ ] Basic trust authentication working
- [ ] Vouch and revocation implemented
- [ ] Security tests pass

---

## TODO-053: Two-Tier Data Classification

**Status**: PENDING
**Owner**: Security Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-020 (DataFlow Models)

### Description

Implement two-tier data classification system to protect sensitive information.

### Acceptance Criteria

- [ ] Tier 1 (Unclassified): Public safety info, resource locations
- [ ] Tier 2 (Sensitive): Responder locations, tactical movements, PII
- [ ] Classification enforcement on data access
- [ ] Classification labels on all data
- [ ] Audit logging for sensitive access

### Data Classification

```python
# src/velus/security/classification.py
from enum import Enum
from typing import Dict, Any
from functools import wraps

class DataTier(Enum):
    UNCLASSIFIED = 1  # Public safety info
    SENSITIVE = 2     # Responder locations, PII


# Classification rules by model and field
CLASSIFICATION_RULES = {
    "Responder": {
        "_default": DataTier.SENSITIVE,  # Default for model
        "id": DataTier.UNCLASSIFIED,
        "name": DataTier.SENSITIVE,      # PII
        "role": DataTier.UNCLASSIFIED,
        "team_id": DataTier.UNCLASSIFIED,
        "agency": DataTier.UNCLASSIFIED,
        "status": DataTier.UNCLASSIFIED,
        "location_lat": DataTier.SENSITIVE,  # Tactical
        "location_lng": DataTier.SENSITIVE,  # Tactical
        "skills": DataTier.UNCLASSIFIED,
        "battery_level": DataTier.UNCLASSIFIED,
    },
    "Task": {
        "_default": DataTier.UNCLASSIFIED,
        "assigned_to": DataTier.SENSITIVE,   # Links to responder
    },
    "Incident": {
        "_default": DataTier.UNCLASSIFIED,
        "reported_by": DataTier.SENSITIVE,   # Links to responder
    },
    "Resource": {
        "_default": DataTier.UNCLASSIFIED,
    }
}


def get_field_classification(model: str, field: str) -> DataTier:
    """Get classification tier for a specific field."""
    model_rules = CLASSIFICATION_RULES.get(model, {})
    return model_rules.get(field, model_rules.get("_default", DataTier.UNCLASSIFIED))


def filter_by_classification(
    data: Dict[str, Any],
    model: str,
    max_tier: DataTier
) -> Dict[str, Any]:
    """Filter data to only include fields up to max_tier classification."""
    filtered = {}
    for field, value in data.items():
        field_tier = get_field_classification(model, field)
        if field_tier.value <= max_tier.value:
            filtered[field] = value
        else:
            filtered[field] = "[REDACTED]"
    return filtered


def classify_access(model: str, fields: list = None):
    """Decorator to enforce classification on data access."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Get user's clearance level
            user_clearance = kwargs.get("user_clearance", DataTier.UNCLASSIFIED)

            # Execute function
            result = await func(*args, **kwargs)

            # Filter result based on clearance
            if isinstance(result, dict):
                return filter_by_classification(result, model, user_clearance)
            elif isinstance(result, list):
                return [filter_by_classification(item, model, user_clearance) for item in result]

            return result
        return wrapper
    return decorator


# Audit logging for sensitive access
class ClassificationAuditLog:
    """Log access to classified data."""

    @staticmethod
    async def log_access(
        user_id: str,
        model: str,
        record_id: str,
        fields_accessed: list,
        tier_accessed: DataTier
    ):
        if tier_accessed == DataTier.SENSITIVE:
            print(f"AUDIT: {user_id} accessed SENSITIVE {model}:{record_id} fields: {fields_accessed}")
            # In production: write to persistent audit log
```

### Tasks

1. [ ] Define classification rules (2h)
2. [ ] Implement classification utilities (3h)
3. [ ] Add filtering to data access (3h)
4. [ ] Implement audit logging (2h)
5. [ ] Write tests (2h)

### Testing Requirements

- [ ] Sensitive fields redacted for low-clearance users
- [ ] Full access for high-clearance users
- [ ] Audit log captures sensitive access
- [ ] Classification correctly assigned

### Definition of Done

- [ ] Two-tier classification working
- [ ] Audit logging functional
- [ ] Tests pass

---

## TODO-054: Role-Based Access Control

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-052 (Peer Trust)

### Description

Implement role-based access control for different user types (responder, commander, admin).

### Acceptance Criteria

- [ ] Role definitions (responder, commander, admin)
- [ ] Permission matrix
- [ ] Role enforcement on operations
- [ ] Role assignment during registration

### Role Matrix

| Operation | Responder | Commander | Admin |
|-----------|-----------|-----------|-------|
| View own status | Yes | Yes | Yes |
| Update own status | Yes | Yes | Yes |
| View all responders | Yes | Yes | Yes |
| View responder locations | No | Yes | Yes |
| Create task | No | Yes | Yes |
| Assign task | No | Yes | Yes |
| View all tasks | Own only | Yes | Yes |
| Report incident | Yes | Yes | Yes |
| Verify incident | No | Yes | Yes |
| Manage resources | No | Yes | Yes |
| Revoke devices | No | No | Yes |

### Implementation

```dart
// lib/core/rbac.dart
enum Role { responder, commander, admin }

enum Permission {
  viewOwnStatus,
  updateOwnStatus,
  viewAllResponders,
  viewResponderLocations,
  createTask,
  assignTask,
  viewAllTasks,
  reportIncident,
  verifyIncident,
  manageResources,
  revokeDevices,
}

class RBAC {
  static final Map<Role, Set<Permission>> _permissions = {
    Role.responder: {
      Permission.viewOwnStatus,
      Permission.updateOwnStatus,
      Permission.viewAllResponders,
      Permission.reportIncident,
    },
    Role.commander: {
      Permission.viewOwnStatus,
      Permission.updateOwnStatus,
      Permission.viewAllResponders,
      Permission.viewResponderLocations,
      Permission.createTask,
      Permission.assignTask,
      Permission.viewAllTasks,
      Permission.reportIncident,
      Permission.verifyIncident,
      Permission.manageResources,
    },
    Role.admin: Permission.values.toSet(), // All permissions
  };

  static bool hasPermission(Role role, Permission permission) {
    return _permissions[role]?.contains(permission) ?? false;
  }

  static void requirePermission(Role role, Permission permission) {
    if (!hasPermission(role, permission)) {
      throw UnauthorizedException(
        'Role $role does not have permission $permission',
      );
    }
  }
}
```

### Tasks

1. [ ] Define roles and permissions (2h)
2. [ ] Implement RBAC class (3h)
3. [ ] Add permission checks to operations (4h)
4. [ ] Add role assignment to registration (2h)
5. [ ] Write tests (3h)

### Testing Requirements

- [ ] Responders cannot assign tasks
- [ ] Commanders can see locations
- [ ] Admins have all permissions
- [ ] Permission denied errors work

### Definition of Done

- [ ] RBAC fully implemented
- [ ] All operations protected
- [ ] Tests pass

---

## TODO-055: Audit Logging

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-020 (DataFlow Models)

### Description

Implement comprehensive audit logging for all security-relevant operations.

### Acceptance Criteria

- [ ] Log all authentication events
- [ ] Log all data access (especially sensitive)
- [ ] Log all administrative actions
- [ ] Tamper-evident log storage
- [ ] Log rotation/archival

### Implementation

```python
# src/velus/security/audit.py
from dataclasses import dataclass
from datetime import datetime, UTC
from typing import Optional, Dict, Any
from dataflow import DataFlow
import hashlib
import json

db = DataFlow("sqlite:///audit.db")


@db.model
class AuditLog:
    id: str                    # UUID
    timestamp: str             # ISO timestamp
    event_type: str            # auth, access, admin, security
    actor_id: str              # Who performed action
    actor_role: str            # Role at time of action
    action: str                # What was done
    resource_type: Optional[str] = None  # Model accessed
    resource_id: Optional[str] = None    # Record accessed
    details: Optional[str] = None        # JSON details
    ip_address: Optional[str] = None     # If applicable
    device_id: Optional[str] = None      # Device identifier
    previous_hash: Optional[str] = None  # For tamper evidence
    hash: Optional[str] = None           # This record's hash


class AuditLogger:
    """Tamper-evident audit logging."""

    def __init__(self, db: DataFlow):
        self.db = db
        self._last_hash: Optional[str] = None

    async def log(
        self,
        event_type: str,
        actor_id: str,
        actor_role: str,
        action: str,
        resource_type: str = None,
        resource_id: str = None,
        details: Dict[str, Any] = None,
        device_id: str = None
    ):
        """Log an audit event with tamper-evident hash chain."""
        import uuid

        record = {
            "id": str(uuid.uuid4()),
            "timestamp": datetime.now(UTC).isoformat(),
            "event_type": event_type,
            "actor_id": actor_id,
            "actor_role": actor_role,
            "action": action,
            "resource_type": resource_type,
            "resource_id": resource_id,
            "details": json.dumps(details) if details else None,
            "device_id": device_id,
            "previous_hash": self._last_hash,
        }

        # Calculate hash for tamper evidence
        record["hash"] = self._calculate_hash(record)
        self._last_hash = record["hash"]

        # Store
        await self.db.express.create("AuditLog", record)

    def _calculate_hash(self, record: Dict) -> str:
        """Calculate SHA-256 hash of record for chain integrity."""
        data = json.dumps({k: v for k, v in record.items() if k != "hash"}, sort_keys=True)
        return hashlib.sha256(data.encode()).hexdigest()

    async def verify_chain(self) -> bool:
        """Verify audit log chain integrity."""
        logs = await self.db.express.list("AuditLog", order_by="timestamp")

        prev_hash = None
        for log in logs:
            # Verify previous hash link
            if log["previous_hash"] != prev_hash:
                return False

            # Verify record hash
            calculated = self._calculate_hash(log)
            if log["hash"] != calculated:
                return False

            prev_hash = log["hash"]

        return True


# Convenience functions
_logger: Optional[AuditLogger] = None


def get_audit_logger() -> AuditLogger:
    global _logger
    if _logger is None:
        _logger = AuditLogger(db)
    return _logger


async def audit_auth(actor_id: str, actor_role: str, action: str, **kwargs):
    await get_audit_logger().log("auth", actor_id, actor_role, action, **kwargs)


async def audit_access(actor_id: str, actor_role: str, action: str, resource_type: str, resource_id: str, **kwargs):
    await get_audit_logger().log("access", actor_id, actor_role, action, resource_type=resource_type, resource_id=resource_id, **kwargs)


async def audit_admin(actor_id: str, actor_role: str, action: str, **kwargs):
    await get_audit_logger().log("admin", actor_id, actor_role, action, **kwargs)
```

### Tasks

1. [ ] Create AuditLog model (1h)
2. [ ] Implement AuditLogger class (3h)
3. [ ] Add hash chain for tamper evidence (2h)
4. [ ] Add verification method (1h)
5. [ ] Integrate with operations (4h)
6. [ ] Write tests (3h)

### Testing Requirements

- [ ] All security events logged
- [ ] Hash chain is valid
- [ ] Tampering is detectable
- [ ] Logs queryable

### Definition of Done

- [ ] Audit logging complete
- [ ] Hash chain working
- [ ] Tests pass

---

## Phase 6 Dependencies Graph

```
TODO-021 (Workflows)
    │
    └──> TODO-050 (Command Post Backend)

TODO-034 (Agent-Workflow) + TODO-044 (Flutter Data)
    │
    └──> TODO-051 (E2E Integration)

TODO-012 (Mesh Protocol)
    │
    └──> TODO-052 (Peer Trust)
              │
              └──> TODO-054 (RBAC)

TODO-020 (DataFlow Models)
    │
    ├──> TODO-053 (Data Classification)
    │
    └──> TODO-055 (Audit Logging)
```

## Phase 6 Completion Criteria

- [ ] Command Post backend operational (if applicable)
- [ ] End-to-end integration verified
- [ ] Peer trust authentication working
- [ ] Data classification enforced
- [ ] RBAC protecting operations
- [ ] Audit logging capturing events
- [ ] **GO/NO-GO GATE**: E2E flows working

## References

- Architecture Plan: `/docs/02-plans/01-architecture-plan.md` (Section 7)
- Security Requirements: `/docs/01-analysis/00-requirements-summary.md`
