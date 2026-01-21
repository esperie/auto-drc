# TODO-030 to TODO-034: AI Agents Phase (Week 7-8)

**Phase**: 4 - AI Agents
**Timeline**: Week 7-8
**Status**: PENDING

**CRITICAL CORRECTIONS FROM KAIZEN SPECIALIST**:
1. BaseAgent init requires `config=` and `signature=` parameters
2. Create BaseEdgeAgent pattern for offline-first design
3. Call agents OUTSIDE workflows (PythonCode node cannot await)
4. Use simple coordinator pattern, NOT full multi-agent orchestration

---

## TODO-030: BaseEdgeAgent Pattern

**Status**: PENDING
**Owner**: AI Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-020 (DataFlow Models)

### Description

Create the BaseEdgeAgent pattern that provides offline-first AI capabilities with automatic rule-based fallbacks. This is the foundation for all ADRC AI agents.

### Acceptance Criteria

- [ ] BaseEdgeAgent class with offline-first design
- [ ] Automatic fallback to rule-based logic when LLM unavailable
- [ ] Configurable LLM/rule-based mode
- [ ] Proper BaseAgent initialization (config=, signature=)
- [ ] Audit logging for all decisions
- [ ] Battery/resource awareness

### Critical Kaizen Corrections

```python
# CORRECT: BaseAgent requires config= and signature= parameters
from kaizen.agent import BaseAgent, AgentConfig
from kaizen.signatures import Signature, InputField, OutputField

class MyAgent(BaseAgent):
    def __init__(self, db, config: AgentConfig = None):
        # CORRECT: Pass config and signature
        super().__init__(
            config=config or AgentConfig(
                name="MyAgent",
                model="gpt-4o-mini",  # or local model
                temperature=0.1
            ),
            signature=MySignature  # Required!
        )
        self.db = db

# WRONG: Missing config and signature
class MyAgent(BaseAgent):
    def __init__(self, db):
        super().__init__()  # WRONG! Missing required params
        self.db = db
```

### BaseEdgeAgent Implementation

```python
# src/velus/agents/base_edge_agent.py
from abc import ABC, abstractmethod
from typing import Any, Dict, Optional
from dataclasses import dataclass
from datetime import datetime, UTC
import logging

from kaizen.agent import BaseAgent, AgentConfig
from kaizen.signatures import Signature

logger = logging.getLogger(__name__)


@dataclass
class EdgeAgentConfig:
    """Configuration for edge-deployed agents."""
    name: str
    model: str = "gpt-4o-mini"  # Default to small, fast model
    temperature: float = 0.1    # Low temp for consistent decisions
    max_retries: int = 3
    timeout_seconds: float = 5.0
    fallback_mode: str = "rule_based"  # "rule_based" | "cached" | "fail"
    offline_mode: bool = False  # Force offline (rule-based) mode
    log_decisions: bool = True  # Audit trail


class BaseEdgeAgent(BaseAgent, ABC):
    """
    Base class for edge-deployed AI agents with offline-first design.

    Features:
    - Automatic fallback to rule-based logic when LLM unavailable
    - Battery/resource awareness
    - Audit logging for all decisions
    - Configurable online/offline modes
    """

    def __init__(
        self,
        signature: type[Signature],
        db,
        config: EdgeAgentConfig = None
    ):
        self.edge_config = config or EdgeAgentConfig(name=self.__class__.__name__)

        # Initialize parent with Kaizen AgentConfig
        kaizen_config = AgentConfig(
            name=self.edge_config.name,
            model=self.edge_config.model,
            temperature=self.edge_config.temperature,
            max_retries=self.edge_config.max_retries,
            timeout=self.edge_config.timeout_seconds
        )
        super().__init__(config=kaizen_config, signature=signature)

        self.db = db
        self._llm_available = not self.edge_config.offline_mode
        self._decision_log = []

    async def decide(self, **inputs) -> Dict[str, Any]:
        """
        Make a decision with automatic fallback.

        1. Try LLM-based decision if available
        2. Fall back to rule-based if LLM fails
        3. Log decision for audit
        """
        decision_record = {
            "agent": self.edge_config.name,
            "timestamp": datetime.now(UTC).isoformat(),
            "inputs": inputs,
            "method": None,
            "output": None,
            "error": None
        }

        try:
            if self._llm_available and not self.edge_config.offline_mode:
                # Try LLM-based decision
                decision_record["method"] = "llm"
                result = await self._llm_decide(**inputs)
                decision_record["output"] = result
            else:
                # Use rule-based fallback
                decision_record["method"] = "rule_based"
                result = await self._rule_based_decide(**inputs)
                decision_record["output"] = result

        except Exception as e:
            logger.warning(f"{self.edge_config.name} LLM failed: {e}, using fallback")
            decision_record["error"] = str(e)

            # Fallback to rule-based
            if self.edge_config.fallback_mode == "rule_based":
                decision_record["method"] = "rule_based_fallback"
                result = await self._rule_based_decide(**inputs)
                decision_record["output"] = result
            elif self.edge_config.fallback_mode == "fail":
                raise
            else:
                result = {"error": "No decision available", "fallback": True}
                decision_record["output"] = result

        # Log decision for audit
        if self.edge_config.log_decisions:
            self._log_decision(decision_record)

        return result

    async def _llm_decide(self, **inputs) -> Dict[str, Any]:
        """LLM-based decision using Kaizen signature."""
        # Use parent class forward() method
        result = await self.forward(**inputs)
        return result

    @abstractmethod
    async def _rule_based_decide(self, **inputs) -> Dict[str, Any]:
        """
        Rule-based fallback decision logic.

        MUST be implemented by subclasses.
        This provides offline functionality when LLM is unavailable.
        """
        pass

    def _log_decision(self, record: Dict):
        """Log decision for audit trail."""
        self._decision_log.append(record)
        logger.info(f"Decision: {record['agent']} -> {record['method']}")

        # Keep bounded log
        if len(self._decision_log) > 1000:
            self._decision_log = self._decision_log[-500:]

    def get_decision_log(self) -> list:
        """Get recent decision log for audit."""
        return self._decision_log.copy()

    def set_offline_mode(self, offline: bool):
        """Toggle offline mode (forces rule-based decisions)."""
        self.edge_config.offline_mode = offline
        self._llm_available = not offline

    def check_llm_availability(self) -> bool:
        """Check if LLM is available (network, API key, etc.)."""
        # Implement actual check
        return self._llm_available
```

### Tasks

1. [ ] Create `src/velus/agents/__init__.py` (30m)
2. [ ] Implement `EdgeAgentConfig` dataclass (1h)
3. [ ] Implement `BaseEdgeAgent` abstract class (4h)
4. [ ] Add decision logging and audit trail (2h)
5. [ ] Add battery/resource awareness hooks (2h)
6. [ ] Write base agent tests (3h)

### Testing Requirements

- [ ] Agent initializes with correct parameters
- [ ] LLM decision works when available
- [ ] Fallback triggers when LLM fails
- [ ] Offline mode forces rule-based
- [ ] Decision log captures all decisions
- [ ] Audit trail is complete

### Risk Assessment

- **MEDIUM**: Agent architecture is foundational
- **Mitigation**: Thorough testing, clear abstractions

### Definition of Done

- [ ] BaseEdgeAgent class implemented
- [ ] All tests pass
- [ ] Offline-first behavior verified

---

## TODO-031: Task Routing Agent

**Status**: PENDING
**Owner**: AI Dev
**Estimated Effort**: 3 days
**Dependencies**: TODO-030 (BaseEdgeAgent)

### Description

Implement Task Routing Agent that assigns tasks to the best available responder based on skills, proximity, and workload.

### Acceptance Criteria

- [ ] TaskRoutingSignature with proper fields
- [ ] TaskRouterAgent extends BaseEdgeAgent
- [ ] LLM-based routing for complex decisions
- [ ] Rule-based scoring fallback (always works)
- [ ] Considers: proximity, skills, workload, role fit
- [ ] Returns ranked recommendations

### Implementation

```python
# src/velus/agents/task_router.py
from typing import List, Optional
from kaizen.signatures import Signature, InputField, OutputField
from .base_edge_agent import BaseEdgeAgent, EdgeAgentConfig
import json
from math import radians, cos, sin, asin, sqrt


class TaskRoutingSignature(Signature):
    """Route a task to the best available responder."""

    task_type: str = InputField(desc="Type of task (rescue, medical, logistics, recon)")
    priority: str = InputField(desc="Task priority (critical, high, medium, low)")
    location_lat: float = InputField(desc="Task latitude")
    location_lng: float = InputField(desc="Task longitude")
    required_skills: List[str] = InputField(
        desc="Skills needed for task",
        default=[]
    )
    available_responders: List[dict] = InputField(
        desc="List of available responders with skills, location, workload"
    )

    selected_responder_id: str = OutputField(desc="ID of selected responder")
    confidence: float = OutputField(desc="Confidence in selection (0-1)")
    reasoning: str = OutputField(desc="Why this responder was selected")
    alternatives: List[str] = OutputField(desc="Alternative responder IDs")


class TaskRouterAgent(BaseEdgeAgent):
    """
    AI agent for intelligent task routing.

    Uses LLM for complex multi-factor decisions.
    Falls back to weighted scoring algorithm when offline.
    """

    # Role-task fit matrix (used in rule-based fallback)
    ROLE_FIT = {
        "medical": {"medic": 1.0, "usar": 0.3, "logistics": 0.1, "command": 0.2},
        "rescue": {"usar": 1.0, "medic": 0.5, "logistics": 0.2, "command": 0.2},
        "logistics": {"logistics": 1.0, "usar": 0.3, "medic": 0.2, "command": 0.3},
        "recon": {"usar": 0.8, "medic": 0.4, "logistics": 0.5, "command": 0.6},
    }

    def __init__(self, db, config: EdgeAgentConfig = None):
        default_config = EdgeAgentConfig(
            name="TaskRouterAgent",
            model="gpt-4o-mini",
            temperature=0.1,
            timeout_seconds=3.0  # Fast response needed
        )
        super().__init__(
            signature=TaskRoutingSignature,
            db=db,
            config=config or default_config
        )

    async def route_task(
        self,
        task_type: str,
        priority: str,
        location: tuple,
        required_skills: List[str] = None
    ) -> dict:
        """
        Route a task to the best responder.

        Args:
            task_type: Type of task
            priority: Priority level
            location: (lat, lng) tuple
            required_skills: Optional list of required skills

        Returns:
            dict with responder_id, confidence, reasoning, alternatives
        """
        # Fetch available responders
        responders = await self.db.express.list(
            "Responder",
            filter={"status": "available"}
        )

        if not responders:
            return {
                "responder_id": None,
                "confidence": 0.0,
                "reasoning": "No available responders",
                "alternatives": []
            }

        # Use decide() which handles LLM/fallback automatically
        result = await self.decide(
            task_type=task_type,
            priority=priority,
            location_lat=location[0],
            location_lng=location[1],
            required_skills=required_skills or [],
            available_responders=responders
        )

        return result

    async def _rule_based_decide(self, **inputs) -> dict:
        """
        Rule-based task routing fallback.

        Uses weighted scoring:
        - 40% proximity (closer is better)
        - 30% skills match
        - 20% role fit
        - 10% workload (lower is better)
        """
        responders = inputs["available_responders"]
        task_type = inputs["task_type"]
        location = (inputs["location_lat"], inputs["location_lng"])
        required_skills = inputs.get("required_skills", [])

        if not responders:
            return {
                "selected_responder_id": None,
                "confidence": 0.0,
                "reasoning": "No available responders",
                "alternatives": []
            }

        scored = []
        for r in responders:
            score = self._calculate_score(r, task_type, location, required_skills)
            scored.append((r, score))

        # Sort by score (highest first)
        scored.sort(key=lambda x: x[1], reverse=True)

        best = scored[0]
        alternatives = [s[0]["id"] for s in scored[1:4]]

        return {
            "selected_responder_id": best[0]["id"],
            "confidence": min(best[1], 1.0),
            "reasoning": f"Score: {best[1]:.2f} (proximity, skills, role fit)",
            "alternatives": alternatives
        }

    def _calculate_score(
        self,
        responder: dict,
        task_type: str,
        location: tuple,
        required_skills: List[str]
    ) -> float:
        """Calculate weighted score for a responder."""
        score = 0.0

        # 1. Proximity (40%) - closer is better
        distance = self._haversine_distance(
            (responder["location_lat"], responder["location_lng"]),
            location
        )
        # Normalize: 0km = 1.0, 10km = 0.0
        proximity_score = max(0, 1.0 - (distance / 10.0))
        score += 0.4 * proximity_score

        # 2. Skills match (30%)
        if required_skills:
            try:
                responder_skills = set(json.loads(responder.get("skills", "[]")))
            except (json.JSONDecodeError, TypeError):
                responder_skills = set()
            skill_match = len(set(required_skills) & responder_skills) / len(required_skills)
        else:
            skill_match = 1.0  # No skills required
        score += 0.3 * skill_match

        # 3. Role fit (20%)
        role = responder.get("role", "")
        role_fit = self.ROLE_FIT.get(task_type, {}).get(role, 0.2)
        score += 0.2 * role_fit

        # 4. Workload (10%) - would need task count, using placeholder
        # Lower workload = higher score
        workload_score = 0.8  # Placeholder - would check task count
        score += 0.1 * workload_score

        return score

    def _haversine_distance(self, coord1: tuple, coord2: tuple) -> float:
        """Calculate distance in km between two coordinates."""
        lat1, lon1 = radians(coord1[0]), radians(coord1[1])
        lat2, lon2 = radians(coord2[0]), radians(coord2[1])
        dlat = lat2 - lat1
        dlon = lon2 - lon1
        a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
        return 2 * 6371 * asin(sqrt(a))  # km
```

### Tasks

1. [ ] Create `TaskRoutingSignature` (1h)
2. [ ] Implement `TaskRouterAgent` class (4h)
3. [ ] Implement rule-based scoring algorithm (4h)
4. [ ] Add skill matching logic (2h)
5. [ ] Add role-fit scoring (1h)
6. [ ] Write comprehensive tests (4h)
7. [ ] Test offline fallback behavior (2h)

### Testing Requirements

- [ ] Agent routes to closest qualified responder
- [ ] Skills matching works correctly
- [ ] Role fit influences selection
- [ ] Fallback works when LLM unavailable
- [ ] Alternatives provided
- [ ] Empty responder list handled

### Risk Assessment

- **LOW**: Well-defined algorithm
- **MEDIUM**: LLM prompt engineering for complex scenarios

### Definition of Done

- [ ] TaskRouterAgent fully implemented
- [ ] All tests pass
- [ ] Offline routing works reliably

---

## TODO-032: Resource Matching Agent

**Status**: PENDING
**Owner**: AI Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-030 (BaseEdgeAgent)

### Description

Implement Resource Matching Agent that finds and recommends available resources based on type, proximity, and quantity needs.

### Acceptance Criteria

- [ ] ResourceMatchingSignature with proper fields
- [ ] ResourceMatcherAgent extends BaseEdgeAgent
- [ ] Matches resources by type and subtype
- [ ] Considers proximity and quantity
- [ ] Rule-based fallback for offline operation

### Implementation

```python
# src/velus/agents/resource_matcher.py
from typing import List, Optional
from kaizen.signatures import Signature, InputField, OutputField
from .base_edge_agent import BaseEdgeAgent, EdgeAgentConfig
from math import radians, cos, sin, asin, sqrt


class ResourceMatchingSignature(Signature):
    """Match resource needs with available resources."""

    resource_type: str = InputField(desc="Type of resource needed (vehicle, medical, equipment)")
    subtype: Optional[str] = InputField(desc="Specific subtype (ambulance, stretcher, etc.)", default=None)
    quantity_needed: int = InputField(desc="Quantity needed", default=1)
    location_lat: float = InputField(desc="Location latitude")
    location_lng: float = InputField(desc="Location longitude")
    max_distance_km: float = InputField(desc="Maximum acceptable distance in km", default=10.0)

    matched_resources: List[dict] = OutputField(desc="List of matched resources sorted by proximity")
    total_available: int = OutputField(desc="Total quantity available within range")
    can_fulfill: bool = OutputField(desc="Whether request can be fully fulfilled")
    recommendation: str = OutputField(desc="Recommendation for resource acquisition")


class ResourceMatcherAgent(BaseEdgeAgent):
    """
    AI agent for matching resource needs with availability.

    Features:
    - Type and subtype matching
    - Proximity-based sorting
    - Quantity aggregation
    - Fulfillment assessment
    """

    def __init__(self, db, config: EdgeAgentConfig = None):
        default_config = EdgeAgentConfig(
            name="ResourceMatcherAgent",
            model="gpt-4o-mini",
            temperature=0.1,
            timeout_seconds=3.0
        )
        super().__init__(
            signature=ResourceMatchingSignature,
            db=db,
            config=config or default_config
        )

    async def match_resources(
        self,
        resource_type: str,
        location: tuple,
        quantity_needed: int = 1,
        subtype: str = None,
        max_distance_km: float = 10.0
    ) -> dict:
        """
        Find and recommend resources matching the request.

        Args:
            resource_type: Type of resource needed
            location: (lat, lng) tuple
            quantity_needed: How many needed
            subtype: Optional specific subtype
            max_distance_km: Maximum search radius

        Returns:
            dict with matched_resources, total_available, can_fulfill, recommendation
        """
        result = await self.decide(
            resource_type=resource_type,
            subtype=subtype,
            quantity_needed=quantity_needed,
            location_lat=location[0],
            location_lng=location[1],
            max_distance_km=max_distance_km
        )

        return result

    async def _rule_based_decide(self, **inputs) -> dict:
        """Rule-based resource matching fallback."""
        resource_type = inputs["resource_type"]
        subtype = inputs.get("subtype")
        quantity_needed = inputs["quantity_needed"]
        location = (inputs["location_lat"], inputs["location_lng"])
        max_distance = inputs["max_distance_km"]

        # Build filter
        query_filter = {
            "type": resource_type,
            "status": "available"
        }
        if subtype:
            query_filter["subtype"] = subtype

        # Query available resources
        resources = await self.db.express.list("Resource", filter=query_filter)

        # Calculate distances and filter by range
        matched = []
        for r in resources:
            distance = self._haversine_distance(
                (r["location_lat"], r["location_lng"]),
                location
            )
            if distance <= max_distance:
                r["distance_km"] = round(distance, 2)
                matched.append(r)

        # Sort by distance
        matched.sort(key=lambda x: x["distance_km"])

        # Calculate totals
        total_available = sum(r.get("quantity", 1) for r in matched)
        can_fulfill = total_available >= quantity_needed

        # Generate recommendation
        if can_fulfill:
            recommendation = f"Request can be fulfilled. {len(matched)} sources within {max_distance}km."
        elif total_available > 0:
            recommendation = f"Partial fulfillment possible. {total_available}/{quantity_needed} available."
        else:
            recommendation = f"No {resource_type} available within {max_distance}km. Expand search area."

        return {
            "matched_resources": matched[:10],  # Top 10
            "total_available": total_available,
            "can_fulfill": can_fulfill,
            "recommendation": recommendation
        }

    def _haversine_distance(self, coord1: tuple, coord2: tuple) -> float:
        """Calculate distance in km between two coordinates."""
        lat1, lon1 = radians(coord1[0]), radians(coord1[1])
        lat2, lon2 = radians(coord2[0]), radians(coord2[1])
        dlat = lat2 - lat1
        dlon = lon2 - lon1
        a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
        return 2 * 6371 * asin(sqrt(a))
```

### Tasks

1. [ ] Create `ResourceMatchingSignature` (1h)
2. [ ] Implement `ResourceMatcherAgent` class (3h)
3. [ ] Implement proximity-based filtering (2h)
4. [ ] Add quantity aggregation logic (1h)
5. [ ] Write tests (3h)

### Testing Requirements

- [ ] Finds resources by type
- [ ] Filters by subtype when specified
- [ ] Filters by distance
- [ ] Calculates total availability
- [ ] Correctly assesses fulfillment
- [ ] Handles no results gracefully

### Definition of Done

- [ ] ResourceMatcherAgent fully implemented
- [ ] All tests pass
- [ ] Offline matching works reliably

---

## TODO-033: Situation Summary Agent

**Status**: PENDING
**Owner**: AI Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-030 (BaseEdgeAgent)

### Description

Implement Situation Summary Agent that aggregates recent events and generates concise situational awareness summaries.

### Acceptance Criteria

- [ ] SituationSummarySignature with proper fields
- [ ] SituationSummaryAgent extends BaseEdgeAgent
- [ ] Aggregates incidents, tasks, responder status
- [ ] Generates human-readable summary
- [ ] Rule-based template fallback

### Implementation

```python
# src/velus/agents/situation_analyzer.py
from typing import List, Optional
from kaizen.signatures import Signature, InputField, OutputField
from .base_edge_agent import BaseEdgeAgent, EdgeAgentConfig
from datetime import datetime, UTC, timedelta


class SituationSummarySignature(Signature):
    """Generate situational awareness summary."""

    incidents: List[dict] = InputField(desc="Recent incidents")
    tasks: List[dict] = InputField(desc="Active tasks")
    responders: List[dict] = InputField(desc="Responder statuses")
    time_window_minutes: int = InputField(desc="Time window for 'recent'", default=60)

    summary: str = OutputField(desc="Human-readable situation summary")
    critical_issues: List[str] = OutputField(desc="Critical issues requiring attention")
    statistics: dict = OutputField(desc="Key statistics")
    recommendations: List[str] = OutputField(desc="Recommended actions")


class SituationSummaryAgent(BaseEdgeAgent):
    """
    AI agent for generating situational awareness summaries.

    Aggregates data from incidents, tasks, and responders to provide
    commanders with a quick operational overview.
    """

    def __init__(self, db, config: EdgeAgentConfig = None):
        default_config = EdgeAgentConfig(
            name="SituationSummaryAgent",
            model="gpt-4o-mini",
            temperature=0.3,  # Slightly more creative for summaries
            timeout_seconds=5.0
        )
        super().__init__(
            signature=SituationSummarySignature,
            db=db,
            config=config or default_config
        )

    async def generate_summary(self, time_window_minutes: int = 60) -> dict:
        """
        Generate situational summary for the given time window.

        Args:
            time_window_minutes: How far back to look

        Returns:
            dict with summary, critical_issues, statistics, recommendations
        """
        # Fetch recent data
        cutoff = (datetime.now(UTC) - timedelta(minutes=time_window_minutes)).isoformat()

        incidents = await self.db.express.list("Incident", filter={"status": "active"})
        tasks = await self.db.express.list("Task", filter={"status": {"$in": ["pending", "assigned", "in_progress"]}})
        responders = await self.db.express.list("Responder")

        result = await self.decide(
            incidents=incidents,
            tasks=tasks,
            responders=responders,
            time_window_minutes=time_window_minutes
        )

        return result

    async def _rule_based_decide(self, **inputs) -> dict:
        """Rule-based situation summary fallback."""
        incidents = inputs["incidents"]
        tasks = inputs["tasks"]
        responders = inputs["responders"]

        # Calculate statistics
        stats = self._calculate_statistics(incidents, tasks, responders)

        # Identify critical issues
        critical = self._identify_critical_issues(incidents, tasks, responders)

        # Generate template-based summary
        summary = self._generate_template_summary(stats, critical)

        # Generate recommendations
        recommendations = self._generate_recommendations(stats, critical)

        return {
            "summary": summary,
            "critical_issues": critical,
            "statistics": stats,
            "recommendations": recommendations
        }

    def _calculate_statistics(self, incidents, tasks, responders) -> dict:
        """Calculate key operational statistics."""
        # Incident stats
        incident_by_severity = {}
        for i in incidents:
            sev = i.get("severity", "unknown")
            incident_by_severity[sev] = incident_by_severity.get(sev, 0) + 1

        # Task stats
        task_by_status = {}
        for t in tasks:
            status = t.get("status", "unknown")
            task_by_status[status] = task_by_status.get(status, 0) + 1

        # Responder stats
        responder_by_status = {}
        for r in responders:
            status = r.get("status", "unknown")
            responder_by_status[status] = responder_by_status.get(status, 0) + 1

        return {
            "total_incidents": len(incidents),
            "incidents_by_severity": incident_by_severity,
            "total_tasks": len(tasks),
            "tasks_by_status": task_by_status,
            "total_responders": len(responders),
            "responders_by_status": responder_by_status,
            "available_responders": responder_by_status.get("available", 0),
            "engaged_responders": responder_by_status.get("engaged", 0)
        }

    def _identify_critical_issues(self, incidents, tasks, responders) -> List[str]:
        """Identify issues requiring immediate attention."""
        critical = []

        # Critical incidents
        critical_incidents = [i for i in incidents if i.get("severity") == "critical"]
        if critical_incidents:
            critical.append(f"{len(critical_incidents)} critical incident(s) active")

        # Unassigned high-priority tasks
        pending_high = [t for t in tasks
                       if t.get("status") == "pending"
                       and t.get("priority") in ["critical", "high"]]
        if pending_high:
            critical.append(f"{len(pending_high)} high-priority task(s) unassigned")

        # Low responder availability
        available = sum(1 for r in responders if r.get("status") == "available")
        total = len(responders)
        if total > 0 and available / total < 0.2:
            critical.append(f"Low responder availability ({available}/{total})")

        # Offline responders
        offline = sum(1 for r in responders if r.get("status") == "offline")
        if offline > 0:
            critical.append(f"{offline} responder(s) offline")

        return critical

    def _generate_template_summary(self, stats: dict, critical: List[str]) -> str:
        """Generate template-based summary."""
        summary_parts = []

        # Header
        summary_parts.append(f"SITUATION REPORT")
        summary_parts.append("-" * 40)

        # Incidents
        summary_parts.append(f"Active Incidents: {stats['total_incidents']}")
        if stats['incidents_by_severity']:
            sev_str = ", ".join(f"{k}: {v}" for k, v in stats['incidents_by_severity'].items())
            summary_parts.append(f"  By severity: {sev_str}")

        # Tasks
        summary_parts.append(f"Active Tasks: {stats['total_tasks']}")
        if stats['tasks_by_status']:
            status_str = ", ".join(f"{k}: {v}" for k, v in stats['tasks_by_status'].items())
            summary_parts.append(f"  By status: {status_str}")

        # Responders
        summary_parts.append(f"Responders: {stats['available_responders']} available, "
                           f"{stats['engaged_responders']} engaged, "
                           f"{stats['total_responders']} total")

        # Critical issues
        if critical:
            summary_parts.append("")
            summary_parts.append("CRITICAL ISSUES:")
            for issue in critical:
                summary_parts.append(f"  - {issue}")

        return "\n".join(summary_parts)

    def _generate_recommendations(self, stats: dict, critical: List[str]) -> List[str]:
        """Generate action recommendations based on situation."""
        recommendations = []

        # Low availability
        if stats.get("available_responders", 0) < 3:
            recommendations.append("Request additional responder deployment")

        # Unassigned tasks
        pending = stats.get("tasks_by_status", {}).get("pending", 0)
        if pending > 5:
            recommendations.append(f"Review {pending} pending tasks for assignment")

        # Critical incidents
        critical_count = stats.get("incidents_by_severity", {}).get("critical", 0)
        if critical_count > 0:
            recommendations.append(f"Prioritize {critical_count} critical incident(s)")

        if not recommendations:
            recommendations.append("Operations proceeding normally")

        return recommendations
```

### Tasks

1. [ ] Create `SituationSummarySignature` (1h)
2. [ ] Implement `SituationSummaryAgent` class (4h)
3. [ ] Implement statistics calculation (2h)
4. [ ] Implement critical issue detection (2h)
5. [ ] Implement template summary generation (2h)
6. [ ] Write tests (3h)

### Testing Requirements

- [ ] Statistics calculated correctly
- [ ] Critical issues identified
- [ ] Summary is human-readable
- [ ] Recommendations are actionable
- [ ] Works with empty data

### Definition of Done

- [ ] SituationSummaryAgent fully implemented
- [ ] All tests pass
- [ ] Summary readable and useful

---

## TODO-034: Agent-Workflow Integration

**Status**: PENDING
**Owner**: AI Dev
**Estimated Effort**: 2 days
**Dependencies**: TODO-031 (Task Router), TODO-021 (Workflows)

### Description

Integrate AI agents with Kailash workflows. **CRITICAL**: Agents must be called OUTSIDE workflows because PythonCode nodes cannot await async operations.

### Acceptance Criteria

- [ ] Agent orchestration layer that calls agents before/after workflows
- [ ] Proper async handling outside workflow execution
- [ ] Simple coordinator pattern (not multi-agent)
- [ ] Clear separation between agent decisions and workflow execution

### Critical Pattern: Agents OUTSIDE Workflows

```python
# WRONG: Calling agent inside PythonCode node
workflow.add_node("PythonCode", "route_task", {
    "code": """
# This will FAIL! PythonCode cannot await
agent = TaskRouterAgent(db)
result = await agent.route_task(...)  # ERROR!
"""
})

# CORRECT: Call agent outside workflow, pass result as input
async def assign_task_with_ai(task_data: dict, db):
    # 1. Call agent OUTSIDE workflow
    agent = TaskRouterAgent(db)
    routing = await agent.route_task(
        task_type=task_data["type"],
        priority=task_data["priority"],
        location=(task_data["lat"], task_data["lng"])
    )

    # 2. Pass agent result as workflow input
    workflow = create_task_assignment_workflow()
    runtime = AsyncLocalRuntime()

    results, run_id = await runtime.execute_workflow_async(
        workflow.build(),
        inputs={
            **task_data,
            "responder_id": routing["selected_responder_id"]  # From agent
        }
    )

    return results
```

### Agent Orchestrator

```python
# src/velus/agents/orchestrator.py
from typing import Dict, Any, Optional
from kailash.runtime import AsyncLocalRuntime
from velus.agents.task_router import TaskRouterAgent
from velus.agents.resource_matcher import ResourceMatcherAgent
from velus.agents.situation_analyzer import SituationSummaryAgent
from velus.workflows import (
    create_task_assignment_workflow,
    create_status_update_workflow,
    create_incident_report_workflow
)
import logging

logger = logging.getLogger(__name__)


class AgentOrchestrator:
    """
    Simple orchestrator for agent + workflow coordination.

    Pattern:
    1. Receive request
    2. Call appropriate agent(s) for decision support
    3. Execute workflow with agent recommendations
    4. Return combined result

    NOTE: This is a SIMPLE coordinator, not multi-agent orchestration.
    """

    def __init__(self, db):
        self.db = db
        self.runtime = AsyncLocalRuntime()

        # Initialize agents (lazy to save resources)
        self._task_router: Optional[TaskRouterAgent] = None
        self._resource_matcher: Optional[ResourceMatcherAgent] = None
        self._situation_analyzer: Optional[SituationSummaryAgent] = None

    @property
    def task_router(self) -> TaskRouterAgent:
        if self._task_router is None:
            self._task_router = TaskRouterAgent(self.db)
        return self._task_router

    @property
    def resource_matcher(self) -> ResourceMatcherAgent:
        if self._resource_matcher is None:
            self._resource_matcher = ResourceMatcherAgent(self.db)
        return self._resource_matcher

    @property
    def situation_analyzer(self) -> SituationSummaryAgent:
        if self._situation_analyzer is None:
            self._situation_analyzer = SituationSummaryAgent(self.db)
        return self._situation_analyzer

    async def create_and_assign_task(
        self,
        task_id: str,
        task_type: str,
        priority: str,
        location: tuple,
        description: str,
        commander_id: str,
        required_skills: list = None
    ) -> Dict[str, Any]:
        """
        Create a task with AI-assisted responder assignment.

        Flow:
        1. Call TaskRouterAgent to get best responder
        2. Execute task assignment workflow with responder
        3. Return task + assignment details
        """
        # Step 1: Get AI recommendation (OUTSIDE workflow)
        routing = await self.task_router.route_task(
            task_type=task_type,
            priority=priority,
            location=location,
            required_skills=required_skills or []
        )

        logger.info(f"AI routing: {routing}")

        if not routing.get("selected_responder_id"):
            return {
                "success": False,
                "error": "No suitable responder found",
                "ai_reasoning": routing.get("reasoning")
            }

        # Step 2: Execute workflow with AI recommendation
        from datetime import datetime, UTC
        workflow = create_task_assignment_workflow()

        results, run_id = await self.runtime.execute_workflow_async(
            workflow.build(),
            inputs={
                "task_id": task_id,
                "task_type": task_type,
                "priority": priority,
                "location_lat": location[0],
                "location_lng": location[1],
                "description": description,
                "commander_id": commander_id,
                "responder_id": routing["selected_responder_id"],
                "timestamp": datetime.now(UTC).isoformat()
            }
        )

        return {
            "success": True,
            "task_id": task_id,
            "assigned_to": routing["selected_responder_id"],
            "ai_confidence": routing.get("confidence"),
            "ai_reasoning": routing.get("reasoning"),
            "alternatives": routing.get("alternatives", []),
            "workflow_run_id": run_id
        }

    async def find_resources(
        self,
        resource_type: str,
        location: tuple,
        quantity_needed: int = 1,
        subtype: str = None,
        max_distance_km: float = 10.0
    ) -> Dict[str, Any]:
        """
        Find resources with AI-assisted matching.
        """
        result = await self.resource_matcher.match_resources(
            resource_type=resource_type,
            location=location,
            quantity_needed=quantity_needed,
            subtype=subtype,
            max_distance_km=max_distance_km
        )

        return result

    async def get_situation_summary(self, time_window_minutes: int = 60) -> Dict[str, Any]:
        """
        Generate AI-assisted situation summary.
        """
        result = await self.situation_analyzer.generate_summary(
            time_window_minutes=time_window_minutes
        )

        return result

    def set_offline_mode(self, offline: bool):
        """Set all agents to offline mode (rule-based only)."""
        if self._task_router:
            self._task_router.set_offline_mode(offline)
        if self._resource_matcher:
            self._resource_matcher.set_offline_mode(offline)
        if self._situation_analyzer:
            self._situation_analyzer.set_offline_mode(offline)
```

### Tasks

1. [ ] Create `AgentOrchestrator` class (4h)
2. [ ] Implement `create_and_assign_task` flow (3h)
3. [ ] Implement `find_resources` flow (1h)
4. [ ] Implement `get_situation_summary` flow (1h)
5. [ ] Add offline mode toggle (1h)
6. [ ] Write integration tests (4h)

### Testing Requirements

- [ ] Task creation with AI routing works
- [ ] Resource finding with AI works
- [ ] Situation summary generation works
- [ ] Offline mode disables LLM calls
- [ ] Workflow execution receives agent outputs
- [ ] Error handling for agent failures

### Risk Assessment

- **MEDIUM**: Integration complexity
- **Mitigation**: Clear separation of concerns, thorough testing

### Definition of Done

- [ ] AgentOrchestrator fully implemented
- [ ] All agents integrated with workflows
- [ ] All tests pass
- [ ] Offline mode verified

---

## Phase 4 Dependencies Graph

```
TODO-030 (BaseEdgeAgent)
    │
    ├──> TODO-031 (Task Router)
    │         │
    │         └──> TODO-034 (Agent-Workflow Integration)
    │
    ├──> TODO-032 (Resource Matcher)
    │         │
    │         └──> TODO-034
    │
    └──> TODO-033 (Situation Summary)
              │
              └──> TODO-034
```

## Phase 4 Completion Criteria

- [ ] BaseEdgeAgent pattern implemented
- [ ] All 3 agents functional with fallbacks
- [ ] Agent-workflow integration working
- [ ] Offline mode works reliably
- [ ] **GO/NO-GO GATE**: AI routing demonstrated

## References

- Architecture Plan: `/docs/02-plans/01-architecture-plan.md` (Section 5)
- Kaizen Documentation: CLAUDE.md Kaizen section
