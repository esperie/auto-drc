# Stakeholder Analysis

## Overview

The ADRC platform operates within a complex multi-stakeholder environment. Understanding each stakeholder's needs, concerns, and influence is critical for successful adoption and operation.

---

## 1. Stakeholder Map

### 1.1 Platform Model Mapping

```
                              ADRC PLATFORM
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
    PRODUCERS              CONSUMERS/PRODUCERS          PARTNERS
         │                         │                         │
    ┌────┴────┐              ┌─────┴─────┐              ┌────┴────┐
    │         │              │           │              │         │
 Sensors  Command      Responders  Civilians       Tech      ASEAN
  IoT     Centers      (dual role)                 Vendors    Bodies
```

### 1.2 Stakeholder Categories

| Category | Stakeholders | Platform Role |
|----------|-------------|---------------|
| **Government** | SCDF, NDCC, SAF, Foreign Affairs | Sponsor, User |
| **Military** | SAF HADR units, Partner nation militaries | User, Data Source |
| **Humanitarian** | Singapore Red Cross, ICRC, OCHA | User, Policy |
| **International** | ASEAN AHA Centre, UN agencies | Standards, Coordination |
| **Technology** | Kailash SDK, Device vendors | Enabler |
| **Affected Population** | Civilians in disaster zones | Consumer |

---

## 2. Detailed Stakeholder Profiles

### 2.1 Singapore Civil Defence Force (SCDF)

**Role**: Primary urban search and rescue, firefighting, emergency medical

**Needs**:
- Real-time team coordination in collapsed structures
- Casualty tracking and triage support
- Resource allocation across multiple incident sites
- Integration with existing SCDF CAD system

**Concerns**:
- Reliability in challenging RF environments
- Battery life for extended operations
- Training burden for new system
- Data security for personnel locations

**Influence**: High - Primary domestic user, defines operational requirements

**Success Criteria**:
- Faster response times
- Better resource utilization
- Reduced cognitive load on incident commanders

### 2.2 Singapore Armed Forces (SAF) - HADR Units

**Role**: Large-scale disaster response, overseas deployment, logistics

**Needs**:
- Interoperability with civilian agencies
- Scalable coordination for large-scale operations
- Secure communications for mixed-classification environments
- Integration with military C2 systems

**Concerns**:
- Security classification handling
- Chain of command clarity
- Interoperability with partner nation forces
- Austere environment operation

**Influence**: High - Represents military requirements and international interop

**Success Criteria**:
- Successful multi-agency exercises
- Secure handling of classified information
- Seamless international coordination

### 2.3 ASEAN AHA Centre

**Role**: Regional coordination body, standards setter

**Needs**:
- Compliance with ASEAN SASOP protocols
- Regional data aggregation capabilities
- Support for multi-nation operations
- Training and capacity building

**Concerns**:
- Data sovereignty for member nations
- Equitable access across varying tech maturity
- Standardization vs. national customization
- Sustainability and funding

**Influence**: High - Sets regional standards, facilitates multi-nation adoption

**Success Criteria**:
- Adoption by multiple ASEAN nations
- Integration with existing AHA Centre systems
- Enhanced regional response capability

### 2.4 Field Responders (USAR, Medical, Logistics)

**Role**: End users in the field, both producers and consumers of data

**Needs**:
- Simple, fast interface for high-stress situations
- Clear task assignments and priorities
- Awareness of nearby resources and hazards
- Minimal administrative burden

**Concerns**:
- Device reliability and battery life
- Network coverage in debris/underground
- Information overload vs. missing critical info
- Trust in AI recommendations

**Influence**: Medium - Critical for adoption success, provide feedback

**Success Criteria**:
- Reduced time per task
- Higher confidence in decisions
- Lower cognitive load
- Fewer communication failures

### 2.5 Incident/Operations Commanders

**Role**: Decision-makers coordinating response operations

**Needs**:
- Comprehensive situational awareness
- AI-assisted resource optimization
- Clear chain of command support
- Post-incident analysis capability

**Concerns**:
- Accountability for AI-assisted decisions
- Information accuracy and timeliness
- Override capability for AI recommendations
- Legal/regulatory implications

**Influence**: High - Champions or blockers for adoption

**Success Criteria**:
- Faster, better-informed decisions
- Clear audit trail for accountability
- Successful coordination of complex operations

### 2.6 Affected Population (Civilians)

**Role**: Ultimate beneficiaries, also potential information sources

**Needs**:
- Clear guidance (evacuation routes, shelter locations)
- Ability to request assistance
- Status updates on rescue progress
- Family reunification support

**Concerns**:
- Privacy of location/medical information
- Accessibility for elderly/disabled
- Language barriers
- Battery/device availability

**Influence**: Low on design, High on outcomes

**Success Criteria**:
- Lives saved
- Reduced suffering
- Clear communication
- Trust in response system

### 2.7 Partner Nation Disaster Agencies

**Role**: Regional partners for cross-border operations

**Needs**:
- Interoperability with their existing systems
- Respect for national sovereignty and command
- Language and cultural accommodation
- Training and capacity building support

**Concerns**:
- Data sharing and sovereignty
- Technology dependency
- Compatibility with national protocols
- Funding and sustainability

**Influence**: Medium - Critical for regional network effects

**Success Criteria**:
- Successful joint exercises
- Seamless cross-border coordination
- Mutual capacity enhancement

### 2.8 Technology Partners (Kailash SDK Team)

**Role**: Platform development and support

**Needs**:
- Clear requirements and use cases
- Feedback on framework capabilities
- Real-world deployment validation
- Long-term partnership opportunity

**Concerns**:
- Technical feasibility within timeline
- Edge computing performance requirements
- BLE mesh scalability
- Ongoing support burden

**Influence**: High - Enables or constrains technical capabilities

**Success Criteria**:
- Successful PoC delivery
- Framework improvements from real use
- Sustainable support model

---

## 3. Stakeholder Needs Matrix

### 3.1 Functional Needs

| Stakeholder | Coordination | Situational Awareness | Resource Mgmt | Communication |
|-------------|--------------|----------------------|---------------|---------------|
| SCDF | Critical | Critical | High | Critical |
| SAF HADR | Critical | Critical | Critical | Critical |
| AHA Centre | High | Critical | High | High |
| Responders | High | High | Medium | Critical |
| Commanders | Critical | Critical | Critical | High |
| Civilians | Low | Medium | Low | High |
| Partners | High | High | Medium | Critical |

### 3.2 Non-Functional Needs

| Stakeholder | Security | Reliability | Usability | Scalability |
|-------------|----------|-------------|-----------|-------------|
| SCDF | High | Critical | Critical | High |
| SAF HADR | Critical | Critical | High | Critical |
| AHA Centre | High | High | Medium | Critical |
| Responders | Medium | Critical | Critical | Medium |
| Commanders | High | Critical | High | High |
| Civilians | Medium | High | Critical | Medium |
| Partners | High | High | High | High |

---

## 4. Influence-Interest Matrix

```
                       INTEREST
            Low                         High
         ┌─────────────────┬─────────────────┐
    High │    Monitor      │   Manage        │
         │                 │                 │
         │  • UN OCHA      │  • SCDF         │
INFLUENCE│  • ICRC         │  • SAF HADR     │
         │  • Tech vendors │  • AHA Centre   │
         │                 │  • Commanders   │
         ├─────────────────┼─────────────────┤
    Low  │    Minimal      │   Keep Informed │
         │    Effort       │                 │
         │  • General      │  • Responders   │
         │    public       │  • Civilians    │
         │                 │  • Partners     │
         └─────────────────┴─────────────────┘
```

### 4.1 Engagement Strategy by Quadrant

**Manage Closely (High Influence, High Interest)**:
- SCDF, SAF, AHA Centre, Commanders
- Regular engagement, involve in design decisions
- Address concerns proactively
- Build as champions

**Keep Satisfied (High Influence, Low Interest)**:
- UN OCHA, ICRC, Tech vendors
- Periodic updates, ensure no blocking concerns
- Leverage for credibility and standards compliance

**Keep Informed (Low Influence, High Interest)**:
- Responders, Civilians, Partner nations
- Regular communication, gather feedback
- Focus on user experience and adoption

**Monitor (Low Influence, Low Interest)**:
- General public
- Minimal engagement, public communications only

---

## 5. Stakeholder Concerns & Mitigations

### 5.1 Security Concerns

| Concern | Stakeholders | Mitigation |
|---------|--------------|------------|
| Data classification | SAF, Government | Multi-tier security architecture |
| Personnel tracking | Responders, Unions | Privacy controls, consent model |
| Cross-border data | Partners, AHA | Data sovereignty framework |
| Authentication offline | All | Peer trust with crypto attestation |

### 5.2 Operational Concerns

| Concern | Stakeholders | Mitigation |
|---------|--------------|------------|
| Reliability | All | Redundancy, graceful degradation |
| Battery life | Responders | Power optimization, backup power |
| Training burden | Agencies | Intuitive design, simulation training |
| AI trust | Commanders | Explainable AI, override capability |

### 5.3 Organizational Concerns

| Concern | Stakeholders | Mitigation |
|---------|--------------|------------|
| Role clarity | Multi-agency | Clear command structure support |
| Accountability | Commanders | Audit trails, decision logging |
| Investment ROI | Sponsors | Measurable outcomes, phased approach |
| Sustainability | AHA, Partners | Open source components, capacity building |

---

## 6. Stakeholder Engagement Plan

### 6.1 PoC Phase (3 months)

| Stakeholder | Engagement | Frequency | Owner |
|-------------|------------|-----------|-------|
| SCDF | Design workshops, testing | Weekly | Project Lead |
| SAF HADR | Requirements review | Bi-weekly | Project Lead |
| AHA Centre | Protocol compliance review | Monthly | Project Lead |
| Responders | User testing sessions | Bi-weekly | UX Lead |
| Tech team | Development sprints | Continuous | Tech Lead |

### 6.2 Pilot Phase (Future)

| Stakeholder | Engagement | Frequency | Owner |
|-------------|------------|-----------|-------|
| All domestic | Exercise participation | Quarterly | Operations |
| AHA Centre | Standards development | Monthly | Standards |
| Partners | Capability briefings | Quarterly | International |
| Civilians | Public awareness | As needed | Comms |

---

## 7. RACI Matrix for Key Decisions

| Decision | SCDF | SAF | AHA | Responders | Tech |
|----------|------|-----|-----|------------|------|
| System architecture | C | C | I | I | R/A |
| UI/UX design | C | C | I | C | R/A |
| Security architecture | C | A | I | I | R |
| Protocol compliance | C | C | A | I | R |
| Deployment model | A | C | C | I | R |
| Training approach | A | C | I | C | R |

R = Responsible, A = Accountable, C = Consulted, I = Informed
