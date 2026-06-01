# provider-api-ai-poc-index

**Healthcare Provider AI Accelerators | Optum Real API + Generative AI POC Collection**

---

A curated index of proof-of-concept applications demonstrating how healthcare providers can harness **Optum Real APIs** combined with **generative AI** (Claude, OpenAI) to eliminate administrative friction, accelerate revenue cycle operations, and surface real-time clinical intelligence at the point of care.

Each POC in this collection is a working, deployable application built on a **HIPAA-conscious, LLM-agnostic architecture** using Next.js, deployed on Vercel. All demo environments use synthetic mock data only. No real patient data is transmitted in any application.

> **This repository contains no application code.** It serves as the master index, navigation hub, and architecture reference for the full POC collection.

---

## Why This Collection Exists

Healthcare providers operate in one of the most administratively burdensome environments in any industry. Prior authorization requests consume an average of 13 hours per physician per week. Claim denial rates climbed to 11.8% in 2024. A 45-minute MRI can require 7 business days of paperwork. These are not edge cases. They are the baseline.

Optum Real APIs provide a direct, programmatic connection to payer intelligence: eligibility data, coverage rules, prior authorization logic, claim validation, and cost transparency. Generative AI provides the reasoning layer that translates raw API responses into actionable clinical and administrative guidance.

This collection demonstrates what becomes possible when you connect those two layers.

---

## POC Index

> While the majority of applications in this collection are built from the **provider** perspective — the hospitals, health systems, and physician groups submitting claims and authorization requests — the first entry approaches the same transaction from the opposite side. **Payer Auth Intelligence** demonstrates the **payer-side** decisioning architecture that receives, evaluates, and adjudicates those submissions. Together with the provider-side POCs that follow, it illustrates the full lifecycle of a prior authorization transaction across the payer-provider boundary.

---

### 1. Payer Auth Intelligence

**Agentic prior authorization decisioning from the payer side of the transaction.**

| | |
|---|---|
| **GitHub** | [paullopez-ai/payer-auth-intelligence](https://github.com/paullopez-ai/payer-auth-intelligence) |
| **Live Demo** | Requires AWS |
| **Primary API** | Optum Real Prior Authorization (payer-side adapter) |
| **AI Layer** | Claude Sonnet via Anthropic SDK / Amazon Bedrock (LLM-agnostic) |
| **Cloud Platform** | AWS (production) |
| **IaC** | AWS CDK (TypeScript) |

**Purpose:**
Payer Auth Intelligence simulates the health-plan system that receives a prior authorization submission from a provider, evaluates it against benefit policy and medical-necessity criteria using a multi-agent LangGraph.js StateGraph, and routes it to an auto-approve, auto-deny, pending-info, or human-review outcome in near real time. The decisioning pipeline runs as an in-process graph with typed nodes for intake validation, benefit coverage determination, LLM-powered medical-necessity scoring, and deterministic threshold-based routing. When confidence is below threshold or the case is flagged as expedited, the graph pauses at a human-review interrupt and resumes with full state preserved after a reviewer acts.

This is the **payer-side complement** to [Prior Auth Radar](https://github.com/paullopez-ai/prior-auth-radar), which handles the same PA transaction from the provider perspective. The integration seam between the two is the `ClaimsEngineAdapter` on the payer side and the eligibility/status endpoint on the provider side — in a real enterprise, this would be the same HL7 FHIR or adjudication API.

**Healthcare Value:**
Leading health plans target 80%+ touchless PA automation. This POC demonstrates the architecture that makes that possible, including the mandatory human-oversight boundary that keeps it responsible. Every decision traces to a versioned policy clause. The LLM never auto-denies on uncertainty — unparseable responses, unavailable policies, and unverifiable eligibility all escalate to human review by design. For payers processing millions of PA requests annually, even incremental automation at the intake and benefit-check layers frees clinical reviewers to focus on genuinely complex cases.

**Three-Track Demo Strategy:**

**Track 1 — Local Mock (recommended for live demos):**
Zero API keys, zero AWS. Full graph execution with MemorySaver; all 8 built-in scenarios run with mock node functions. Instant and free.

**Track 2 — Local Sandbox with Live LLM:**
Real Claude reasoning on mock submissions. Still local, still MemorySaver, approximately $0.01 per run.

**Track 3 — AWS Deployment (production architecture validation):**
Next.js on Amplify, Lambda per agent node, DynamoDB audit trail, Bedrock for the LLM. Deploy once, record, tear down via `cdk destroy`.

**Production Environment Variables:**
`LLM_PROVIDER`, `ANTHROPIC_API_KEY`, `AWS_REGION`, `BEDROCK_MODEL_ID`, `USE_DYNAMODB_CHECKPOINTER`, `DYNAMODB_DECISIONS_TABLE`, `DYNAMODB_SUBMISSIONS_TABLE`, `S3_POLICY_BUCKET`

**Enterprise Integration Points (Adapter Stubs):**

| Adapter | Represents | Enterprise Systems |
|---|---|---|
| `ClaimsEngineAdapter` | Adjudication platform / claims core | Trizetto QNXT, proprietary core, HL7 FHIR R4 Coverage |
| `PolicyRepositoryAdapter` | Medical policy & benefit config | MCG, InterQual, proprietary policy management |
| `MemberEligibilityAdapter` | Real-time eligibility | X12 270/271, FHIR R4 Coverage, Optum Real |
| `NotificationAdapter` | Provider notification | FHIR R4 Task, portal webhook, Direct Secure Messaging, fax API |

**Architecture:**

```
 LangGraph.js StateGraph (in-process decisioning pipeline)

                                          ┌──────────────────────────────┐
                                          │ expedited & confidence < 0.95 │
                                          v                               │
  [START] → intake → benefit_check → med_necessity ──(routing)──► routing ──► [END]
                                          │                               │
                                          └───────────► human_review ◄────┘
                                               (interruptBefore)    ^ ESCALATED_TO_HUMAN
                                                                    │
                                           POST /api/decisions/[id]/resume
                                           injects reviewer input, resumes

 Nodes:
   intake         — validates completeness, enriches with member eligibility
   benefit_check  — coverage determination from policy repository (no LLM)
   med_necessity  — LLM reasoning node; scores confidence against criteria
   routing        — deterministic threshold rules → final PADecision (no LLM)
   human_review   — resumes after reviewer approves/denies/pends escalated case

 AWS Production (Track 3):
┌─────────────────────────────────────────────────────────────┐
│               Payer Auth Intelligence                        │
│                  (AWS Amplify + Lambda)                      │
└───────────────────┬─────────────────────┬───────────────────┘
                    │                     │
         ┌──────────▼──────────┐ ┌────────▼──────────────────┐
         │  LangGraph.js       │ │  Amazon Bedrock            │
         │  StateGraph         │ │  Claude Sonnet (inference)  │
         │  (Lambda per node)  │ │  S3 (policy documents)     │
         └──────────┬──────────┘ └────────┬──────────────────┘
                    │                     │
         ┌──────────▼─────────────────────▼───────────┐
         │          DynamoDB Audit Trail                │
         │   Checkpoints + Decisions + Submissions     │
         │   Full state preserved across interrupts    │
         └─────────────────────────────────────────────┘

Infrastructure provisioned via AWS CDK (infrastructure/)
```

---

### 2. Prior Auth Radar

**The fastest path from clinical order to authorization decision.**

| | |
|---|---|
| **GitHub** | [paullopez-ai/prior-auth-radar](https://github.com/paullopez-ai/prior-auth-radar) |
| **Live Demo** | [prior-auth-radar.vercel.app](https://prior-auth-radar.vercel.app) |
| **Primary API** | Optum Real Prior Authorization |
| **AI Layer** | Claude via Amazon Bedrock |
| **Cloud Platform** | AWS (production) |
| **IaC** | Terraform |

**Purpose:**
Prior Auth Radar gives clinical and administrative staff an AI-powered interface to initiate, track, and resolve prior authorization requests in real time. Instead of navigating complex payer portals, staff describe the clinical scenario in natural language and the system determines authorization requirements, surfaces relevant payer criteria, and guides the submission process.

This repo contains two independent implementations: a Next.js/Vercel app for rapid API demonstration, and a fully separate AWS production implementation hosted on ECS with Claude via Bedrock and pgvector on RDS for multi-turn clinical reasoning. The two implementations do not share infrastructure.

**Healthcare Value:**
Prior authorization is the single largest source of care delay in the U.S. system. 93% of physicians report that PA delays necessary care. This tool closes the gap between the clinical order and the coverage decision, targeting same-visit resolution where the underlying API supports it.

**Implementation 1: Next.js Demo (Vercel)**
Next.js 15, TypeScript, Tailwind CSS, Optum Real Prior Auth API, synthetic mock data fallback for demo mode.

**Implementation 2: AWS Production**
Node.js backend, Claude via Amazon Bedrock, pgvector on Amazon RDS (vector memory), Amazon ECS (container cluster hosting), Terraform IaC (`infrastructure/` folder).

**Production Environment Variables (`backend/.env.example`):**
`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION`, Bedrock model IDs

**Architecture:**

```
 IMPLEMENTATION 1: Next.js Demo
┌─────────────────────────────────────────────────────────────┐
│                     Prior Auth Radar                        │
│                    (Next.js / Vercel)                       │
└───────────────────┬─────────────────────────────────────────┘
                    │
         ┌──────────▼──────────┐
         │  Optum Real         │
         │  Prior Auth API     │
         │  (Sandbox / Mock)   │
         └──────────┬──────────┘
                    │
         ┌──────────▼──────────────────────────────┐
         │   Auth requirements + Clinical guidance  │
         │   + Submission checklist                 │
         └─────────────────────────────────────────┘

 IMPLEMENTATION 2: AWS Production
┌─────────────────────────────────────────────────────────────┐
│                     Prior Auth Radar                        │
│                  (AWS ECS Container Cluster)                │
└───────────────────┬─────────────────────┬───────────────────┘
                    │                     │
         ┌──────────▼──────────┐ ┌────────▼──────────────────┐
         │  Optum Real         │ │  Amazon Bedrock            │
         │  Prior Auth API     │ │  Claude (inference)        │
         │                     │ │  pgvector on RDS (memory)  │
         └──────────┬──────────┘ └────────┬──────────────────┘
                    │                     │
         ┌──────────▼─────────────────────▼───────────┐
         │          Response Synthesis                  │
         │   Auth requirements + Clinical guidance     │
         │   + Submission checklist                    │
         └─────────────────────────────────────────────┘

Infrastructure provisioned via Terraform (infrastructure/)
```

---

### 3. Patient Cost Clarity

**Real-time cost estimates before the patient leaves the exam room.**

| | |
|---|---|
| **GitHub** | [paullopez-ai/patient-cost-clarity-starter](https://github.com/paullopez-ai/patient-cost-clarity-starter) |
| **Live Demo** | [patient-cost-clarity-starter.vercel.app](https://patient-cost-clarity-starter.vercel.app) |
| **Primary API** | Optum Real Cost Transparency |
| **AI Layer** | Azure OpenAI GPT-5.4 + Semantic Kernel |
| **Cloud Platform** | Azure (production) |
| **IaC** | Bicep |

**Purpose:**
Patient Cost Clarity translates Optum Real cost transparency data into plain-language patient estimates. Providers enter procedure codes and patient insurance details; the application returns real-time cost estimates enriched with AI-generated explanations that front-desk staff can communicate confidently without clinical coding expertise.

This repo contains two independent implementations: a Next.js/Vercel app for rapid API demonstration, and a fully separate Azure production implementation hosted on Azure Container Apps with Azure OpenAI GPT-5.4, Semantic Kernel orchestration, Azure AI Search for RAG, and Application Insights for observability. The two implementations do not share infrastructure.

**Healthcare Value:**
Surprise billing is a top driver of patient dissatisfaction and post-visit disputes. Federal price transparency requirements are expanding. Giving patients accurate cost estimates at scheduling or check-in reduces billing disputes, improves collection rates, and rebuilds trust at a critical touchpoint. The AI layer ensures the response is readable by a patient, not just a billing specialist.

**Implementation 1: Next.js Demo (Vercel)**
Next.js 15, TypeScript, Tailwind CSS, Optum Real Cost Transparency API, synthetic mock data fallback for demo mode.

**Implementation 2: Azure Production**
Node.js backend, Azure OpenAI GPT-5.4, Semantic Kernel (orchestration), Azure AI Search (RAG for coverage rules), Azure Container Apps (container cluster hosting), Application Insights (observability), Bicep IaC.

**Production Environment Variables (`backend/.env.example`):**
`AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY`, `AZURE_SEARCH_ENDPOINT`, `APPINSIGHTS_INSTRUMENTATION_KEY`

**Architecture:**

```
 IMPLEMENTATION 1: Next.js Demo
┌─────────────────────────────────────────────────────────────┐
│                  Patient Cost Clarity                       │
│                    (Next.js / Vercel)                       │
└───────────────────┬─────────────────────────────────────────┘
                    │
         ┌──────────▼──────────┐
         │  Optum Real Cost    │
         │  Transparency API   │
         │  (Sandbox / Mock)   │
         └──────────┬──────────┘
                    │
         ┌──────────▼──────────────────────────────────┐
         │   Cost estimates + Plain-language summary    │
         └─────────────────────────────────────────────┘

 IMPLEMENTATION 2: Azure Production
┌─────────────────────────────────────────────────────────────┐
│                  Patient Cost Clarity                       │
│             (Azure Container Apps Cluster)                  │
└───────────────────┬─────────────────────┬───────────────────┘
                    │                     │
         ┌──────────▼──────────┐ ┌────────▼──────────────────┐
         │  Optum Real Cost    │ │  Azure OpenAI GPT-5.4      │
         │  Transparency API   │ │  Semantic Kernel           │
         │                     │ │  Azure AI Search (RAG)     │
         └──────────┬──────────┘ └────────┬──────────────────┘
                    │                     │
         ┌──────────▼─────────────────────▼───────────┐
         │          Patient-Facing Estimate            │
         │   Procedure costs + Insurance breakdown     │
         │   + Plain-language summary for patient      │
         └─────────────────────────────────────────────┘

Observability via Application Insights
Infrastructure provisioned via Bicep
```

---

### 4. Optum Real Eligibility Starter

**Instant eligibility verification with AI-powered coverage interpretation.**

| | |
|---|---|
| **GitHub** | [paullopez-ai/optum-real-eligibility-starter](https://github.com/paullopez-ai/optum-real-eligibility-starter) |
| **Live Demo** | Coming soon (Vercel deployment pending) |
| **Primary API** | Optum Real Eligibility |
| **AI Layer** | Claude / OpenAI (LLM-agnostic via env variable) |

**Purpose:**
The Eligibility Starter provides a clean interface for real-time eligibility verification against Optum's payer network. Beyond raw eligibility status, the AI layer interprets coverage details, flags potential coverage gaps, and surfaces actionable guidance for staff. The goal is to eliminate the phone-based eligibility verification workflow that consumes significant front-desk time.

**Healthcare Value:**
Eligibility verification failures are one of the top causes of claim denials. Manual eligibility checks via phone or payer portals average 8-12 minutes per patient. For a 30-provider group practice seeing 200 patients per day, that is a meaningful operational cost. Real-time API-based verification with AI interpretation reduces this to seconds and surfaces issues before the patient encounter begins.

**Tech Stack:**
Next.js 15, TypeScript, Tailwind CSS, Optum Real Eligibility API (sandbox), Claude Sonnet / OpenAI GPT-5.4 (switchable), Vercel deployment (pending), synthetic mock data for demo mode.

**Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│             Optum Real Eligibility Starter                  │
│                    (Next.js / Vercel)                       │
└───────────────────┬─────────────────────┬───────────────────┘
                    │                     │
         ┌──────────▼──────────┐ ┌────────▼────────────┐
         │  Optum Real         │ │  AI Interpretation   │
         │  Eligibility API    │ │  Coverage gap flags  │
         │  (Sandbox)          │ │  + Staff guidance    │
         └──────────┬──────────┘ └────────┬────────────┘
                    │                     │
         ┌──────────▼─────────────────────▼───────────┐
         │          Eligibility Summary                 │
         │   Coverage status + Benefit breakdown       │
         │   + Denial risk flags + Recommended action  │
         └─────────────────────────────────────────────┘
```

---

### 5. Claim Pre-Check

**Validate before you submit. Catch denials before they happen.**

| | |
|---|---|
| **GitHub** | [paullopez-ai/claim-precheck](https://github.com/paullopez-ai/claim-precheck) |
| **Live Demo** | [claim-precheck-starter.vercel.app](https://claim-precheck-starter.vercel.app) |
| **Primary API** | Optum Real Claim Pre-Check |
| **AI Layer** | Claude / OpenAI (LLM-agnostic via env variable) |

**Purpose:**
Claim Pre-Check runs a submitted claim through Optum Real's validation engine before it reaches the payer, returning a scored risk assessment and specific correction guidance. The AI layer translates technical validation failures into plain-language action items that clinical coders and billing staff can act on immediately without payer-specific expertise.

**Healthcare Value:**
54% of initially denied claims are ultimately paid after appeal, meaning most denials represent rework costs rather than coverage decisions. The average cost to rework a denied claim is $25-$118 depending on complexity. Pre-submission validation eliminates the rework cycle at the source. For a mid-size health system processing 10,000 claims per month, even a 20% reduction in denials generates significant cost savings.

**Tech Stack:**
Next.js 15, TypeScript, Tailwind CSS, Optum Real Claim Pre-Check API (sandbox), Claude Sonnet / OpenAI GPT-5.4 (switchable), Vercel deployment, API console with mock/sandbox toggle.

**Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                    Claim Pre-Check                          │
│                    (Next.js / Vercel)                       │
└───────────────────┬─────────────────────┬───────────────────┘
                    │                     │
         ┌──────────▼──────────┐ ┌────────▼────────────┐
         │  Optum Real Claim   │ │  AI Risk Analysis    │
         │  Pre-Check API      │ │  Plain-language      │
         │  (Sandbox)          │ │  correction guidance │
         └──────────┬──────────┘ └────────┬────────────┘
                    │                     │
         ┌──────────▼─────────────────────▼───────────┐
         │          Pre-Submission Report               │
         │   Validation score + Failure codes          │
         │   + Correction checklist + Resubmit path    │
         └─────────────────────────────────────────────┘
```

---

### 6. Claim Status Radar

**Real-time claim tracking with AI-powered next-step intelligence.**

| | |
|---|---|
| **GitHub** | [paullopez-ai/claim-status-radar-starter](https://github.com/paullopez-ai/claim-status-radar-starter) |
| **Live Demo** | Coming soon (Vercel deployment pending) |
| **Primary API** | Optum Real Claim Status |
| **AI Layer** | Claude / OpenAI (LLM-agnostic via env variable) |

**Purpose:**
Claim Status Radar surfaces real-time claim status from Optum's payer network and uses AI to interpret status codes, predict likely outcomes, and recommend specific next actions for billing staff. Rather than a raw status feed, it delivers a work queue: here is what is pending, here is what needs attention today, here is exactly what to do about it.

**Healthcare Value:**
Billing staff spend a significant portion of their day on status calls to payers, each taking 8-20 minutes. Real-time programmatic status access eliminates the call entirely. The AI layer converts cryptic claim status codes into a prioritized action queue, which is the difference between a status lookup tool and an actual workflow accelerator.

**Tech Stack:**
Next.js 15, TypeScript, Tailwind CSS, Optum Real Claim Status API (sandbox), Claude Sonnet / OpenAI GPT-5.4 (switchable), Vercel deployment (pending), synthetic mock data for demo mode.

**Architecture:**

```
┌─────────────────────────────────────────────────────────────┐
│                  Claim Status Radar                         │
│                    (Next.js / Vercel)                       │
└───────────────────┬─────────────────────┬───────────────────┘
                    │                     │
         ┌──────────▼──────────┐ ┌────────▼────────────┐
         │  Optum Real Claim   │ │  AI Status Engine    │
         │  Status API         │ │  Outcome prediction  │
         │  (Sandbox)          │ │  + Action queue      │
         └──────────┬──────────┘ └────────┬────────────┘
                    │                     │
         ┌──────────▼─────────────────────▼───────────┐
         │          Claim Work Queue                    │
         │   Status by claim + Priority flags          │
         │   + Recommended actions + Escalation paths  │
         └─────────────────────────────────────────────┘
```

---

## Shared Architecture Principles

All POCs in this collection follow the same foundational design decisions.

**LLM-Agnostic:** Every application selects its AI provider via an environment variable (`LLM_PROVIDER=claude` or `LLM_PROVIDER=openai`). No code changes are required to switch between Claude and OpenAI. This ensures the architecture survives any individual provider's product changes.

**Demo-First Design:** Every application ships with a mock data mode that requires no API credentials. Switching to sandbox mode connects to Optum Real sandbox environments. Production mode requires valid Optum Real API credentials and a contracted relationship.

**HIPAA-Conscious by Default:** No real patient data is required or accepted in sandbox or demo modes. All synthetic data follows realistic clinical patterns while remaining entirely fictitious. Production deployments require appropriate BAA agreements and PHI handling controls.

**Provider-Only Focus:** The majority of use cases target the provider side of the payer-provider relationship: hospitals, health systems, physician groups, and their administrative and clinical staff.

**Zero External Dependencies in Demo Mode:** All POCs can be demoed in a completely offline-compatible mock mode. No API calls, no credentials, no network dependency.

---

## Technology Stack Summary

| Layer | Technology |
|---|---|
| Demo Framework | Next.js 15 (App Router), TypeScript, Tailwind CSS |
| Demo Hosting | Vercel |
| Payer Data | Optum Real APIs (sandbox / mock) |
| AWS Production (payer-auth-intelligence) | Claude Sonnet via Amazon Bedrock, LangGraph.js StateGraph, DynamoDB, Lambda, Amplify, AWS CDK |
| AWS Production (prior-auth-radar) | Claude via Amazon Bedrock, pgvector on RDS, Amazon ECS, Terraform |
| Azure Production (patient-cost-clarity-starter) | Azure OpenAI GPT-5.4, Semantic Kernel, Azure AI Search, Azure Container Apps, Bicep |
| Observability (Azure) | Application Insights |
| All other POCs | Next.js 15, TypeScript, Tailwind CSS, Vercel, Claude Sonnet direct |
| Data | Synthetic mock data (HIPAA-compliant patterns) |

---

## About This Work

These POCs represent applied R&D at the intersection of payer data infrastructure and generative AI. The underlying thesis: Optum's ownership of the payer data layer via Real APIs is an architectural advantage no AI provider can replicate. The question this collection explores is what becomes possible when you connect that data layer to a capable reasoning model.

Built by **Paul Lopez** | Principal AI Architect
[LinkedIn](https://www.linkedin.com/in/paullopez1/)  
[paullopez.ai](https://paullopez.ai) | [GitHub: paullopez-ai](https://github.com/paullopez-ai)

---

> All applications use Optum Real API sandbox environments. All data is synthetic and for demonstration and educational purposes only. No real patient data is transmitted. Production use requires appropriate Optum Real API agreements and HIPAA compliance controls.
