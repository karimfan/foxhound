# Sprint 002: GTM Strategies for Agentic AI Security in the Enterprise

## Overview

The agentic AI security market has validated product-market fit through $2B+ in M&A, but most startups in this category have not yet validated *go-to-market fit*. The gap between building a defensible product and building a repeatable GTM engine is where most early-stage AI security companies will succeed or die in 2026-2028.

This sprint expands the thesis's Section 9 into a comprehensive GTM framework for the agentic AI security category. It answers three questions a VC should ask of every founder: (1) How will you get your first 20 enterprise customers? (2) What makes your GTM structurally different from the last generation of cybersecurity vendors? (3) Who on your team can actually execute this?

The core argument: **GTM for agentic AI security is neither pure enterprise sales nor pure PLG — it is a new hybrid model we call "Threat-Led Growth" (TLG), where demonstrated risk evidence creates urgency that traditional demand generation cannot.** The companies that win will combine technical credibility with the ability to manufacture urgency through proof-of-risk, not proof-of-concept.

## Use Cases

1. **VC due diligence on GTM capability**: Evaluate whether a founding team can build a repeatable sales motion in this specific market
2. **Founder GTM playbook**: Provide actionable frameworks for seed/A-stage founders building their first GTM teams
3. **GTM team design**: Define the profiles, personas, and org structures for AI-native GTM teams
4. **Anti-pattern identification**: Catalog what doesn't work so investors can spot red flags early
5. **GTM-to-exit mapping**: Connect GTM strategy to the "build to be acquired" dynamic identified in the thesis

## Architecture

This is a research/analysis sprint. The "architecture" is the analytical framework:

```
GTM Framework for Agentic AI Security
├── 1. What's Genuinely New
│   ├── Buyer topology changes
│   ├── Threat-Led Growth (TLG) model
│   ├── AI-native sales tooling
│   └── Compressed time-to-value
├── 2. What Works
│   ├── GTM Model A: Security Review Gate
│   ├── GTM Model B: Platform Marketplace
│   ├── GTM Model C: Compliance Mandate
│   ├── GTM Model D: Incident Response Wedge
│   └── GTM Model E: Developer-Led Security
├── 3. What Doesn't Work
│   ├── Anti-pattern catalog
│   └── Red flags for investors
├── 4. AI-Native GTM Team Profiles
│   ├── Persona definitions
│   ├── Hiring frameworks
│   └── Org design by stage
└── 5. GTM Mapped to Investment Tiers
    ├── Per-category GTM playbooks
    └── GTM defensibility scoring
```

## Implementation Plan

### Phase 1: What's Genuinely New in Enterprise GTM for Agentic AI (~25%)

**The Buyer Topology Has Changed**

The traditional cybersecurity GTM model assumes a single buyer (CISO) with an established budget category. Agentic AI security breaks this in four ways:

| Dimension | Traditional Cyber GTM | Agentic AI Security GTM |
|-----------|----------------------|------------------------|
| Primary buyer | CISO (sole authority) | CISO + AI Platform Eng + Compliance (committee buy) |
| Budget source | Established security line item | Contested: security budget vs. AI/innovation budget vs. compliance budget |
| Technical evaluator | SecOps/SOC team | SecOps + ML engineers + platform team (cross-functional) |
| Urgency driver | Active breach / compliance deadline | Agent deployment blockers + regulatory uncertainty |
| Sales cycle trigger | Vendor outreach / analyst report | Security review gates AI rollout (organic pull) |
| Champion profile | Security architect | "AI Security Lead" (new role, often doesn't exist yet) |

**Key insight**: The fact that security review gates AI deployment is the single most important GTM lever in this market. When CISO organizations can block agent rollout until security controls are in place, the AI security vendor becomes a *mandatory* purchase, not a discretionary one. This creates a fundamentally different demand curve than traditional security products.

**Threat-Led Growth (TLG): The New GTM Primitive**

TLG is the defining GTM innovation for agentic AI security. It works as follows:

1. **Discovery scan**: Vendor runs a non-invasive discovery of the prospect's AI estate — shadow AI models, unregistered agents, MCP server connections, tool access patterns, delegated authority chains.
2. **Risk report**: Output is a concrete, quantified risk assessment: "You have 47 agents with production database access, 12 with no identity governance, 3 with access to PII datasets that violate your stated GDPR controls."
3. **Urgency manufacture**: The risk report creates internal urgency that no cold email or analyst report can match. The CISO now has a *problem they didn't know they had* with *evidence they can show the board*.
4. **Land**: Initial deal closes on the discovery/posture layer — the vendor is now the system of record for the AI estate.
5. **Expand**: Upsell to enforcement, compliance, incident response — each expansion driven by new risk evidence from the installed posture layer.

**Why TLG is structurally new (not just a rebrand of "free assessment")**:
- Traditional security assessments are labor-intensive and commoditized. AI estate discovery is *automated* and *novel* — most enterprises genuinely don't know what agents they have running.
- The output is not "you have vulnerabilities" (generic) but "Agent X accessed Customer Y's data via Tool Z without authorization at timestamp T" (specific, actionable, scary).
- The discovery telemetry itself becomes a data moat — the vendor accumulates a unique map of the enterprise's AI estate that competitors cannot replicate without equivalent access.

**Noma's execution of TLG**: Noma's AI-SPM (Security Posture Management) + Agentic Risk Map (ARM) is the clearest implementation of TLG. The 1,300% ARR growth is partially explained by this model — once you show a CISO their unsecured AI estate, the deal dynamics shift from "should we buy this?" to "how fast can we deploy?"

**AI-Native Sales Tooling**

The GTM teams themselves are being transformed by AI:

- **AI SDRs and prospecting**: Companies like 11x, Artisan, and Relevance AI are building AI agents that handle outbound prospecting, lead qualification, and meeting scheduling. Early-stage AI security startups are using these tools to punch above their weight on pipeline generation with 2-3 person GTM teams.
- **AI-assisted technical selling**: Pre-call research, competitive intelligence, and custom demo preparation are increasingly AI-augmented. A technical AE can prepare for an enterprise security review in 30 minutes instead of 3 hours.
- **AI-powered customer success**: Automated health scoring, usage pattern analysis, and proactive outreach based on product telemetry. Critical for the land-and-expand model.
- **Signal-based selling**: Tools that monitor for AI security incidents, regulatory changes, and competitor vulnerabilities to trigger timely outreach. When EU AI Act enforcement actions begin (Aug 2026), the GTM teams that can respond within hours will capture outsized pipeline.

**Compressed Time-to-Value**

Traditional enterprise security products require 6-12 month deployments. The best agentic AI security vendors are achieving:
- **Day 1**: API-based discovery of AI estate (no agent installation, no network changes)
- **Week 1**: Full posture assessment with risk scoring
- **Week 2-4**: Policy enforcement live on critical agents
- **Month 2-3**: Full deployment across agent fleet

This compression matters for GTM because it enables:
- Faster proof-of-value → faster close → shorter sales cycles (target: 45-60 days for mid-market, 90-120 days for enterprise)
- Higher customer velocity per AE (8-12 enterprise deals/year vs. traditional 4-6)
- Faster expansion revenue (land with discovery, expand to enforcement within same quarter)

### Phase 2: What Works — Five GTM Models (~25%)

**GTM Model A: Security Review Gate ("The Blocker")**

*Best for: AI-SPM, Cross-platform governance, Tool gateway*

The most powerful GTM motion in this market. Works when:
1. Enterprise has an AI deployment initiative (e.g., deploying Salesforce Agentforce, building internal agents on AWS Bedrock)
2. Security team is required to sign off before production deployment
3. Security team has no tooling to assess agent risk
4. Vendor provides the assessment capability → becomes mandatory purchase

**Why it works**: Creates *inbound demand* driven by deployment timelines, not outbound prospecting. The AI platform team becomes an internal champion because they need security sign-off to ship.

**Execution requirements**:
- Must integrate with the agent platforms the enterprise is actually deploying (Bedrock, Copilot Studio, Agentforce, LangChain, CrewAI)
- Must produce output that satisfies security review frameworks (NIST AI RMF, internal risk frameworks)
- Sales cycle is tied to the enterprise's AI deployment timeline, not the vendor's fiscal quarter

**Evidence**: Noma's AWS Security Hub Extended partnership and Databricks integration are Security Review Gate plays. The 80+ integrations create the breadth needed to block across the entire AI estate.

**Metrics that matter**: % of deals sourced from security review requests, time from security review trigger to close, attach rate to AI deployment projects.

---

**GTM Model B: Platform Marketplace ("The Channel")**

*Best for: Any category; essential for scale*

AWS Marketplace and Azure Marketplace are the dominant distribution channels for enterprise security. For agentic AI security specifically:

- **AWS Marketplace**: Enables procurement through existing AWS enterprise agreements (EDP). Reduces procurement friction from months to days. AWS's co-sell program (ISV Accelerate) provides AE-to-AE introductions.
- **Azure Marketplace**: Microsoft's MACC (Microsoft Azure Consumption Commitment) credits drive purchase behavior — enterprises spending down committed Azure budgets prefer marketplace purchases.
- **Salesforce AppExchange**: Relevant for Agentforce security specifically. Salesforce's ecosystem is deploying agents aggressively; security tooling on AppExchange has a captive audience.

**Why it works**: Eliminates procurement as a bottleneck. When an enterprise CISO can buy through an existing cloud commitment rather than a new vendor approval, deal velocity increases 3-5x.

**The GSI multiplier**: Global Systems Integrators (Accenture, Deloitte, Capgemini, Wipro) are building AI security practices. They influence 40%+ of enterprise security purchases. Early GSI partnerships create a force multiplier — the GSI recommends your tool as part of their AI deployment methodology.

**Execution requirements**:
- Marketplace listing with metered billing (consumption-based aligns with enterprise AI spending patterns)
- Co-sell registration with cloud provider sales teams
- GSI partnership with at least one Tier-1 integrator by Series A
- Reference architectures published in cloud provider documentation

**Evidence**: Noma's AWS partnership validates this model. The AWS Security Hub Extended integration means Noma appears in the AWS security console — the ultimate distribution advantage.

---

**GTM Model C: Compliance Mandate ("The Regulator")**

*Best for: Continuous assurance, AI-BOM/supply chain, Regulated AI compliance*

Regulatory mandates create the most predictable demand curves in security. For agentic AI:

- **EU AI Act** (fully applicable Aug 2026): Mandates risk assessments, transparency, human oversight for high-risk AI systems. Agentic AI systems are likely "high-risk" under the Act.
- **NIST AI RMF**: Voluntary but increasingly referenced in US federal procurement and financial services regulations.
- **FFIEC guidance**: Financial services-specific requirements for AI governance.
- **ISO/IEC 42001**: AI management system standard — certification will drive tooling purchases.

**Why it works**: Compliance purchases are non-discretionary. The buyer is often GRC/Internal Audit, not CISO — a different budget with different dynamics. Compliance tools that become the "system of record" for AI governance attestation are extremely sticky.

**The timing play**: EU AI Act enforcement begins Aug 2026. Companies selling compliance tooling should be in proof-of-concept with regulated enterprises by Q2 2026 and closing deals in Q3-Q4 2026. The enforcement window creates a 12-18 month demand spike.

**Execution requirements**:
- Deep regulatory expertise (hire ex-regulators, ex-Big Four AI audit partners)
- Control-to-product mapping (each regulatory article → specific product capability → evidence artifact)
- Vertical specialization initially (financial services or healthcare, not horizontal)
- Auditor partnerships (Big Four firms that will recommend your tool during AI audits)

**Anti-pattern**: Don't build horizontal "AI governance" — too vague, unclear buyer, long sales cycles. Build specific compliance evidence generation for specific regulations in specific verticals.

---

**GTM Model D: Incident Response Wedge ("The Fire Truck")**

*Best for: Agent IR, SOC integration, Behavioral forensics*

Works when something goes wrong — and in 2026, things are starting to go wrong:
- Agent prompt injection attacks are increasing
- Data exfiltration via agent tool access is a documented threat
- Agent-to-agent compromise is theoretical but credible

**Why it works**: Incident response creates *emergency demand* with compressed procurement cycles. A CISO who just had an agent security incident will buy within days, not quarters.

**The playbook**:
1. Publish research on agent security incidents (threat intelligence as content marketing)
2. Offer free incident response retainers (similar to CrowdStrike's early model)
3. Convert IR engagements to ongoing monitoring/prevention contracts
4. Use incident forensics data to improve product (data flywheel)

**Execution requirements**:
- World-class threat research team (publish credible, novel findings)
- 24/7 IR capability (even if initially through partners)
- Integration with existing SIEM/SOAR/XDR (Splunk, Sentinel, CrowdStrike)
- Executive-level IR communication capability (board-ready incident reports)

**Timing**: This model becomes more viable as agent deployments scale and real incidents accumulate. In 2026, it's a wedge play; by 2028, it could be a primary GTM motion.

---

**GTM Model E: Developer-Led Security ("The Bottom-Up")**

*Best for: AI-BOM tooling, Testing/red-teaming, Open-source security scanning*

The MLOps/AI engineering community is the fastest-growing technical audience in enterprise software. Developer-led GTM in AI security means:

- **Open-source tools**: Release agent security scanning, model verification, or policy-as-code tools as open source. Build community → convert to enterprise.
- **Developer experience**: Integrate into CI/CD pipelines, IDE plugins, CLI tools. Security that developers adopt voluntarily, not security imposed by the CISO.
- **Content-led growth**: Technical blog posts, conference talks (RSA, Black Hat, DEF CON AI Village, KubeCon), research papers on agent vulnerabilities.

**Why it works**: Bottom-up adoption creates a technical champion inside the enterprise before the vendor ever talks to the CISO. When the CISO is evaluating AI security tools, the ML engineers already have a preference.

**Evidence**: Protect AI's open-source model scanner found issues in 1.2% of HuggingFace models — this kind of community tool creates credibility and adoption before enterprise sales. Promptfoo (before OpenAI acquisition) built significant developer traction through open-source LLM evaluation tools.

**Execution requirements**:
- Genuine open-source commitment (not "open-core theater")
- Developer-quality documentation and onboarding
- Community engagement (Discord, GitHub, conference circuit)
- Clear upgrade path from free → team → enterprise

**Caution**: Pure developer-led GTM is too slow for this market's acquisition timeline. Use it as a *supplement* to enterprise sales, not a replacement. The companies that got acquired at 10x+ had enterprise revenue, not just GitHub stars.

### Phase 3: What Doesn't Work — Anti-Patterns and Red Flags (~15%)

**Anti-Pattern 1: "AI Security Dashboard" GTM**

Selling visibility without enforcement. Dashboards don't create urgency — they create "that's interesting" followed by inaction. The thesis's defensibility framework scores "dashboard/visibility only" as a red flag for a reason.

*Red flag for investors*: Founder describes GTM as "we show CISOs their AI risk" but has no enforcement capability. Discovery without enforcement is a feature, not a product.

**Anti-Pattern 2: Horizontal AI Governance Positioning**

"We secure all AI" is too broad. The buyer doesn't know who owns this internally, the budget doesn't exist, and the sales cycle is 12-18 months.

*What works instead*: Narrow wedge → expand. "We secure your Salesforce Agentforce deployment" → "We secure all your agents" → "We govern your AI estate." Noma started with AI-SPM and expanded to full lifecycle. Zenity started with low-code/no-code and expanded to agentic AI.

**Anti-Pattern 3: Competing with the Platform on Platform Features**

Building guardrails, content moderation, or single-model governance as a standalone product. OpenAI acquiring Promptfoo and AWS shipping Bedrock Guardrails confirms: basic model-boundary security is a platform feature, not a venture-scale business.

*Red flag for investors*: If the answer to "what happens when AWS/OpenAI bundles this?" is "we do it better," the company will be compressed.

**Anti-Pattern 4: Enterprise Sales Without Technical Credibility**

Hiring enterprise sales reps before establishing technical credibility. In AI security, the technical evaluators (ML engineers, security researchers) have veto power. A polished sales deck without a credible demo, published research, or open-source contribution will fail technical evaluation.

*Red flag for investors*: GTM team has 5 AEs and 0 security researchers/engineers in customer-facing roles.

**Anti-Pattern 5: Treating AI Security as a Feature of Existing Security Products**

Some founders try to add "AI security" to an existing CNAPP, DLP, or IAM product. The problem: AI security requires fundamentally different telemetry (agent reasoning traces, tool-call graphs, prompt-response pairs) that existing security architectures weren't designed to collect.

*What works instead*: Purpose-built products with native integration to existing security stacks. Noma integrates with AWS Security Hub (existing tool) but collects fundamentally different telemetry.

**Anti-Pattern 6: Over-Indexing on PLG Metrics in a CISO-Sold Market**

CISOs don't sign up for free trials. Enterprise security purchases require security reviews, SOC 2 compliance, BAAs, data processing agreements. Pure PLG motions that work in developer tools (Datadog, Snyk) don't translate to AI security without significant adaptation.

*Nuance*: Developer-led *influence* works (see GTM Model E). Developer-led *purchasing* doesn't — the economic buyer is still the CISO.

**Anti-Pattern 7: "We'll Figure Out GTM After Product-Market Fit"**

In a market with 3-5 year acquisition windows and hyperscaler bundling, you don't have time to sequence product → GTM. The companies that exited at 10x+ had GTM running from day one. Protect AI, Prompt Security, and Aim Security all had enterprise revenue within 12 months of founding.

### Phase 4: AI-Native GTM Team Profiles and Personas (~20%)

**The AI-Native GTM Team: What's Different**

Traditional cybersecurity GTM teams are organized around: marketing → SDR → AE → SE → CSM. AI-native GTM teams reorganize around *technical credibility as the primary GTM asset* and *AI tooling as a force multiplier*.

**Core Principle**: In AI security, the GTM team IS the product's credibility. A security researcher who publishes a novel agent vulnerability finding generates more qualified pipeline than 10 SDRs. A solutions engineer who builds a custom agent security assessment for a prospect creates more deal velocity than any sales deck.

---

**Persona 1: The Threat Research GTM Lead**

*Role*: Head of Threat Research / Field CTO
*Profile*:
- Background: 8-12 years in offensive security, red teaming, or vulnerability research
- Must-haves: Published CVEs or original research, conference speaking history (Black Hat, DEF CON, RSA), deep understanding of LLM/agent attack surfaces
- AI-native twist: Actively uses AI tools for research (automated fuzzing, AI-assisted code review, agent behavior analysis). Understands agentic AI from the attacker's perspective.
- GTM function: Produces threat research that drives media coverage, analyst attention, and inbound demand. Directly involved in enterprise technical evaluations. The "credibility engine" of the GTM motion.

*Why this role is new*: Traditional security vendors hire threat researchers for product teams. In AI security GTM, threat research IS marketing. A single well-publicized agent vulnerability finding (like Zenity's AgentKit bypass or HiddenLayer's model attacks) generates more qualified demand than a $2M marketing budget.

*Hiring signal for investors*: If the founding team includes someone with this profile, GTM credibility is built in. If not, this should be the first non-founding hire.

---

**Persona 2: The AI-Augmented Enterprise AE**

*Role*: Enterprise Account Executive
*Profile*:
- Background: 5-8 years in enterprise security sales (CNAPP, EDR, IAM, or DLP)
- Must-haves: Existing CISO relationships at F500, proven $200K+ ACV track record, understanding of security procurement processes
- AI-native twist: Uses AI for pre-call research, competitive analysis, proposal generation, and follow-up sequencing. Can articulate AI security concepts without falling into "AI hype" language. Comfortable in technical conversations with ML engineers.
- GTM function: Manages enterprise sales cycles. Works with threat research lead for technical credibility. Leverages marketplace channels for procurement acceleration.

*Team sizing*: 2-3 AEs at Series A, 6-8 at Series B. Each AE should carry $1.5-2.5M quota initially (reflecting high ACV but early-market dynamics).

*Key difference from traditional cyber AE*: Must be comfortable selling to a committee (CISO + AI Platform Eng + Compliance) rather than a single buyer. Must understand that the "security review gates AI deployment" dynamic means the buyer has internal pressure to close quickly — don't stretch the sales cycle.

---

**Persona 3: The Solutions Engineer / Security Architect**

*Role*: Solutions Engineering Lead
*Profile*:
- Background: Security engineering + cloud architecture. Ideally has built or operated AI/ML systems.
- Must-haves: Hands-on with AWS Bedrock, Azure AI, GCP Vertex AI, and at least one agent framework (LangChain, CrewAI, AutoGen). Can write code, build integrations, and conduct live technical assessments.
- AI-native twist: Can build custom agent security assessments during the sales process. Uses AI-assisted coding tools to rapidly prototype integrations. Understands the technical architecture of the prospect's agent deployment, not just the security layer.
- GTM function: Runs technical evaluations and proof-of-value engagements. Builds custom risk reports (TLG model). Often the most trusted voice in the room because they can speak the ML engineer's language.

*Why this role is critical*: In AI security, the SE does more than demo the product — they deliver the initial threat assessment that creates deal urgency. A strong SE can compress a 6-month evaluation to 6 weeks.

---

**Persona 4: The AI-Native Marketing Lead**

*Role*: Head of Marketing / Head of Growth
*Profile*:
- Background: Content marketing or developer marketing in cybersecurity or DevOps
- Must-haves: Ability to translate technical threat research into compelling narratives for different audiences (CISO, board, ML engineer, compliance officer). Experience with analyst relations (Gartner, Forrester).
- AI-native twist: Uses AI extensively for content production, SEO optimization, competitive monitoring, and signal-based campaign targeting. Runs a "content machine" that produces 10x the output of a traditional marketing team. Manages AI SDR tooling for outbound.
- GTM function: Converts threat research into demand. Manages analyst relations (getting into Gartner MQ/Market Guide is a major GTM accelerator in security). Runs event strategy (RSA Conference is the single highest-ROI event for enterprise security).

*AI content production model*: AI-native marketing teams use LLMs to draft content, but the *credibility* comes from original threat research and proprietary data. The formula: original research (human-generated) → AI-amplified distribution → measured conversion.

---

**Persona 5: The Channel/Alliance Lead**

*Role*: VP of Alliances / Channel Lead
*Profile*:
- Background: 8-12 years in channel/alliance roles at security platforms or cloud providers
- Must-haves: Relationships at AWS, Azure, and/or GCP partner organizations. Experience building GSI partnerships. Understanding of marketplace listing, co-sell, and ISV programs.
- AI-native twist: Understands the AI-specific channel dynamics — cloud provider AI teams have different partner needs than traditional security teams. Can navigate the "security partner" vs. "AI partner" distinction within cloud providers.
- GTM function: Builds and manages cloud marketplace listings, co-sell registrations, GSI partnerships, and technology alliance integrations.

*Timing*: This hire should happen at or just before Series A. The cloud marketplace motion takes 6-9 months to build and is essential for scaling beyond $5M ARR.

---

**GTM Org Design by Stage**

| Stage | Headcount (GTM) | Key Hires | GTM Model Focus |
|-------|-----------------|-----------|-----------------|
| Pre-seed / Seed | 2-3 | Founders + 1 threat researcher OR SE | TLG (founder-led sales) |
| Post-Seed | 4-6 | + 1-2 AEs, marketing lead | Security Review Gate + TLG |
| Series A | 8-15 | + Channel lead, 2-3 more AEs, SDR team or AI SDR tooling | Marketplace + GSI + all models |
| Series B | 20-35 | + VP Sales, expanded SE team, field marketing | Full-scale enterprise + international |

**Critical ratio**: Maintain at least 1:1 technical (SE + researcher) to commercial (AE + SDR) headcount through Series A. The market penalizes commercial over-rotation before technical credibility is established.

### Phase 5: GTM Mapped to Investment Tiers (~15%)

Mapping specific GTM playbooks to the thesis's five priority investment categories:

**1. AI Supply Chain Security (AI-BOM, Provenance)**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary GTM model | Compliance Mandate (EU AI Act) + Developer-Led |
| Primary buyer | GRC/Compliance + ML Platform Engineering |
| Wedge motion | Open-source model scanner → enterprise AI-BOM platform |
| Key channel | Cloud marketplace (model registries) + PyPI/npm integration |
| Time to first enterprise deal | 6-9 months |
| GTM red flag | No open-source presence; selling to CISOs instead of ML engineers |

**2. Agent Identity / Delegated Authorization**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary GTM model | Security Review Gate + Platform Marketplace |
| Primary buyer | IAM team + CISO (joint) |
| Wedge motion | Agent credential audit → token brokering → full agent IAM |
| Key channel | IAM vendor partnerships (Okta, CyberArk) + cloud marketplace |
| Time to first enterprise deal | 3-6 months (IAM buyers have established budgets) |
| GTM red flag | Positioning as "NHI management" (crowded) instead of agent-specific identity |

**3. Tool Gateway / Policy Enforcement Point**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary GTM model | TLG (discovery of ungoverned tool access) + Security Review Gate |
| Primary buyer | CISO / SecOps |
| Wedge motion | Tool access audit → policy enforcement → full gateway |
| Key channel | Direct enterprise sales (inline security is high-trust) |
| Time to first enterprise deal | 4-8 months |
| GTM red flag | No inline deployment story; API-only monitoring without enforcement |

**4. Regulated AI Compliance (Vertical-Specific)**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary GTM model | Compliance Mandate (primary) + GSI partnerships |
| Primary buyer | GRC / Internal Audit / Chief Compliance Officer |
| Wedge motion | Single regulation (EU AI Act OR FFIEC) in single vertical → expand |
| Key channel | Big Four audit partnerships + industry conferences (not security conferences) |
| Time to first enterprise deal | 6-12 months (compliance sales cycles are longer) |
| GTM red flag | Horizontal "AI governance" positioning; no regulatory domain expertise on team |

**5. Agent Incident Response**

| GTM Dimension | Recommendation |
|---------------|---------------|
| Primary GTM model | IR Wedge (primary) + SOAR/XDR integration |
| Primary buyer | SOC / IR team within CISO org |
| Wedge motion | Free IR retainer → incident forensics → ongoing monitoring |
| Key channel | SOAR/XDR vendor partnerships (CrowdStrike, Palo Alto, Splunk) |
| Time to first enterprise deal | Opportunistic — depends on incident volume. Expect 9-12 months to build pipeline. |
| GTM red flag | No published threat research; no IR capability; selling prevention without detection |

## Files Summary

| File | Action | Purpose |
|------|--------|---------|
| `thesis.md` | Modify | Expand Section 9 with GTM framework (or append as Section 10) |
| `docs/sprints/SPRINT-002.md` | Create | Sprint document |

## Definition of Done

- [ ] Comprehensive GTM framework covering all five models
- [ ] Clear articulation of what's genuinely new (TLG, buyer topology, AI-native tooling)
- [ ] Anti-pattern catalog with investor red flags
- [ ] Five AI-native GTM personas with hiring frameworks
- [ ] GTM playbooks mapped to all five investment tier categories
- [ ] Framework is actionable for both VCs (evaluation) and founders (execution)
- [ ] Content reviewed for analytical rigor (VC perspective, not vendor marketing)

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| GTM advice becomes generic/recycled SaaS playbook | Medium | High | Ground every recommendation in agentic AI specifics; reference thesis data |
| Over-prescription for early-stage (too rigid) | Medium | Medium | Frame as frameworks, not rules; acknowledge stage-dependent variation |
| Market moves faster than analysis | High | Medium | Date the analysis; identify assumptions that should trigger re-evaluation |
| Founder profiles too narrow | Low | Medium | Include "non-obvious" profiles alongside canonical ones |

## Security Considerations

- No code or infrastructure changes — research/writing sprint only
- Ensure competitive intelligence is sourced from public information only
- No proprietary company data included without attribution

## Dependencies

- Sprint 001 (completed) — tracker tooling available
- thesis.md — primary source document

## Open Questions

1. Should the TLG (Threat-Led Growth) framework be developed into a standalone concept with its own positioning, or kept as one model among five?
2. How should we weight international GTM (EU regulatory-driven demand) vs. US-first strategies?
3. Should we include specific compensation benchmarks for the GTM personas, or keep it at the role/profile level?
4. Is there a "GTM for acquisition" playbook that's distinct from "GTM for scale" — and should we develop it explicitly?
