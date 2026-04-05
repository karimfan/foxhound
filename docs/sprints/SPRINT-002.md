# Sprint 002: GTM Strategies for Agentic AI Security in the Enterprise

## Overview

The agentic AI security market has validated product-market fit through $2B+ in M&A at median 8.7x multiples, but most startups in this category have not yet validated *go-to-market fit*. The gap between a defensible control point and a repeatable revenue engine is where most early-stage companies will succeed or die in 2026-2028.

This sprint produces a comprehensive GTM framework to be appended to the thesis as Section 10. It answers three questions a VC should ask of every founder: (1) How will you get your first 20 enterprise customers? (2) What makes your GTM structurally different from the last generation of cybersecurity vendors? (3) Who on your team can actually execute this?

The framework is grounded in the thesis's existing market data, control-point map, defensibility dimensions, and investment tiers. It is intended for both VCs evaluating GTM capability and founders building their first 24 months of revenue motion.

## Use Cases

1. **VC diligence on GTM capability**: Evaluate whether a seed/A startup's GTM plan is plausible in a market where platforms bundle baseline controls
2. **Founder GTM playbook**: Blueprint for building GTM from zero to $10-20M ARR with constrained budgets and long enterprise cycles
3. **Buyer mapping**: Clarify how security review gates, compliance mandates, and AI platform teams influence the purchase
4. **GTM team design**: Define AI-native GTM personas with hiring sequencing by stage
5. **Exit-path alignment**: Map GTM motions to the "build to be acquired" dynamic vs. "build to scale"

## Architecture

Analytical framework structure:

```
Section 10: GTM Strategies for Agentic AI Security
├── 10.1 What's Genuinely New
│   ├── Buyer topology shift
│   ├── Security review as demand driver
│   ├── AI-native sales tooling
│   └── Compressed time-to-value
├── 10.2 Five GTM Models
│   ├── A. Security Review Gate
│   ├── B. Threat-Led Growth (TLG)
│   ├── C. Platform Marketplace
│   ├── D. Compliance Mandate
│   └── E. Developer-Led Security
├── 10.3 What Doesn't Work
│   └── Seven anti-patterns with investor red flags
├── 10.4 AI-Native GTM Team Profiles
│   ├── Five persona definitions
│   ├── Hiring sequencing by stage
│   └── Org design principles
├── 10.5 GTM for Acquisition
│   └── Optimizing GTM for strategic exit
├── 10.6 GTM Mapped to Investment Tiers
│   ├── Per-category playbooks
│   └── GTM-to-defensibility cross-reference
└── 10.7 GTM Evaluation Scorecard
    └── Diligence framework for investors
```

## Implementation Plan

### Phase 1: What's Genuinely New in Enterprise GTM for Agentic AI Security (~20%)

**Tasks:**
- [ ] Draft buyer topology comparison (traditional cyber vs. agentic AI security)
- [ ] Articulate the "security review gates AI deployment" demand lever with evidence
- [ ] Document AI-native sales tooling adoption patterns
- [ ] Quantify compressed time-to-value and its GTM implications

**Content:**

**The Buyer Topology Has Changed**

Traditional cybersecurity GTM assumes a single buyer (CISO) with an established budget category. Agentic AI security breaks this:

| Dimension | Traditional Cyber GTM | Agentic AI Security GTM |
|-----------|----------------------|------------------------|
| Primary buyer | CISO (sole authority) | CISO + AI Platform Eng + Compliance (committee) |
| Budget source | Established security line item | Contested: security vs. AI/innovation vs. compliance budget |
| Technical evaluator | SecOps/SOC team | SecOps + ML engineers + platform team (cross-functional) |
| Urgency driver | Active breach / compliance deadline | Agent deployment blockers + regulatory uncertainty |
| Sales cycle trigger | Vendor outreach / analyst report | Security review gates AI rollout (organic pull) |
| Champion profile | Security architect | "AI Security Lead" (new role, often doesn't exist yet) |

**The Security Review Gate as Demand Driver**

The most structurally important GTM dynamic in this market: security review gates AI deployment. When CISO organizations can block agent rollout until security controls are in place, the AI security vendor becomes a *mandatory* purchase, not a discretionary one. This creates inbound demand driven by deployment timelines rather than outbound prospecting.

Evidence: 53% of enterprises plan agentic AI adoption by 2026 (UBS), yet only 6% have an advanced AI security strategy. The gap between deployment ambition and security readiness is a structural demand engine.

**AI-Native Sales Tooling**

GTM teams themselves are being transformed by AI. Early-stage AI security startups are using AI agents for outbound prospecting, lead qualification, and meeting scheduling — allowing 2-3 person GTM teams to generate pipeline that previously required 8-10 SDRs. AI-assisted technical selling (pre-call research, competitive intelligence, custom demo preparation) compresses preparation time from hours to minutes. Signal-based selling tools that monitor for AI security incidents, regulatory changes, and competitor movements enable timely outreach that manual monitoring cannot match.

**Compressed Time-to-Value**

The best agentic AI security vendors achieve deployment timelines unimaginable in traditional enterprise security:
- Day 1: API-based discovery of AI estate (no agent installation, no network changes)
- Week 1: Full posture assessment with risk scoring
- Week 2-4: Policy enforcement live on critical agents

This compression enables shorter sales cycles (target: 45-60 days mid-market, 90-120 days enterprise vs. traditional 6-12 month deployments), higher customer velocity per AE, and faster expansion revenue within the same quarter as the initial land.

### Phase 2: Five GTM Models (~25%)

**Tasks:**
- [ ] Draft each GTM model with execution requirements and evidence
- [ ] Map models to company examples from thesis
- [ ] Include metrics that matter for each model

**Model A: Security Review Gate ("The Blocker")**

*Best for: AI-SPM, cross-platform governance, tool gateway*

The most powerful GTM motion. Works when the enterprise has an AI deployment initiative, the security team must sign off before production, and the security team lacks tooling to assess agent risk. The vendor provides the assessment capability and becomes a mandatory purchase.

Execution requirements: Must integrate with the agent platforms the enterprise is deploying (Bedrock, Copilot Studio, Agentforce, LangChain, CrewAI). Must produce output that satisfies security review frameworks (NIST AI RMF, internal risk frameworks). Sales cycle is tied to the enterprise's AI deployment timeline.

Evidence: Noma's 80+ integrations and AWS Security Hub Extended partnership are Security Review Gate plays. Zenity's governance capabilities for Copilot Studio and Power Platform similarly gate Microsoft-ecosystem AI deployments.

Metrics that matter: % of deals sourced from security review requests, time from security review trigger to close, attach rate to AI deployment projects.

---

**Model B: Threat-Led Growth ("The Risk Report")**

*Best for: AI-SPM, agent identity audit, tool access discovery*

An emerging GTM pattern where vendors run non-invasive discovery of the prospect's AI estate and deliver a concrete, quantified risk assessment. The output — "You have 47 agents with production database access, 12 with no identity governance, 3 with access to PII datasets that violate your stated GDPR controls" — creates urgency that no cold email or analyst report can match.

This differs from traditional "free security assessments" in that AI estate discovery is automated (not labor-intensive), the output is novel (most enterprises genuinely don't know what agents they have running), and the discovery telemetry becomes a data moat for the vendor.

Execution requirements: Must have an agentless, API-based discovery capability that produces results within hours. Risk report must be executive-ready (board-presentable, not just technical). Follow-up must connect discovery to enforcement — discovery alone is a feature, not a product.

Evidence: Noma's AI-SPM and Agentic Risk Map (ARM) implement this pattern. Pillar Security's AI asset discovery product follows a similar model at earlier stage.

Caution: This model works as a wedge but requires enforcement capability to sustain. Vendors that only deliver visibility risk the "interesting dashboard" trap (see Anti-Patterns).

---

**Model C: Platform Marketplace ("The Channel")**

*Best for: Any category; essential for scaling past $5M ARR*

AWS Marketplace and Azure Marketplace are the dominant distribution channels for enterprise security. For agentic AI security:

- AWS Marketplace enables procurement through existing enterprise agreements (EDP), reducing procurement friction from months to days. AWS's co-sell program provides AE-to-AE introductions.
- Azure Marketplace and MACC (Microsoft Azure Consumption Commitment) credits drive purchase behavior — enterprises spending down committed Azure budgets prefer marketplace purchases.
- GSI multiplier: Accenture, Deloitte, Capgemini, and Wipro are building AI security practices. They influence 40%+ of enterprise security purchases. Early GSI partnerships create a force multiplier.

Execution requirements: Marketplace listing with consumption-based billing (aligns with enterprise AI spending patterns). Co-sell registration with cloud provider sales teams. At least one Tier-1 GSI partnership by Series A. Reference architectures published in cloud provider documentation.

Evidence: Noma's AWS Security Hub Extended integration means it appears in the AWS security console — significant distribution advantage. HiddenLayer's Microsoft relationship (M12-backed) positions it for Azure Marketplace co-sell.

---

**Model D: Compliance Mandate ("The Regulator")**

*Best for: Continuous assurance, AI-BOM/supply chain, regulated AI compliance*

Regulatory mandates create the most predictable demand curves in security:

- EU AI Act (fully applicable Aug 2026): Mandates risk assessments, transparency, human oversight for high-risk AI systems. Agentic AI systems are likely "high-risk" under the Act.
- NIST AI RMF: Voluntary but increasingly referenced in US federal procurement and financial services.
- FFIEC guidance: Financial services-specific requirements for AI governance.
- ISO/IEC 42001: AI management system standard driving certification purchases.

The buyer is often GRC/Internal Audit, not CISO — a different budget with different dynamics. Compliance tools that become the system of record for AI governance attestation are extremely sticky.

Execution requirements: Deep regulatory expertise (hire ex-regulators or ex-Big Four AI audit partners). Control-to-product mapping (each regulatory article mapped to specific product capability mapped to evidence artifact). Vertical specialization initially (financial services or healthcare). Auditor partnerships with Big Four firms.

Evidence: Credo AI ($39M raised) and ValidMind ($11M seed) are executing this model in financial services. The EU AI Act enforcement timeline (Aug 2026) creates a specific demand window.

---

**Model E: Developer-Led Security ("The Bottom-Up")**

*Best for: AI-BOM tooling, testing/red-teaming, open-source security scanning*

Bottom-up adoption through the ML engineering community:

- Open-source tools: Release agent security scanning, model verification, or policy-as-code tools. Build community, convert to enterprise.
- Developer experience: Integrate into CI/CD pipelines, IDE plugins, CLI tools. Security that developers adopt voluntarily.
- Content-led growth: Technical blog posts, conference talks (RSA, Black Hat, DEF CON AI Village), research papers on agent vulnerabilities.

Evidence: Protect AI's open-source model scanner found issues in 1.2% of HuggingFace models — community credibility before enterprise sales. Promptfoo built developer traction through open-source LLM evaluation tools before OpenAI acquisition.

Execution requirements: Genuine open-source commitment (not "open-core theater"). Developer-quality documentation. Community engagement (Discord, GitHub, conferences). Clear upgrade path from free to team to enterprise.

Important caveat: Pure developer-led GTM is too slow for this market's 3-5 year acquisition window. Use it as a supplement to enterprise sales, not a replacement. The companies that exited at 10x+ had enterprise revenue, not just GitHub stars.

### Phase 3: What Doesn't Work — Anti-Patterns (~10%)

**Tasks:**
- [ ] Document seven anti-patterns with investor red flags
- [ ] Ground each in thesis evidence

**Anti-Pattern 1: "AI Security Dashboard" GTM**

Selling visibility without enforcement. Dashboards create "that's interesting" followed by inaction. The thesis's defensibility framework scores "dashboard/visibility only" as a red flag (Section 6, Dimension 1: Control Point Ownership).

*Investor red flag*: Founder describes GTM as "we show CISOs their AI risk" but has no enforcement capability.

**Anti-Pattern 2: Horizontal AI Governance Positioning**

"We secure all AI" is too broad. The buyer doesn't know who owns this internally, the budget doesn't exist, and the sales cycle extends to 12-18 months. What works instead: narrow wedge, then expand. Zenity started with low-code/no-code governance and expanded to agentic AI. Noma started with AI-SPM and expanded to full lifecycle.

*Investor red flag*: No clear initial wedge; positioning against "AI risk" broadly.

**Anti-Pattern 3: Competing with the Platform on Platform Features**

Building guardrails, content moderation, or single-model governance as a standalone product. OpenAI acquiring Promptfoo and AWS shipping Bedrock Guardrails confirm: basic model-boundary security is a platform feature. The thesis's control-point map (Section 2.1) rates these capabilities as HIGH bundling risk.

*Investor red flag*: If the answer to "what happens when AWS/OpenAI bundles this?" is "we do it better."

**Anti-Pattern 4: Enterprise Sales Without Technical Credibility**

Hiring enterprise AEs before establishing technical credibility through published research, credible demos, or open-source contributions. In AI security, ML engineers and security researchers have veto power in technical evaluations.

*Investor red flag*: GTM team has 5 AEs and 0 security researchers/engineers in customer-facing roles.

**Anti-Pattern 5: Treating AI Security as a Feature of Existing Security Products**

Adding "AI security" to an existing CNAPP, DLP, or IAM product. AI security requires fundamentally different telemetry (agent reasoning traces, tool-call graphs, prompt-response pairs) that existing architectures weren't designed to collect.

*Investor red flag*: Product is an add-on module to an existing security product without purpose-built data collection.

**Anti-Pattern 6: Over-Indexing on PLG Metrics in a CISO-Sold Market**

CISOs don't sign up for free trials. Enterprise security purchases require security reviews, SOC 2 compliance, BAAs, and data processing agreements. Developer-led *influence* works (see Model E); developer-led *purchasing* does not in this market.

*Investor red flag*: GTM plan centers on self-serve signups and product-qualified leads without enterprise sales capacity.

**Anti-Pattern 7: Sequencing Product Before GTM**

In a market with 3-5 year acquisition windows and hyperscaler bundling, there is no time to sequence product then GTM. The companies that exited at 10x+ (Prompt Security, Aim Security, Koi Security) had enterprise revenue within 12 months of founding.

*Investor red flag*: 18+ months post-founding with no enterprise customers.

### Phase 4: AI-Native GTM Team Profiles (~15%)

**Tasks:**
- [ ] Define five core personas with backgrounds and AI-native requirements
- [ ] Create hiring sequencing table by stage
- [ ] Establish critical ratios

**Core Principle**: In AI security, the GTM team IS the product's credibility. A security researcher who publishes a novel agent vulnerability finding generates more qualified pipeline than a team of SDRs. A solutions engineer who builds a custom agent security assessment creates more deal velocity than any sales deck.

---

**Persona 1: Threat Research GTM Lead (Field CTO)**

Background: 8-12 years in offensive security, red teaming, or vulnerability research. Published CVEs or original research. Conference speaking history (Black Hat, DEF CON, RSA).

AI-native requirement: Actively uses AI tools for research. Understands agentic AI from the attacker's perspective. Can articulate novel agent attack surfaces credibly.

GTM function: Produces threat research that drives media coverage, analyst attention, and inbound demand. Directly involved in enterprise technical evaluations. The credibility engine of the GTM motion.

Why this role is new: Traditional security vendors hire threat researchers for product teams. In agentic AI security, threat research IS marketing. Zenity's published AgentKit bypass research and HiddenLayer's model attack findings generate qualified demand that outperforms traditional marketing spend.

---

**Persona 2: Enterprise Account Executive (AI-Augmented)**

Background: 5-8 years in enterprise security sales (CNAPP, EDR, IAM, or DLP). Existing CISO relationships at F500. Proven $200K+ ACV track record.

AI-native requirement: Uses AI for pre-call research, competitive analysis, proposal generation, and follow-up sequencing. Comfortable selling to a committee (CISO + AI Platform Eng + Compliance) rather than a single buyer.

Key difference from traditional cyber AE: Must understand that the "security review gates AI deployment" dynamic means the buyer has internal pressure to close. Don't stretch the sales cycle — compress it.

Team sizing: 2-3 AEs at Series A, 6-8 at Series B. Each AE should carry $1.5-2.5M quota initially.

---

**Persona 3: Solutions Engineer / Security Architect**

Background: Security engineering + cloud architecture. Hands-on with AWS Bedrock, Azure AI, GCP Vertex AI, and at least one agent framework (LangChain, CrewAI, AutoGen).

AI-native requirement: Can build custom agent security assessments during the sales process. Uses AI-assisted coding tools to rapidly prototype integrations. Speaks the ML engineer's language as fluently as the security architect's.

GTM function: Runs technical evaluations and proof-of-value engagements. Builds custom risk reports (Threat-Led Growth model). Often the most trusted voice in the room.

Why critical: In this market, the SE delivers the initial threat assessment that creates deal urgency. A strong SE compresses a 6-month evaluation to 6 weeks.

---

**Persona 4: Channel/Alliance Lead**

Background: 8-12 years in channel/alliance roles at security platforms or cloud providers. Relationships at AWS, Azure, and/or GCP partner organizations.

AI-native requirement: Understands AI-specific channel dynamics — cloud provider AI teams have different partner needs than traditional security teams. Can navigate the "security partner" vs. "AI partner" distinction within cloud providers.

GTM function: Builds and manages cloud marketplace listings, co-sell registrations, GSI partnerships, and technology alliance integrations.

Timing: Hire at or just before Series A. The marketplace motion takes 6-9 months to build.

---

**Persona 5: Compliance/Evidence Lead (for Compliance Mandate model)**

Background: Ex-regulator, ex-Big Four AI audit, or ex-GRC at a regulated enterprise. Deep familiarity with EU AI Act, NIST AI RMF, FFIEC, or ISO/IEC 42001.

AI-native requirement: Can map regulatory requirements to product capabilities to automated evidence generation. Understands how AI systems are audited and what evidence auditors require.

GTM function: Leads compliance-driven sales cycles. Partners with auditors and regulators. Translates regulatory language into product requirements and back.

When to hire: Only if pursuing Compliance Mandate GTM model. Essential for vertical plays in financial services, healthcare, or defense.

---

**GTM Org Design by Stage**

| Stage | GTM Headcount | Key Hires | Primary GTM Model |
|-------|--------------|-----------|-------------------|
| Pre-seed / Seed | 2-3 | Founders + 1 threat researcher or SE | Threat-Led Growth (founder-led sales) |
| Post-Seed ($1-3M ARR) | 4-6 | + 1-2 AEs, marketing lead | Security Review Gate + TLG |
| Series A ($3-8M ARR) | 8-15 | + Channel lead, 2-3 more AEs, AI SDR tooling | Marketplace + GSI + all models |
| Series B ($8-20M ARR) | 20-35 | + VP Sales, expanded SE team, field marketing | Full-scale enterprise + international |

**Critical ratio**: Maintain at least 1:1 technical-to-commercial headcount through Series A. This market penalizes commercial over-rotation before technical credibility is established.

### Phase 5: GTM for Acquisition (~5%)

**Tasks:**
- [ ] Draft dedicated section on GTM optimized for strategic exit
- [ ] Connect to thesis exit dynamics

The thesis documents a remarkable exit dynamic: 8 acquisitions at median 8.7x on funding raised, with time-to-exit compressing (Koi Security acquired ~1 year after founding). This creates a distinct GTM consideration: **optimizing for strategic acquisition is a legitimate GTM strategy, not a failure of ambition**.

**GTM Signals That Attract Acquirers**

| Signal | Why It Matters | How to Build It |
|--------|---------------|----------------|
| Customer overlap with acquirer | De-risks integration; instant cross-sell | Target acquirer's customer base deliberately |
| Telemetry the acquirer lacks | Fills a data gap in acquirer's platform | Collect proprietary runtime data (tool graphs, permission diffs, agent behavior traces) |
| Integration with acquirer's stack | Reduces M&A integration cost | Build integrations with likely acquirers' APIs/consoles |
| Regulatory coverage acquirer needs | Provides compliance capability acquirer can't build fast enough | Specialize in regulations relevant to acquirer's enterprise customers |
| Team the acquirer wants | Acqui-hire plus product | Hire from the acquirer's competitor talent pool (offensive security, ML research) |

**Acquirer-Specific GTM Plays**

Based on the thesis's identified acquirer gaps (Section 3.2):

- **CrowdStrike**: Needs agent security for Falcon platform. GTM play: build CrowdStrike Falcon Fund relationship, integrate with Falcon API, target CrowdStrike's F500 customers.
- **Fortinet**: Needs AI security for Security Fabric. GTM play: build FortiGuard Labs relationship, publish joint threat research, target Fortinet's mid-market base.
- **Zscaler**: Needs agent security for zero-trust platform. GTM play: integrate with Zscaler Internet Access/Private Access, target Zscaler's cloud-first enterprises.

**GTM for Exit vs. GTM for Scale**

| Dimension | Optimize for Exit | Optimize for Scale |
|-----------|------------------|-------------------|
| Customer breadth | Narrow (overlap with 2-3 likely acquirers) | Broad (maximize TAM coverage) |
| Revenue target | $5-15M ARR (enough to prove PMF, not so much to be expensive) | $50M+ ARR path |
| Team size | Lean (15-25 people) | Scaling (50-100+) |
| GTM investment | Efficient (capital-light, high-leverage) | Aggressive (field sales, international) |
| Timeline | 18-36 months | 5-7 years |
| Channel strategy | Acquirer's marketplace + direct | Multi-cloud marketplace + GSI + direct |
| Metrics emphasis | ARR growth rate > absolute ARR | Absolute ARR + retention + expansion |

Most seed/A investments in agentic AI security should be underwritten with the exit path in mind. The 3-5 year bundling compression window (Section 2) means the "build to scale" path requires either a genuinely unbundleable control point or enough velocity to reach escape velocity before platforms catch up.

### Phase 6: GTM Mapped to Investment Tiers and Defensibility (~5%)

**Tasks:**
- [ ] Create per-category GTM playbook tables
- [ ] Cross-reference GTM models to thesis defensibility dimensions

**Per-Category GTM Playbooks**

**1. AI Supply Chain Security (AI-BOM, Provenance)**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary model | Compliance Mandate (EU AI Act) + Developer-Led |
| Primary buyer | GRC/Compliance + ML Platform Engineering |
| Wedge | Open-source model scanner → enterprise AI-BOM platform |
| Channel | Cloud marketplace (model registries) + PyPI/npm integration |
| Time to first deal | 6-9 months |
| Red flag | No open-source presence; selling to CISOs instead of ML engineers |

**2. Agent Identity / Delegated Authorization**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary model | Security Review Gate + Marketplace |
| Primary buyer | IAM team + CISO (joint) |
| Wedge | Agent credential audit → token brokering → full agent IAM |
| Channel | IAM vendor partnerships (Okta, CyberArk) + cloud marketplace |
| Time to first deal | 3-6 months (IAM buyers have established budgets) |
| Red flag | Positioning as generic "NHI management" instead of agent-specific identity |

**3. Tool Gateway / Policy Enforcement Point**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary model | Threat-Led Growth (ungoverned tool access discovery) + Security Review Gate |
| Primary buyer | CISO / SecOps |
| Wedge | Tool access audit → policy enforcement → full gateway |
| Channel | Direct enterprise sales (inline security is high-trust, high-touch) |
| Time to first deal | 4-8 months |
| Red flag | No inline deployment; API-only monitoring without enforcement |

**4. Regulated AI Compliance (Vertical-Specific)**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary model | Compliance Mandate (primary) + GSI partnerships |
| Primary buyer | GRC / Internal Audit / Chief Compliance Officer |
| Wedge | Single regulation in single vertical → expand |
| Channel | Big Four audit partnerships + industry conferences |
| Time to first deal | 6-12 months (compliance cycles are longer) |
| Red flag | Horizontal "AI governance" positioning; no regulatory domain expertise |

**5. Agent Incident Response**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary model | Published threat research → IR retainer → ongoing monitoring |
| Primary buyer | SOC / IR team within CISO org |
| Wedge | Free IR retainer → incident forensics → monitoring contract |
| Channel | SOAR/XDR vendor partnerships (CrowdStrike, Palo Alto, Splunk) |
| Time to first deal | Opportunistic; 9-12 months to build pipeline |
| Red flag | No published threat research; no IR capability; selling prevention without detection |

**GTM-to-Defensibility Cross-Reference**

Mapping the five GTM models to the thesis's six defensibility dimensions (Section 6):

| GTM Model | Control Point | Cross-Platform | Telemetry | Workflow Embed | Buyer Clarity | ROI Proof |
|-----------|:---:|:---:|:---:|:---:|:---:|:---:|
| A. Security Review Gate | Medium | High | Medium | High | High | High |
| B. Threat-Led Growth | Low | High | High | Medium | High | High |
| C. Platform Marketplace | Low | Medium | Low | Medium | Medium | Medium |
| D. Compliance Mandate | Medium | Medium | Low | High | High | High |
| E. Developer-Led | Low | Medium | Medium | Low | Medium | Low |

Reading the table: GTM models that score HIGH on Workflow Embedding and Buyer Clarity (Models A and D) produce the most predictable revenue. Models that score HIGH on Telemetry (Model B) create compounding advantages over time. The Platform Marketplace model (C) is a distribution accelerant, not a standalone GTM strategy.

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `thesis.md` | Modify | Append as new Section 10: GTM Strategies |
| `docs/sprints/SPRINT-002.md` | Create | Sprint document |

## Definition of Done

- [ ] Five GTM models fully articulated with execution requirements and evidence
- [ ] "What's genuinely new" section with explicit comparison to traditional cyber GTM
- [ ] Seven anti-patterns documented with investor red flags
- [ ] Five AI-native GTM personas with hiring frameworks and stage sequencing
- [ ] Dedicated "GTM for Acquisition" section with acquirer-specific plays
- [ ] Per-category GTM playbooks mapped to all five investment tiers
- [ ] GTM-to-defensibility cross-reference table
- [ ] Named company examples distributed across Noma, Zenity, HiddenLayer, Protect AI, Promptfoo, Credo AI, ValidMind, Pillar Security
- [ ] Analytical/VC-grade tone throughout — no promotional language
- [ ] Content appended to thesis.md as Section 10

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| GTM advice reads as recycled SaaS playbook | Medium | High | Ground every recommendation in agentic AI specifics; reference thesis data points |
| Market-specific claims become dated quickly | High | Medium | Date the analysis; identify assumptions that trigger re-evaluation |
| Over-prescription for early-stage founders | Medium | Medium | Frame as frameworks with stage-dependent variation, not rigid rules |
| Company examples become stale (acquisitions, pivots) | Medium | Low | Focus on the GTM pattern, not the company; note that examples are point-in-time |
| Bundling accelerates faster than projected | Medium | High | Flag that GTM for exit should be the default posture unless control point is genuinely unbundleable |

## Security Considerations

- Research/writing sprint only — no code or infrastructure changes
- All competitive intelligence sourced from public information
- No proprietary company data included without attribution
- Company references limited to publicly available information (press releases, published research, public filings)

## Dependencies

- Sprint 001 (completed) — tracker tooling available
- thesis.md — primary source document for market data, control-point map, defensibility dimensions, and investment tiers

## References

- thesis.md Sections 1 (market sizing), 2 (bundling), 3 (competitive landscape), 4 (investment tiers), 5 (deal landscape), 6 (defensibility framework), 8 (investment thesis), 9 (existing GTM section)
- Noma Security: noma.security, PRNewswire, SecurityWeek
- Zenity: zenity.io, Gartner Cool Vendor report
- HiddenLayer: hiddenlayer.com, published ML attack research
- Protect AI: protectai.com, HuggingFace model scanning data
- Credo AI / ValidMind: company websites, funding announcements
- EU AI Act: Official Journal of the European Union
- NIST AI RMF: nist.gov
