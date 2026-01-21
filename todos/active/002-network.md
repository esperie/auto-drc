# TODO-010 to TODO-014: BLE Mesh Network Phase (Week 3-4)

**Phase**: 2 - BLE Mesh Network
**Timeline**: Week 3-4
**Status**: PENDING

**RISK GATE**: End of Week 4 - Must have 5+ nodes communicating via mesh
**FALLBACK**: If BLE mesh fails, evaluate WiFi Direct as alternative

---

## TODO-010: BLE Library Integration

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-003 (Flutter Setup)

### Description

Integrate flutter_blue_plus library and establish basic BLE scanning, advertising, and connection capabilities.

### Acceptance Criteria

- [ ] flutter_blue_plus added and configured
- [ ] BLE state monitoring (on/off, permissions)
- [ ] Device scanning with filters
- [ ] GATT service/characteristic discovery
- [ ] Basic read/write to characteristics
- [ ] Connection state management
- [ ] Error handling for common BLE issues

### Tasks

1. [ ] Add flutter_blue_plus to pubspec.yaml (30m)
2. [ ] Create `ble_service.dart` wrapper class (4h)
   ```dart
   class BleService {
     Stream<BleState> get stateStream;
     Future<void> startScan({Duration timeout});
     Future<void> stopScan();
     Stream<ScanResult> get scanResults;
     Future<void> connect(BluetoothDevice device);
     Future<void> disconnect(BluetoothDevice device);
     Future<List<BluetoothService>> discoverServices(BluetoothDevice device);
     Future<void> writeCharacteristic(characteristic, data);
     Stream<List<int>> readCharacteristic(characteristic);
   }
   ```
3. [ ] Implement permission request flow (2h)
4. [ ] Create BLE state provider/notifier (2h)
5. [ ] Build debug screen for BLE testing (3h)
6. [ ] Test on physical Android device (2h)

### Technical Notes

**GATT Service Design** (for custom mesh protocol):
```
Service UUID: Custom (e.g., 12345678-1234-5678-1234-56789abcdef0)
├── TX Characteristic (Write): Send messages to peer
├── RX Characteristic (Notify): Receive messages from peer
└── Info Characteristic (Read): Device ID, mesh state
```

### Testing Requirements

- [ ] Can scan and find nearby BLE devices
- [ ] Can connect to another device running the app
- [ ] Can send/receive data via characteristics
- [ ] Handles BLE off/on gracefully
- [ ] Handles permission denial gracefully

### Risk Assessment

- **MEDIUM**: BLE behavior varies across Android versions
- **Mitigation**: Test on multiple Android versions (10, 11, 12, 13, 14)

### Definition of Done

- [ ] All acceptance criteria met
- [ ] BLE test screen shows scanning/connection working
- [ ] Works on at least 2 different physical Android devices

---

## TODO-011: Device Discovery Service

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-010 (BLE Library Integration)

### Description

Implement mesh-aware device discovery that maintains a registry of nearby ADRC devices and their capabilities.

### Acceptance Criteria

- [ ] Continuous background scanning for ADRC devices
- [ ] Device registry with:
  - Device ID (UUID)
  - Signal strength (RSSI)
  - Last seen timestamp
  - Connection state
  - Device role (responder, commander)
- [ ] Automatic reconnection to known devices
- [ ] Device timeout/removal after period of non-visibility
- [ ] Advertisement of self as ADRC device

### Tasks

1. [ ] Define ADRC service UUID and advertisement format (2h)
   ```dart
   class AdrcAdvertisement {
     static const serviceUuid = '12345678-...';
     String deviceId;
     String role;  // responder, commander
     String teamId;
     int meshHopCount;  // For routing
   }
   ```

2. [ ] Create `DeviceRegistry` class (4h)
   ```dart
   class DeviceRegistry {
     List<DiscoveredDevice> get nearbyDevices;
     Stream<DeviceEvent> get deviceEvents; // join, leave, update
     void updateDevice(String id, ScanResult result);
     void removeStaleDevices(Duration timeout);
   }
   ```

3. [ ] Implement continuous scanning with battery optimization (3h)
   - Scan intervals (active: 2s scan / 5s pause)
   - Background mode considerations

4. [ ] Implement self-advertisement (2h)
   - Advertise when app in foreground
   - Consider background advertising limits

5. [ ] Create device list UI component (2h)

### Testing Requirements

- [ ] Two devices discover each other within 10 seconds
- [ ] Device list updates when devices come/go
- [ ] RSSI values update in real-time
- [ ] Stale devices removed after timeout
- [ ] Battery impact < 5% per hour during active scanning

### Risk Assessment

- **MEDIUM**: Android background BLE restrictions
- **Mitigation**: Foreground service for continuous operation

### Definition of Done

- [ ] All acceptance criteria met
- [ ] Devices reliably discover each other
- [ ] Battery impact acceptable

---

## TODO-012: Custom Mesh Protocol

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 4 days
**Dependencies**: TODO-011 (Device Discovery)

### Description

Implement custom mesh networking protocol for message routing between ADRC devices. Uses flooding with TTL and message deduplication.

### Acceptance Criteria

- [ ] Message format defined (header + payload)
- [ ] Flooding protocol with configurable TTL
- [ ] Message deduplication (prevent loops)
- [ ] Acknowledgment mechanism for reliable delivery
- [ ] Multi-hop message relay
- [ ] Network topology awareness (neighbor list)
- [ ] Message prioritization (critical > high > normal)

### Message Format

```dart
class MeshMessage {
  String messageId;       // UUID for deduplication
  String sourceId;        // Originating device
  String? destinationId;  // null = broadcast
  int ttl;                // Time-to-live (hops remaining)
  int hopCount;           // Hops traveled
  MessagePriority priority;
  MessageType type;       // status, task, incident, sync, ack
  DateTime timestamp;
  Map<String, dynamic> payload;

  // Routing info
  List<String> path;      // Device IDs traversed
}

enum MessagePriority { critical, high, normal, low }
enum MessageType { status, task, incident, resource, sync, ack, ping }
```

### Tasks

1. [ ] Define message format and serialization (4h)
   - Use Protocol Buffers or MessagePack for efficiency
   - JSON fallback for debugging

2. [ ] Implement message routing logic (8h)
   ```dart
   class MeshRouter {
     Future<void> sendMessage(MeshMessage message);
     Future<void> broadcast(MeshMessage message);
     void handleIncomingMessage(MeshMessage message);
     bool shouldRelay(MeshMessage message);
     void updateRoutingTable(MeshMessage message);
   }
   ```

3. [ ] Implement deduplication cache (2h)
   - Store message IDs for 5 minutes
   - LRU eviction when cache full

4. [ ] Implement acknowledgment system (4h)
   - ACK messages for unicast
   - Retry logic with exponential backoff
   - Delivery confirmation callback

5. [ ] Implement priority queuing (2h)
   - Critical messages sent first
   - Queue management for congestion

6. [ ] Build mesh debug/visualization screen (4h)
   - Show network topology
   - Message flow visualization
   - Statistics (sent, received, relayed, dropped)

### Protocol Rules

1. **TTL Management**:
   - Default TTL = 5 hops
   - Critical messages = 10 hops
   - Decrement TTL on relay, drop when TTL = 0

2. **Deduplication**:
   - Cache message IDs for 5 minutes
   - Drop duplicate messages immediately

3. **Relay Decision**:
   ```dart
   bool shouldRelay(MeshMessage msg) {
     if (msg.ttl <= 0) return false;
     if (seenMessages.contains(msg.messageId)) return false;
     if (msg.destinationId == myDeviceId) return false;
     return true;
   }
   ```

4. **Acknowledgment**:
   - Unicast messages require ACK within 5 seconds
   - Retry up to 3 times with exponential backoff
   - Report delivery failure after max retries

### Testing Requirements

- [ ] Message reaches device 3 hops away
- [ ] Duplicate messages are not relayed
- [ ] Messages with TTL=0 are not relayed
- [ ] ACK received for delivered messages
- [ ] Delivery failure reported for unreachable devices
- [ ] Priority messages processed before normal

### Risk Assessment

- **HIGH**: This is the core of mesh networking
- **Mitigation**: Start simple (flooding), optimize later if needed
- **Fallback**: If mesh doesn't work, consider hub-and-spoke model

### Definition of Done

- [ ] All acceptance criteria met
- [ ] 5+ devices form mesh and exchange messages
- [ ] Messages relay through intermediate nodes
- [ ] Reliable delivery for unicast messages

---

## TODO-013: Message Serialization

**Status**: PENDING
**Owner**: Backend Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-002 (Architecture ADRs)

### Description

Define and implement message serialization format that works across Python backend and Flutter mobile. Ensure efficient encoding for BLE transmission.

### Acceptance Criteria

- [ ] Message schema defined (shared between Python and Flutter)
- [ ] Serialization library selected (Protocol Buffers recommended)
- [ ] Python serialization/deserialization
- [ ] Dart serialization/deserialization
- [ ] Message validation
- [ ] Size optimization for BLE (< 512 bytes per message)

### Message Types

```protobuf
// messages.proto
syntax = "proto3";

message MeshMessage {
  string message_id = 1;
  string source_id = 2;
  string destination_id = 3;  // empty = broadcast
  uint32 ttl = 4;
  uint32 hop_count = 5;
  Priority priority = 6;
  MessageType type = 7;
  int64 timestamp = 8;
  bytes payload = 9;
  repeated string path = 10;
}

enum Priority {
  PRIORITY_LOW = 0;
  PRIORITY_NORMAL = 1;
  PRIORITY_HIGH = 2;
  PRIORITY_CRITICAL = 3;
}

enum MessageType {
  TYPE_STATUS = 0;
  TYPE_TASK = 1;
  TYPE_INCIDENT = 2;
  TYPE_RESOURCE = 3;
  TYPE_SYNC = 4;
  TYPE_ACK = 5;
  TYPE_PING = 6;
}

// Payload messages
message StatusUpdate {
  string responder_id = 1;
  string status = 2;
  double latitude = 3;
  double longitude = 4;
  int32 battery_level = 5;
}

message TaskAssignment {
  string task_id = 1;
  string type = 2;
  string priority = 3;
  string description = 4;
  double latitude = 5;
  double longitude = 6;
  string assigned_to = 7;
  string assigned_by = 8;
}

// ... etc for other payload types
```

### Tasks

1. [ ] Define protobuf schema (4h)
2. [ ] Generate Python code from protobuf (1h)
3. [ ] Generate Dart code from protobuf (1h)
4. [ ] Create Python message helpers (2h)
5. [ ] Create Dart message helpers (2h)
6. [ ] Add message validation (2h)
7. [ ] Test cross-platform serialization (2h)

### Size Constraints

BLE characteristic max size: ~512 bytes (negotiated MTU)
- Header: ~50 bytes
- Payload: ~450 bytes max
- If larger: implement message fragmentation

### Testing Requirements

- [ ] Python can serialize/deserialize all message types
- [ ] Dart can serialize/deserialize all message types
- [ ] Message serialized in Python deserializes correctly in Dart
- [ ] Message serialized in Dart deserializes correctly in Python
- [ ] All messages < 512 bytes

### Risk Assessment

- **LOW**: Standard serialization task
- **MEDIUM**: Cross-platform compatibility edge cases

### Definition of Done

- [ ] All acceptance criteria met
- [ ] Shared schema documented
- [ ] Cross-platform tests pass

---

## TODO-014: BLE Range Testing

**Status**: PENDING
**Owner**: QA
**Estimated Effort**: 2 days
**Dependencies**: TODO-012 (Custom Mesh Protocol)

### Description

Conduct comprehensive testing of BLE mesh performance including range, reliability, and multi-hop behavior.

### Acceptance Criteria

- [ ] Range measurements documented (indoor/outdoor)
- [ ] Reliability metrics collected (delivery rate at various distances)
- [ ] Multi-hop performance validated
- [ ] Battery impact measured
- [ ] Interference testing (crowded RF environment)
- [ ] Recommendations documented for deployment

### Test Scenarios

1. **Direct Range Test**:
   - Measure max reliable range (line of sight)
   - Measure max reliable range (with obstacles)
   - Document at 10m, 20m, 30m, 50m, 100m

2. **Multi-Hop Test**:
   - 3-hop message delivery
   - 5-hop message delivery
   - Latency measurement per hop

3. **Reliability Test**:
   - Send 1000 messages, measure delivery rate
   - Test under varying distances
   - Test with interference (other BLE devices, WiFi)

4. **Battery Test**:
   - Measure battery drain over 2 hours
   - Active scanning + mesh operation
   - Document % drain per hour

5. **Capacity Test**:
   - 5 devices in mesh
   - 10 devices in mesh
   - 20 devices in mesh (if available)

### Tasks

1. [ ] Create test plan document (2h)
2. [ ] Prepare test devices (5-10 Android phones) (1h)
3. [ ] Execute direct range tests (4h)
4. [ ] Execute multi-hop tests (4h)
5. [ ] Execute reliability tests (4h)
6. [ ] Execute battery tests (3h)
7. [ ] Compile results and recommendations (4h)

### Metrics to Collect

| Metric | Target | Minimum |
|--------|--------|---------|
| Direct range (LoS) | 50m | 30m |
| Direct range (obstacles) | 20m | 10m |
| 3-hop delivery rate | 95% | 90% |
| 5-hop delivery rate | 90% | 85% |
| Latency per hop | < 100ms | < 200ms |
| Battery drain/hour | < 5% | < 10% |

### Testing Requirements

- [ ] All test scenarios executed
- [ ] Results documented with evidence
- [ ] Pass/Fail determined against criteria
- [ ] Recommendations for production deployment

### Risk Assessment

- **CRITICAL**: This is the Go/No-Go gate for BLE viability
- **Fallback**: If BLE doesn't meet criteria, evaluate WiFi Direct

### Definition of Done

- [ ] All test scenarios completed
- [ ] Results documented
- [ ] Go/No-Go recommendation made
- [ ] Stakeholders informed of results

---

## Phase 2 Dependencies Graph

```
TODO-010 (BLE Library)
    │
    └──> TODO-011 (Device Discovery)
              │
              └──> TODO-012 (Mesh Protocol) <── TODO-013 (Serialization)
                        │
                        └──> TODO-014 (Range Testing)
```

## Phase 2 Completion Criteria

- [ ] BLE mesh operational with 5+ devices
- [ ] Messages relay through intermediate nodes
- [ ] Range and reliability meet minimum criteria
- [ ] Message serialization works cross-platform
- [ ] **GO/NO-GO DECISION**: BLE viability confirmed

## Risk Gate (Week 4)

**Criteria**: 5+ nodes must successfully form mesh and exchange messages

**If PASS**: Continue to Phase 3 (Data Layer)

**If FAIL**:
1. Analyze root cause
2. Evaluate WiFi Direct as alternative
3. Consider hybrid approach
4. Escalate to stakeholders

## References

- Architecture Plan: `/docs/02-plans/01-architecture-plan.md` (Section 1.3)
- Roadmap Risk Management: `/docs/02-plans/00-poc-roadmap.md` (Section 7)
