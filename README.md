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

### 1. Prior Auth Radar

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

### 2. Patient Cost Clarity

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

### 3. Optum Real Eligibility Starter

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

### 4. Claim Pre-Check

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

### 5. Claim Status Radar

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

**Provider-Only Focus:** Every use case targets the provider side of the payer-provider relationship: hospitals, health systems, physician groups, and their administrative and clinical staff.

**Zero External Dependencies in Demo Mode:** All POCs can be demoed in a completely offline-compatible mock mode. No API calls, no credentials, no network dependency.

---

## Technology Stack Summary

| Layer | Technology |
|---|---|
| Demo Framework | Next.js 15 (App Router), TypeScript, Tailwind CSS |
| Demo Hosting | Vercel |
| Payer Data | Optum Real APIs (sandbox / mock) |
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
