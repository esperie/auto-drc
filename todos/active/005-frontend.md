# TODO-040 to TODO-044: Frontend Phase (Ongoing, Focus Week 7-8)

**Phase**: 5 - Frontend
**Timeline**: Ongoing development, primary focus Week 7-8
**Status**: PENDING

**CRITICAL REQUIREMENTS FROM FRONTEND SPECIALIST**:
1. Create design system FIRST (colors, typography, components)
2. 56dp MINIMUM touch targets (gloved hands)
3. Dark mode as DEFAULT (outdoor visibility)
4. Multi-modal feedback (visual + haptic + audio)
5. Pure Flutter implementation (no Python on mobile)

---

## TODO-040: Core UI Components

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 4 days
**Dependencies**: TODO-004 (Design System Foundation)

### Description

Build the core reusable UI components following the established design system. All components must meet emergency response accessibility requirements.

### Acceptance Criteria

- [ ] Button component (primary, secondary, danger, ghost variants)
- [ ] Card component with proper elevation
- [ ] Input fields (text, dropdown, search)
- [ ] Status badges and chips
- [ ] List items with proper touch targets
- [ ] Loading states and skeletons
- [ ] Error states and empty states
- [ ] **All touch targets >= 56dp**
- [ ] **All components support dark mode**
- [ ] **Multi-modal feedback integrated**

### Component Specifications

#### 1. Emergency Button

```dart
// lib/presentation/widgets/emergency_button.dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import '../core/design_tokens.dart';
import '../core/feedback_service.dart';

enum ButtonVariant { primary, secondary, danger, ghost }

class EmergencyButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;
  final ButtonVariant variant;
  final IconData? icon;
  final bool isLoading;
  final bool isDisabled;

  const EmergencyButton({
    required this.label,
    required this.onPressed,
    this.variant = ButtonVariant.primary,
    this.icon,
    this.isLoading = false,
    this.isDisabled = false,
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      height: DesignTokens.minTouchTarget, // 56dp minimum!
      child: ElevatedButton(
        onPressed: isDisabled || isLoading
            ? null
            : () {
                FeedbackService.instance.buttonPress();
                onPressed();
              },
        style: _getButtonStyle(context),
        child: isLoading
            ? const SizedBox(
                width: 24,
                height: 24,
                child: CircularProgressIndicator(strokeWidth: 2),
              )
            : Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  if (icon != null) ...[
                    Icon(icon, size: 24),
                    const SizedBox(width: 8),
                  ],
                  Text(label, style: DesignTokens.buttonText),
                ],
              ),
      ),
    );
  }

  ButtonStyle _getButtonStyle(BuildContext context) {
    switch (variant) {
      case ButtonVariant.primary:
        return ElevatedButton.styleFrom(
          backgroundColor: DesignTokens.primaryBlue,
          foregroundColor: Colors.white,
          padding: const EdgeInsets.symmetric(horizontal: 24),
        );
      case ButtonVariant.danger:
        return ElevatedButton.styleFrom(
          backgroundColor: DesignTokens.emergencyRed,
          foregroundColor: Colors.white,
          padding: const EdgeInsets.symmetric(horizontal: 24),
        );
      case ButtonVariant.secondary:
        return ElevatedButton.styleFrom(
          backgroundColor: DesignTokens.surfaceColor,
          foregroundColor: DesignTokens.textPrimary,
          padding: const EdgeInsets.symmetric(horizontal: 24),
        );
      case ButtonVariant.ghost:
        return ElevatedButton.styleFrom(
          backgroundColor: Colors.transparent,
          foregroundColor: DesignTokens.textPrimary,
          elevation: 0,
          padding: const EdgeInsets.symmetric(horizontal: 24),
        );
    }
  }
}
```

#### 2. Status Card

```dart
// lib/presentation/widgets/status_card.dart
import 'package:flutter/material.dart';
import '../core/design_tokens.dart';

enum StatusLevel { critical, warning, normal, offline }

class StatusCard extends StatelessWidget {
  final String title;
  final String? subtitle;
  final StatusLevel status;
  final Widget? trailing;
  final VoidCallback? onTap;

  const StatusCard({
    required this.title,
    this.subtitle,
    this.status = StatusLevel.normal,
    this.trailing,
    this.onTap,
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      color: _getBackgroundColor(),
      elevation: 2,
      child: InkWell(
        onTap: onTap,
        borderRadius: BorderRadius.circular(12),
        child: Container(
          constraints: const BoxConstraints(
            minHeight: DesignTokens.minTouchTarget, // 56dp!
          ),
          padding: const EdgeInsets.all(16),
          child: Row(
            children: [
              _buildStatusIndicator(),
              const SizedBox(width: 12),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      title,
                      style: DesignTokens.bodyLarge.copyWith(
                        fontWeight: FontWeight.w600,
                      ),
                    ),
                    if (subtitle != null) ...[
                      const SizedBox(height: 4),
                      Text(
                        subtitle!,
                        style: DesignTokens.bodySmall.copyWith(
                          color: DesignTokens.textSecondary,
                        ),
                      ),
                    ],
                  ],
                ),
              ),
              if (trailing != null) trailing!,
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildStatusIndicator() {
    return Container(
      width: 12,
      height: 12,
      decoration: BoxDecoration(
        color: _getStatusColor(),
        shape: BoxShape.circle,
      ),
    );
  }

  Color _getStatusColor() {
    switch (status) {
      case StatusLevel.critical:
        return DesignTokens.emergencyRed;
      case StatusLevel.warning:
        return DesignTokens.warningAmber;
      case StatusLevel.normal:
        return DesignTokens.safetyGreen;
      case StatusLevel.offline:
        return DesignTokens.textSecondary;
    }
  }

  Color _getBackgroundColor() {
    if (status == StatusLevel.critical) {
      return DesignTokens.emergencyRed.withOpacity(0.1);
    }
    return DesignTokens.cardBackground;
  }
}
```

#### 3. Touch-Safe List Item

```dart
// lib/presentation/widgets/touch_list_item.dart
import 'package:flutter/material.dart';
import '../core/design_tokens.dart';
import '../core/feedback_service.dart';

class TouchListItem extends StatelessWidget {
  final Widget leading;
  final String title;
  final String? subtitle;
  final Widget? trailing;
  final VoidCallback? onTap;
  final VoidCallback? onLongPress;

  const TouchListItem({
    required this.leading,
    required this.title,
    this.subtitle,
    this.trailing,
    this.onTap,
    this.onLongPress,
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    return InkWell(
      onTap: onTap != null
          ? () {
              FeedbackService.instance.lightTap();
              onTap!();
            }
          : null,
      onLongPress: onLongPress != null
          ? () {
              FeedbackService.instance.heavyTap();
              onLongPress!();
            }
          : null,
      child: Container(
        constraints: const BoxConstraints(
          minHeight: DesignTokens.minTouchTarget, // 56dp minimum!
        ),
        padding: const EdgeInsets.symmetric(
          horizontal: 16,
          vertical: 12,
        ),
        child: Row(
          children: [
            SizedBox(
              width: 48,
              height: 48,
              child: Center(child: leading),
            ),
            const SizedBox(width: 16),
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(title, style: DesignTokens.bodyLarge),
                  if (subtitle != null)
                    Text(
                      subtitle!,
                      style: DesignTokens.bodySmall.copyWith(
                        color: DesignTokens.textSecondary,
                      ),
                    ),
                ],
              ),
            ),
            if (trailing != null) trailing!,
          ],
        ),
      ),
    );
  }
}
```

### Tasks

1. [ ] Create `EmergencyButton` component (2h)
2. [ ] Create `StatusCard` component (2h)
3. [ ] Create `TouchListItem` component (2h)
4. [ ] Create `PriorityBadge` component (1h)
5. [ ] Create `LoadingOverlay` component (1h)
6. [ ] Create `EmptyState` component (1h)
7. [ ] Create `ErrorState` component (1h)
8. [ ] Create `SearchField` component (2h)
9. [ ] Create `DropdownField` component (2h)
10. [ ] Build component showcase screen (4h)
11. [ ] Test all components at different sizes (2h)

### Touch Target Validation

```dart
// In debug mode, validate touch targets
void validateTouchTarget(BuildContext context, RenderBox renderBox) {
  final size = renderBox.size;
  if (size.height < 56 || size.width < 56) {
    debugPrint('WARNING: Touch target too small: ${size.height}x${size.width}');
  }
}
```

### Testing Requirements

- [ ] All buttons have minimum 56dp height
- [ ] All list items have minimum 56dp height
- [ ] Dark mode colors applied correctly
- [ ] Haptic feedback triggers on interactions
- [ ] Components render correctly at different screen sizes
- [ ] Accessibility labels present

### Risk Assessment

- **LOW**: Standard component development
- **HIGH IMPACT**: Skipping touch targets causes usability issues

### Definition of Done

- [ ] All core components implemented
- [ ] All touch targets >= 56dp verified
- [ ] Component showcase functional
- [ ] Dark mode verified

---

## TODO-041: Responder App Screens

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 4 days
**Dependencies**: TODO-040 (Core UI Components)

### Description

Build the main screens for the Responder mobile app interface.

### Acceptance Criteria

- [ ] Home/Dashboard screen
- [ ] Task list screen
- [ ] Task detail screen
- [ ] Status update screen
- [ ] Incident report screen
- [ ] Team view screen
- [ ] Settings screen
- [ ] Navigation between screens

### Screen Specifications

#### 1. Responder Dashboard

```dart
// lib/presentation/screens/responder/dashboard_screen.dart
class ResponderDashboardScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ADRC Responder'),
        actions: [
          // Battery/connectivity indicators
          _buildStatusIndicators(),
        ],
      ),
      body: SafeArea(
        child: Column(
          children: [
            // Current status card
            _buildMyStatusCard(),

            // Active task (if any)
            _buildActiveTaskCard(),

            // Quick actions (56dp buttons)
            _buildQuickActions(),

            // Recent alerts
            Expanded(child: _buildAlertsList()),
          ],
        ),
      ),
      bottomNavigationBar: _buildBottomNav(),
    );
  }

  Widget _buildQuickActions() {
    return Padding(
      padding: const EdgeInsets.all(16),
      child: Row(
        children: [
          Expanded(
            child: EmergencyButton(
              label: 'Report Incident',
              icon: Icons.warning,
              variant: ButtonVariant.danger,
              onPressed: () => _navigateToIncidentReport(),
            ),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: EmergencyButton(
              label: 'Update Status',
              icon: Icons.update,
              variant: ButtonVariant.primary,
              onPressed: () => _navigateToStatusUpdate(),
            ),
          ),
        ],
      ),
    );
  }
}
```

#### 2. Task Detail Screen

```dart
// lib/presentation/screens/responder/task_detail_screen.dart
class TaskDetailScreen extends StatelessWidget {
  final String taskId;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Task Details'),
      ),
      body: SafeArea(
        child: Column(
          children: [
            // Priority banner
            _buildPriorityBanner(),

            Expanded(
              child: SingleChildScrollView(
                padding: const EdgeInsets.all(16),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    // Task info
                    _buildTaskInfo(),

                    // Location with map
                    _buildLocationSection(),

                    // Description
                    _buildDescriptionSection(),

                    // Assigned by
                    _buildAssignmentInfo(),
                  ],
                ),
              ),
            ),

            // Action buttons (56dp+)
            _buildActionButtons(),
          ],
        ),
      ),
    );
  }

  Widget _buildActionButtons() {
    return SafeArea(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          children: [
            Expanded(
              child: EmergencyButton(
                label: 'Navigate',
                icon: Icons.navigation,
                variant: ButtonVariant.secondary,
                onPressed: _openNavigation,
              ),
            ),
            const SizedBox(width: 12),
            Expanded(
              child: EmergencyButton(
                label: task.status == 'assigned' ? 'Start Task' : 'Complete',
                icon: task.status == 'assigned' ? Icons.play_arrow : Icons.check,
                variant: ButtonVariant.primary,
                onPressed: _updateTaskStatus,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### 3. Incident Report Screen

```dart
// lib/presentation/screens/responder/incident_report_screen.dart
class IncidentReportScreen extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Report Incident'),
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              // Incident type selector (large touch targets)
              _buildTypeSelector(),

              const SizedBox(height: 16),

              // Severity selector
              _buildSeveritySelector(),

              const SizedBox(height: 16),

              // Location (auto-filled with current)
              _buildLocationField(),

              const SizedBox(height: 16),

              // Description
              _buildDescriptionField(),

              const SizedBox(height: 16),

              // Responders needed
              _buildRespondersNeeded(),

              const SizedBox(height: 24),

              // Submit button
              EmergencyButton(
                label: 'Submit Report',
                icon: Icons.send,
                variant: ButtonVariant.danger,
                onPressed: _submitReport,
              ),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildTypeSelector() {
    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('Incident Type', style: DesignTokens.labelLarge),
        const SizedBox(height: 8),
        Wrap(
          spacing: 8,
          runSpacing: 8,
          children: [
            for (final type in incidentTypes)
              _buildTypeChip(type),
          ],
        ),
      ],
    );
  }

  Widget _buildTypeChip(IncidentType type) {
    return ChoiceChip(
      label: Text(type.label),
      selected: selectedType == type,
      onSelected: (_) => setState(() => selectedType = type),
      // Ensure minimum touch target
      padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
      labelStyle: DesignTokens.bodyMedium,
    );
  }
}
```

### Tasks

1. [ ] Create `ResponderDashboardScreen` (4h)
2. [ ] Create `TaskListScreen` (3h)
3. [ ] Create `TaskDetailScreen` (3h)
4. [ ] Create `StatusUpdateScreen` (2h)
5. [ ] Create `IncidentReportScreen` (4h)
6. [ ] Create `TeamViewScreen` (2h)
7. [ ] Create `SettingsScreen` (2h)
8. [ ] Set up navigation with go_router (2h)
9. [ ] Connect screens to data layer (4h)
10. [ ] Test full user flows (4h)

### Testing Requirements

- [ ] All screens render correctly
- [ ] Navigation works between all screens
- [ ] Forms validate inputs
- [ ] Touch targets verified on all interactive elements
- [ ] Works in landscape and portrait
- [ ] Works on tablet sizes

### Definition of Done

- [ ] All responder screens implemented
- [ ] Full user flow testable
- [ ] All touch targets >= 56dp

---

## TODO-042: Commander Dashboard

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-040 (Core UI Components)

### Description

Build the Commander dashboard for operational oversight and task management.

### Acceptance Criteria

- [ ] Overview dashboard with key metrics
- [ ] Responder list with status
- [ ] Task management (create, assign, track)
- [ ] Incident overview
- [ ] Resource status
- [ ] Map-based view
- [ ] Situation summary display

### Screen Specifications

#### Commander Overview

```dart
// lib/presentation/screens/commander/overview_screen.dart
class CommanderOverviewScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Command Post'),
        actions: [
          IconButton(
            icon: const Icon(Icons.refresh),
            onPressed: _refreshData,
            tooltip: 'Refresh',
          ),
        ],
      ),
      body: SafeArea(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              // Key metrics row
              _buildMetricsRow(),

              const SizedBox(height: 16),

              // Situation summary (from AI agent)
              _buildSituationSummary(),

              const SizedBox(height: 16),

              // Critical alerts
              _buildCriticalAlerts(),

              const SizedBox(height: 16),

              // Active tasks overview
              _buildActiveTasksSection(),

              const SizedBox(height: 16),

              // Responder status grid
              _buildResponderStatusGrid(),
            ],
          ),
        ),
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: _createNewTask,
        icon: const Icon(Icons.add),
        label: const Text('New Task'),
        backgroundColor: DesignTokens.primaryBlue,
      ),
      bottomNavigationBar: _buildCommanderBottomNav(),
    );
  }

  Widget _buildMetricsRow() {
    return Row(
      children: [
        Expanded(child: _buildMetricCard('Active', '12', Icons.assignment)),
        const SizedBox(width: 12),
        Expanded(child: _buildMetricCard('Responders', '24', Icons.people)),
        const SizedBox(width: 12),
        Expanded(child: _buildMetricCard('Incidents', '5', Icons.warning)),
      ],
    );
  }

  Widget _buildMetricCard(String label, String value, IconData icon) {
    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            Icon(icon, size: 32, color: DesignTokens.primaryBlue),
            const SizedBox(height: 8),
            Text(value, style: DesignTokens.headlineLarge),
            Text(label, style: DesignTokens.bodySmall),
          ],
        ),
      ),
    );
  }
}
```

### Tasks

1. [ ] Create `CommanderOverviewScreen` (4h)
2. [ ] Create `ResponderManagementScreen` (3h)
3. [ ] Create `TaskCreateScreen` (3h)
4. [ ] Create `IncidentManagementScreen` (3h)
5. [ ] Create `ResourceManagementScreen` (2h)
6. [ ] Connect to situation summary agent (2h)
7. [ ] Test commander workflows (2h)

### Testing Requirements

- [ ] Dashboard displays correct metrics
- [ ] Task creation works
- [ ] Responder list updates
- [ ] Situation summary displays

### Definition of Done

- [ ] Commander dashboard fully functional
- [ ] Task management working
- [ ] Situation summary integration complete

---

## TODO-043: Map Integration

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-041 (Responder Screens)

### Description

Integrate offline-capable maps for situational awareness and navigation.

### Acceptance Criteria

- [ ] Map display with offline tiles
- [ ] Responder locations as markers
- [ ] Incident locations as markers
- [ ] Task locations as markers
- [ ] Current user location
- [ ] Marker clustering for dense areas
- [ ] Navigation launch to external app

### Implementation Options

1. **flutter_map + offline tiles** (Recommended)
   - OpenStreetMap tiles
   - Can pre-download regions
   - No API key needed

2. **mapbox_maps_flutter**
   - Better styling
   - Requires API key
   - Offline support available

### Map Component

```dart
// lib/presentation/widgets/situation_map.dart
import 'package:flutter/material.dart';
import 'package:flutter_map/flutter_map.dart';
import 'package:latlong2/latlong.dart';

class SituationMap extends StatelessWidget {
  final List<ResponderMarker> responders;
  final List<IncidentMarker> incidents;
  final List<TaskMarker> tasks;
  final LatLng? currentLocation;
  final void Function(String id)? onResponderTap;
  final void Function(String id)? onIncidentTap;

  @override
  Widget build(BuildContext context) {
    return FlutterMap(
      options: MapOptions(
        center: currentLocation ?? defaultCenter,
        zoom: 14.0,
        maxZoom: 18.0,
        minZoom: 10.0,
      ),
      children: [
        // Tile layer (offline-capable)
        TileLayer(
          urlTemplate: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
          // For offline: use cached tiles
          tileProvider: _getOfflineTileProvider(),
        ),

        // Incident markers (high priority)
        MarkerLayer(
          markers: incidents.map(_buildIncidentMarker).toList(),
        ),

        // Task markers
        MarkerLayer(
          markers: tasks.map(_buildTaskMarker).toList(),
        ),

        // Responder markers
        MarkerLayer(
          markers: responders.map(_buildResponderMarker).toList(),
        ),

        // Current location marker
        if (currentLocation != null)
          MarkerLayer(
            markers: [_buildCurrentLocationMarker(currentLocation!)],
          ),
      ],
    );
  }

  Marker _buildResponderMarker(ResponderMarker r) {
    return Marker(
      point: LatLng(r.lat, r.lng),
      width: 48, // Touch-friendly
      height: 48,
      builder: (ctx) => GestureDetector(
        onTap: () => onResponderTap?.call(r.id),
        child: Container(
          padding: const EdgeInsets.all(4),
          decoration: BoxDecoration(
            color: _getStatusColor(r.status),
            shape: BoxShape.circle,
            border: Border.all(color: Colors.white, width: 2),
          ),
          child: Icon(
            _getRoleIcon(r.role),
            color: Colors.white,
            size: 20,
          ),
        ),
      ),
    );
  }

  Marker _buildIncidentMarker(IncidentMarker i) {
    return Marker(
      point: LatLng(i.lat, i.lng),
      width: 56, // Larger for incidents
      height: 56,
      builder: (ctx) => GestureDetector(
        onTap: () => onIncidentTap?.call(i.id),
        child: Container(
          padding: const EdgeInsets.all(8),
          decoration: BoxDecoration(
            color: _getSeverityColor(i.severity),
            shape: BoxShape.circle,
            border: Border.all(color: Colors.white, width: 3),
            boxShadow: [
              BoxShadow(
                color: _getSeverityColor(i.severity).withOpacity(0.5),
                blurRadius: 8,
                spreadRadius: 2,
              ),
            ],
          ),
          child: const Icon(
            Icons.warning,
            color: Colors.white,
            size: 24,
          ),
        ),
      ),
    );
  }
}
```

### Offline Tile Caching

```dart
// lib/data/services/map_cache_service.dart
class MapCacheService {
  /// Pre-download map tiles for a region
  Future<void> cacheRegion({
    required LatLngBounds bounds,
    required int minZoom,
    required int maxZoom,
    void Function(double progress)? onProgress,
  }) async {
    // Calculate tiles needed
    final tiles = _calculateTilesInBounds(bounds, minZoom, maxZoom);

    int downloaded = 0;
    for (final tile in tiles) {
      await _downloadAndCacheTile(tile);
      downloaded++;
      onProgress?.call(downloaded / tiles.length);
    }
  }

  /// Check if region is cached
  Future<bool> isRegionCached(LatLngBounds bounds, int zoom) async {
    // Check local cache
    return _checkTilesExist(bounds, zoom);
  }
}
```

### Tasks

1. [ ] Add flutter_map dependency (30m)
2. [ ] Create `SituationMap` component (4h)
3. [ ] Implement responder markers (2h)
4. [ ] Implement incident markers (2h)
5. [ ] Implement task markers (1h)
6. [ ] Add current location tracking (2h)
7. [ ] Implement offline tile caching (4h)
8. [ ] Add navigation launch (1h)
9. [ ] Test map performance (2h)

### Testing Requirements

- [ ] Map renders correctly
- [ ] Markers display at correct positions
- [ ] Tap handlers work on markers
- [ ] Offline tiles load when cached
- [ ] Performance acceptable with 50+ markers

### Definition of Done

- [ ] Map fully functional
- [ ] Offline tiles working
- [ ] All marker types implemented

---

## TODO-044: Offline-First Data Layer (Flutter)

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-003 (Flutter Setup)

### Description

Implement Flutter-side data layer with SQLite for offline-first operation and sync with mesh network.

### Acceptance Criteria

- [ ] Local SQLite database setup
- [ ] Repository pattern for data access
- [ ] Data models matching Python backend
- [ ] Sync queue for outgoing changes
- [ ] Merge logic for incoming changes
- [ ] Conflict resolution (LWW)

### Data Layer Architecture

```dart
// lib/data/repositories/responder_repository.dart
abstract class ResponderRepository {
  Future<List<Responder>> getAll();
  Future<Responder?> getById(String id);
  Future<void> save(Responder responder);
  Future<void> delete(String id);
  Stream<List<Responder>> watchAll();
  Future<void> syncFromMesh(List<Map<String, dynamic>> updates);
}

class LocalResponderRepository implements ResponderRepository {
  final Database _db;
  final SyncQueue _syncQueue;

  @override
  Future<void> save(Responder responder) async {
    // Save locally
    await _db.insert('responders', responder.toMap(),
        conflictAlgorithm: ConflictAlgorithm.replace);

    // Queue for mesh broadcast
    await _syncQueue.enqueue(SyncUpdate(
      type: 'responder',
      operation: 'update',
      data: responder.toMap(),
      timestamp: DateTime.now(),
    ));
  }

  @override
  Future<void> syncFromMesh(List<Map<String, dynamic>> updates) async {
    for (final update in updates) {
      // LWW conflict resolution
      final existing = await getById(update['id']);
      if (existing == null ||
          update['timestamp'] > existing.lastUpdated.toIso8601String()) {
        await _db.insert('responders', update,
            conflictAlgorithm: ConflictAlgorithm.replace);
      }
    }
  }
}
```

### Tasks

1. [ ] Set up sqflite database (2h)
2. [ ] Create data models (Task, Responder, Incident, Resource) (4h)
3. [ ] Create repository interfaces (2h)
4. [ ] Implement local repositories (6h)
5. [ ] Create sync queue (3h)
6. [ ] Implement LWW merge logic (2h)
7. [ ] Write data layer tests (3h)

### Testing Requirements

- [ ] CRUD operations work offline
- [ ] Sync queue persists across app restarts
- [ ] LWW correctly resolves conflicts
- [ ] Watch streams update on changes

### Definition of Done

- [ ] Data layer fully functional
- [ ] Offline operation verified
- [ ] Sync queue working

---

## Phase 5 Dependencies Graph

```
TODO-004 (Design System)
    │
    └──> TODO-040 (Core Components)
              │
              ├──> TODO-041 (Responder Screens)
              │         │
              │         └──> TODO-043 (Map Integration)
              │
              └──> TODO-042 (Commander Dashboard)

TODO-003 (Flutter Setup)
    │
    └──> TODO-044 (Data Layer)
```

## Phase 5 Completion Criteria

- [ ] Design system established and documented
- [ ] All core UI components built
- [ ] Responder app screens functional
- [ ] Commander dashboard functional
- [ ] Map integration working
- [ ] Offline data layer operational

## References

- Architecture Plan: `/docs/02-plans/01-architecture-plan.md` (Section 2.3)
- Frontend Requirements: This document
