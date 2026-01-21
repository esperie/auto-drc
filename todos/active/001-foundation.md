# TODO-001 to TODO-004: Foundation Phase (Week 1-2)

**Phase**: 1 - Foundation
**Timeline**: Week 1-2
**Status**: PENDING

---

## TODO-001: Project Setup & Environment

**Status**: PENDING
**Owner**: Tech Lead
**Estimated Effort**: 2 days
**Dependencies**: None

### Description

Set up the development environment for ADRC PoC including Python/Kailash SDK configuration, repository structure, and tooling.

### Acceptance Criteria

- [ ] Python 3.11+ environment configured with uv/pyenv
- [ ] Kailash SDK installed (`pip install kailash kailash-dataflow kailash-kaizen`)
- [ ] Project structure created matching architecture plan:
  ```
  auto-drc/
  ├── src/velus/           # Backend (Python/Kailash)
  │   ├── models/          # DataFlow models
  │   ├── workflows/       # Kailash workflows
  │   ├── agents/          # Kaizen AI agents
  │   ├── api/             # Nexus API (optional)
  │   └── sync/            # CRDT sync engine
  ├── apps/mobile/         # Flutter app
  ├── tests/               # Python tests
  └── docs/
  ```
- [ ] Git repository initialized with proper .gitignore
- [ ] Pre-commit hooks configured (ruff, black, mypy)
- [ ] `.env.example` created with required environment variables
- [ ] CI/CD pipeline skeleton (GitHub Actions)
- [ ] Development README with setup instructions

### Tasks

1. [ ] Create virtual environment and install dependencies (1h)
2. [ ] Create project directory structure (1h)
3. [ ] Configure linting and formatting (2h)
4. [ ] Set up pre-commit hooks (1h)
5. [ ] Create .env.example with LLM API keys placeholder (30m)
6. [ ] Set up basic GitHub Actions workflow (2h)
7. [ ] Write development setup README (1h)

### Testing Requirements

- [ ] `pip install -e .` works without errors
- [ ] `pytest` runs (even if no tests yet)
- [ ] Pre-commit hooks pass on sample code
- [ ] CI pipeline runs on push

### Risk Assessment

- **LOW**: Standard setup, well-documented processes

### Definition of Done

- [ ] All acceptance criteria met
- [ ] Team can clone and start developing within 15 minutes
- [ ] CI passes on main branch

---

## TODO-002: Architecture Decisions & ADRs

**Status**: PENDING
**Owner**: Tech Lead
**Estimated Effort**: 3 days
**Dependencies**: TODO-001

### Description

Finalize architecture decisions and document them as Architecture Decision Records (ADRs). This includes framework selections, protocol choices, and integration patterns.

### Acceptance Criteria

- [ ] ADR-001: Mobile Platform (Flutter vs Native)
  - Decision: Flutter
  - Rationale documented
- [ ] ADR-002: BLE Mesh Protocol
  - Decision: Custom protocol on flutter_blue_plus
  - Message format specified (JSON/Protobuf)
- [ ] ADR-003: Database Strategy
  - Decision: SQLite via DataFlow
  - Schema migration approach
- [ ] ADR-004: State Synchronization
  - Decision: CRDT library selection
  - Conflict resolution rules
- [ ] ADR-005: AI Agent Architecture
  - Decision: Kaizen with offline-first BaseEdgeAgent pattern
  - **CRITICAL**: Rule-based fallbacks required for offline operation
- [ ] ADR-006: API Strategy
  - **CRITICAL**: Nexus only for optional Command Post backend
  - Mobile devices use pure Flutter (no Python runtime)

### Tasks

1. [ ] Draft ADR-001: Mobile Platform selection (2h)
2. [ ] Draft ADR-002: BLE Mesh Protocol design (4h)
3. [ ] Draft ADR-003: Database Strategy (2h)
4. [ ] Draft ADR-004: State Sync approach (4h)
5. [ ] Draft ADR-005: AI Agent Architecture (4h)
   - **Include**: BaseEdgeAgent pattern for offline-first
   - **Include**: Agent calls outside workflows (PythonCode cannot await)
6. [ ] Draft ADR-006: API Strategy with Nexus corrections (2h)
   - **Include**: `register()` not `register_workflow()`
   - **Include**: `start()` not `run()`
7. [ ] Review session with team (2h)
8. [ ] Finalize and commit ADRs (2h)

### ADR Template

```markdown
# ADR-XXX: [Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
[Why are we making this decision?]

## Decision
[What is the decision?]

## Consequences
[What are the positive/negative implications?]

## Alternatives Considered
[What other options were evaluated?]
```

### Specialist Corrections to Document

**Kaizen (AI Agents)**:
- BaseAgent requires `config=` and `signature=` parameters
- Agents must be called OUTSIDE PythonCode nodes (cannot await)
- Use simple coordinator pattern, not full multi-agent

**Nexus (API)**:
- Use `register()` not `register_workflow()`
- Use `start()` not `run()`
- Python cannot run on mobile devices
- Command Post backend is optional

**DataFlow (Database)**:
- Template syntax: `${}` not `{{}}`
- Never manually set `created_at`/`updated_at`
- Use `add_connection()` not `connect()`

### Testing Requirements

- [ ] ADRs reviewed by at least 2 team members
- [ ] Technical decisions validated against requirements
- [ ] No conflicts between ADRs

### Risk Assessment

- **MEDIUM**: Wrong architecture decisions could require rework
- **Mitigation**: Early validation, clear Go/No-Go gates

### Definition of Done

- [ ] All 6 ADRs documented and approved
- [ ] Team consensus on architecture
- [ ] ADRs committed to `/docs/adrs/` directory

---

## TODO-003: Flutter Mobile Project Setup

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-001

### Description

Initialize the Flutter mobile application with proper architecture, dependencies, and project structure.

### Acceptance Criteria

- [ ] Flutter project created at `apps/mobile/`
- [ ] Project structure follows clean architecture:
  ```
  apps/mobile/lib/
  ├── core/              # Utilities, constants, theme
  ├── data/              # Data sources, repositories impl
  ├── domain/            # Entities, use cases, repository interfaces
  ├── presentation/      # UI (screens, widgets, state)
  ├── mesh/              # BLE mesh implementation
  └── sync/              # Data sync integration
  ```
- [ ] Key dependencies added to `pubspec.yaml`:
  - `flutter_blue_plus` - BLE communication
  - `sqflite` - Local SQLite database
  - `provider` or `riverpod` - State management
  - `go_router` - Navigation
  - `flutter_map` or offline map solution
- [ ] Android permissions configured in `AndroidManifest.xml`
  - Bluetooth, Bluetooth Admin, Location
- [ ] Build configurations for debug/release
- [ ] App icon and splash screen placeholders

### Tasks

1. [ ] Create Flutter project with `flutter create` (30m)
2. [ ] Set up directory structure (1h)
3. [ ] Add dependencies to pubspec.yaml (1h)
4. [ ] Configure Android permissions (1h)
5. [ ] Set up basic app shell with navigation (2h)
6. [ ] Create theme configuration placeholder (1h)
7. [ ] Configure build flavors (debug/release) (1h)
8. [ ] Write mobile README with build instructions (1h)

### Android Permissions Required

```xml
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

### Testing Requirements

- [ ] `flutter build apk` succeeds
- [ ] App launches on Android emulator
- [ ] App launches on physical Android device
- [ ] BLE permissions requestable (may not grant yet)

### Risk Assessment

- **LOW**: Standard Flutter setup
- **MEDIUM**: BLE permission edge cases on different Android versions

### Definition of Done

- [ ] All acceptance criteria met
- [ ] App builds and runs
- [ ] README documents build process

---

## TODO-004: Design System Foundation

**Status**: PENDING
**Owner**: Mobile Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-003

### Description

Create comprehensive design system BEFORE building features. This prevents UI inconsistencies and enables rapid, consistent development.

### Acceptance Criteria

**Color System**:
- [ ] Primary palette (emergency red, safety green, warning amber)
- [ ] Secondary/neutral palette
- [ ] Semantic colors (success, error, warning, info)
- [ ] **Dark mode as DEFAULT** (outdoor visibility)
- [ ] High contrast variants for bright sunlight

**Typography**:
- [ ] Font family selection (highly readable, supports multiple weights)
- [ ] Type scale (heading, body, caption sizes)
- [ ] Line heights and letter spacing
- [ ] Minimum readable size: 16sp for body text

**Spacing & Layout**:
- [ ] Spacing scale (4, 8, 12, 16, 24, 32, 48px)
- [ ] Grid system for layouts
- [ ] Safe areas and margins
- [ ] **56dp MINIMUM touch targets** (gloved hands requirement)

**Component Tokens**:
- [ ] Button styles (primary, secondary, danger, ghost)
- [ ] Card styles with proper elevation
- [ ] Input field styles
- [ ] Icon sizes and touch areas

**Feedback System**:
- [ ] **Visual feedback** - Color/animation on interactions
- [ ] **Haptic feedback** - Vibration patterns for actions
- [ ] **Audio feedback** - Optional sounds for critical alerts

### Tasks

1. [ ] Create `design_tokens.dart` with all constants (4h)
   ```dart
   class DesignTokens {
     // Colors
     static const primaryRed = Color(0xFFDC2626);
     static const safetyGreen = Color(0xFF16A34A);
     // ... etc

     // Spacing
     static const space4 = 4.0;
     static const space8 = 8.0;
     // ... etc

     // Touch targets
     static const minTouchTarget = 56.0; // CRITICAL: Gloved hands
   }
   ```

2. [ ] Create `app_theme.dart` with ThemeData (4h)
   - Dark theme as default
   - Light theme as alternative
   - High contrast variant

3. [ ] Create `typography.dart` with text styles (2h)

4. [ ] Create `feedback_service.dart` for multi-modal feedback (3h)
   ```dart
   class FeedbackService {
     void success() {
       HapticFeedback.mediumImpact();
       // Optional: play success sound
     }
     void error() {
       HapticFeedback.heavyImpact();
       // Optional: play error sound
     }
     void alert() {
       HapticFeedback.vibrate();
       // Play alert sound
     }
   }
   ```

5. [ ] Create design system documentation (2h)

6. [ ] Create component showcase/demo screen (3h)
   - Preview all tokens, colors, components

### Critical Design Requirements

**From Frontend Specialist**:

1. **56dp Minimum Touch Targets**
   - Responders wear gloves
   - Standard 48dp is insufficient
   - All buttons, toggles, list items must be 56dp+

2. **Dark Mode Default**
   - Outdoor use in variable lighting
   - Reduces eye strain in low light
   - Better visibility in direct sunlight (counterintuitively)

3. **Multi-Modal Feedback**
   - Cannot rely on visual feedback alone
   - Haptic confirms actions without looking
   - Audio for critical alerts (optional, configurable)

4. **High Contrast**
   - Emergency situations = high stress
   - Must be instantly readable
   - Color-blind accessible

### Testing Requirements

- [ ] All tokens documented and accessible
- [ ] Theme applies correctly across app
- [ ] Dark mode renders correctly
- [ ] Touch targets measure 56dp+ in layout inspector
- [ ] Haptic feedback triggers on test device

### Risk Assessment

- **LOW**: Design system is internal work
- **HIGH IMPACT**: Skipping this causes compounding inconsistencies

### Definition of Done

- [ ] All acceptance criteria met
- [ ] Design tokens documented
- [ ] Team can use tokens consistently
- [ ] Component showcase screen functional

---

## Phase 1 Dependencies Graph

```
TODO-001 (Environment Setup)
    │
    ├──> TODO-002 (Architecture ADRs)
    │
    └──> TODO-003 (Flutter Setup)
              │
              └──> TODO-004 (Design System)
```

## Phase 1 Completion Criteria

- [ ] Development environment fully functional
- [ ] All ADRs documented and approved
- [ ] Flutter project builds and runs
- [ ] Design system tokens established
- [ ] Team aligned on architecture decisions

## References

- Roadmap Week 1-2: `/docs/02-plans/00-poc-roadmap.md`
- Architecture Plan: `/docs/02-plans/01-architecture-plan.md`
