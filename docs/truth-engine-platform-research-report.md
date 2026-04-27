# Truth Engine Platform Research Report

**Status:** Architecture research and roadmap
**Briefing cadence anchor:** May 18, 2026
**Companion documents:** `forensic-audit/tradecraft-reframe.md`, `forensic-audit/form-102-pipeline.md`

---
## Executive Summary

The current estate says two things at once. First, `gabearce1-oss/truthengine360` contains real product ambition: a React application, Base44 auth/client plumbing, workflow-facing serverless functions, a research pipeline subproject, and UI modules for TruthEngine, research data, n8n pipelines, reporting, and case-oriented workflows.

Second, it is architecturally split across multiple competing futures: a Base44-generated app, a nested Python `research-db` pipeline, and separate FastAPI/Docker deployment artifacts that do not cleanly line up with the live application structure.

In plain English: there is a platform here, but it currently has three steering wheels.

The other audited repositories are not yet material building blocks. Connector inspection showed Case-Vault is effectively empty/bootstrap-level, while Autocraft and claude are concept-stage or README-stage repos rather than active services. The platform plan should therefore treat those repos as placeholders or future bounded contexts, not as reusable production modules today.

The meaningful codebase for this initiative is **truthengine360**.

The best path forward is not a total rewrite and not a continued pile-on in the current repo. The best path is a three-round planning sequence followed by a controlled strangler migration into an evidence-centric platform with four planes: experience, orchestration, evidence/data, and observability/security.

Recommended workflow split:

- **n8n** for fast SaaS and low-code integrations.
- **Temporal** for durable, never-lose mission-critical workflows.
- **Prefect** for Python-heavy research/data pipelines.

Recommended data stack:

- **PostgreSQL + pgvector** as transactional source of truth.
- **OpenSearch** for hybrid/semantic retrieval.
- **Neo4j** for relationship/evidence graphing.
- **Object storage** for source artifacts.

Security and delivery should be treated as first-class architecture from day one (SSDF, SBOM, API security baselines, and observability).

The commercial context matters as well: the April 2026 SEO/positioning artifact and the **May 18, 2026** briefing cadence mean the roadmap should prioritize evidence reliability, publishable outputs, and workflow auditability ahead of broad feature sprawl.

---

## Connector Reconnaissance and Audit Scope

The connector scan produced a clear hierarchy of value:

1. **GitHub**: primary source of implementation truth.
2. **Google Drive**: strategic/go-to-market context.
3. **Google Calendar**: time-sensitive briefing milestones.

Figma and Acrobat were not materially leveraged due to missing actionable file keys/URLs or format mismatch.

Additional external research catalog used for topic discovery:

- **EBSCO Research Starters:** https://www.ebsco.com/research-starters

### High-value file inspection set

| Repo | Files inspected first | Why high-value |
|---|---|---|
| `gabearce1-oss/truthengine360` | `README.md`, `package.json`, `src/App.jsx`, `src/lib/AuthContext.jsx`, `src/api/base44Client.js`, `src/lib/app-params.js`, `src/lib/query-client.js`, `src/functions/reportAutomation.js`, `base44/functions/*`, `src/pages/TruthEngine.jsx`, `src/.github/workflows/deploy.yml`, `src/DEPLOYMENT_GUIDE.md`, `src/research-db/*`, `src/deploy.sh`, `src/components/te/N8nPipeline.jsx` | Enough to identify stack, integrations, orchestration intent, deployment drift, and evidence-processing design |
| `gabearce1-oss/Case-Vault` | Initial commit/bootstrap | Confirmed non-material runtime value |
| `gabearce1-oss/Autocraft` | README/initial commit | Confirmed concept-stage status |
| `gabearce1-oss/claude` | README/initial commit | Confirmed concept-stage status |
| Google Drive | `TruthEngine360_SEO_Positioning_Playbook` | Strategic priorities and deadlines |

---

## Repository Audit Findings

### What Is Working

- React + Base44 shell exists and runs.
- Route/auth/query/client wiring is coherent for a generated app baseline.
- Workflow-oriented product surfaces exist (truth workflows, reports, case activities).
- Evidence/FOIA/DCAS-oriented serverless function intent exists in `base44/functions`.
- A nested Python `research-db` project contains reusable long-term architectural signal.

### What Is Not Working Yet

1. **Workflow source-of-truth is fragmented**: some workflow intent is hard-coded in UI components.
2. **Runtime model is split**: Base44 app + Base44 functions + nested Python pipeline + separate FastAPI/Docker artifacts.
3. **Deployment narrative drift**: deployment docs/scripts describe structures that do not match the live app shape.
4. **Evidence fidelity gaps**: function behaviors appear partially simulated/heuristic in places where deterministic lineage is required.
5. **Potential auth defect**: `syncToGitHub` authorization semantics appear suspect and need explicit validation/fix.

### Audit Verdict by Repo

| Repo | Architecture | Modules | Automation patterns | Main gaps | Verdict |
|---|---|---|---|---|---|
| `truthengine360` | Mixed architecture (Base44 React + Base44 functions + nested Python + drifting deployment artifacts) | TruthEngine UI, report automation, FOIA/DCAS/evidence workflows, research-db docs | UI-defined workflows + function-level automation + ingestion ambitions | No single runtime model, deployment drift, workflow source-of-truth weakness, auth/evidence concerns | Promising but unstable; use as extraction source, not end-state |
| `Case-Vault` | Bootstrap-only | Minimal | None reusable yet | No schema/service contracts | Define before coding |
| `Autocraft` | Concept-stage | Minimal | None reusable yet | No runtime substance | Do not depend on it |
| `claude` | Concept-stage | Minimal | None reusable yet | No runtime substance | Do not depend on it |

---

## Planning rounds

The plan deliberately separates planning from building.

## Round one — estate stabilization and truth mapping (2–3 weeks)

| Field | Plan |
|---|---|
| Objectives | Freeze architecture drift; inventory workflows; define canonical domain model; classify functions as keep/replace/prototype/delete |
| Assumptions | `truthengine360` remains live reference; other repos are non-constraining; near-term cadence favors reliability over breadth |
| Deliverables | Repository map; connector inventory; data glossary; event catalog; ADR set; security backlog; migration inventory; workflow source-of-truth document |
| Risks | Hidden Base44 dependencies; undocumented credentials; logic trapped in UI literals; inability to export operational state |
| Success metrics | 100% critical workflow mapping; 100% secrets ownership identified; 90%+ function classification; first E2E trace |
| Required roles | Platform lead, full-stack, data/automation, security, analyst/PM |
| Decision points | Keep or retire Base44 shell; workflow split finalization; canonical evidence object and approvals model |

### Round-one timeline anchors

- **2026-04-28 to 2026-05-12**
- Repo inventory → workflow mapping → data model → secrets/security → tracing baseline → ADR closure

## Round two — platform skeleton and service extraction (4–6 weeks)

| Field | Plan |
|---|---|
| Objectives | Build platform skeleton; extract first durable workflows; establish API/event contracts |
| Assumptions | Dual-run period acceptable; first extraction targets intake/evidence/approvals/reporting |
| Deliverables | API gateway; auth; evidence service; workflow runtime integration; event bus; Postgres+pgvector schema; object layout; initial search index; CI baseline |
| Risks | Dual-write inconsistencies; early indexing complexity; connector throttling; under-specified approval semantics |
| Success metrics | One workflow rerouted end-to-end; 95% persisted workflow state; p95 API latency < 400ms on pilot endpoints; deterministic replay on at least one evidence flow |
| Required roles | 2 backend, 1 frontend, 1 data, 1 platform engineer |
| Decision points | OpenSearch timing; JetStream vs SQS default; Neo4j introduction point |

### Round-two timeline anchors

- **2026-05-13 to 2026-06-03**
- API/auth → data/storage bootstrap → n8n lane → Temporal worker → Prefect lane → first extracted workflow

## Round three — hardening, migration, operating model (6–8 weeks)

| Field | Plan |
|---|---|
| Objectives | Production hardening; staged migration; runbooks and KPI operations |
| Assumptions | Round-two services are stable enough for staged rollout |
| Deliverables | GitOps/CD; SLOs; alerting; SBOM; security gates; audit logs; DR plan; rollout playbooks; KPI dashboard; operator training |
| Risks | Dual-mode user confusion; observability blind spots; relevance complaints; cost creep from parallel infra |
| Success metrics | 99.5% workflow success on migrated flows; MTTR < 30 minutes; 60% manual touch-time reduction; >95% trace coverage |
| Required roles | Platform lead, 2 backend, 1 frontend, 1 SRE/platform, 1 QA/security, stakeholder owner |
| Decision points | Cutover sequencing; n8n cloud vs self-host; repo boundary splits |

### Round-three timeline anchors

- **2026-06-05 to 2026-06-27**
- SLO/alerts → SBOM/security gates → migration batches → runbooks/training → KPI optimization

---

## Platform architecture and workflow mapping

### Target architecture

- React operator console retained.
- OpenAPI-first service layer.
- Hybrid workflow plane (n8n + Temporal + Prefect).
- Layered persistence and retrieval (Postgres+pgvector, OpenSearch, Neo4j, object storage).
- Event bus and OpenTelemetry-backed observability.

### Logical entities

- `CASE`
- `DOCUMENT`
- `EVIDENCE_ITEM`
- `WORKFLOW_RUN`
- `OCR_CHUNK`
- `APPROVAL`
- `TASK_EVENT`
- `CONNECTOR`
- `REPORT`
- `REPORT_ARTIFACT`

### Current-state problem map

- UI panels carry workflow literals.
- Base44 functions act as isolated automation islands.
- Research pipeline exists but is not integrated as a first-class runtime plane.
- Deployment docs/scripts point to a different architecture.

### Optimized workflow shape

1. Intake event.
2. Create case/document records.
3. Store source artifact.
4. Emit `document.received`.
5. Prefect OCR/extraction.
6. Temporal verification/approval.
7. Index to Postgres/OpenSearch/Neo4j.
8. Generate report package.
9. Publish export + audit trail.

---

## Decision Tables

### Workflow Engines

| Option | Best use | Strengths | Weaknesses | Recommendation |
|---|---|---|---|---|
| n8n | SaaS integrations and webhook glue | Node-based workflows, draft/publish, execution visibility, custom nodes | Not ideal as sole durable core for long-running compliance workflows | Use for connector-heavy automations |
| Temporal | Case-critical durable workflows | Crash-proof durable execution, retries/compensation, long waits | Higher engineering overhead | Use as workflow spine for high-value flows |
| Prefect | OCR/NLP/research/backfills | Python-native orchestration, state/recovery, event-driven operation | Less suited than Temporal for deep business-process durability | Use for research/data plane |

### Storage and Search

| Option | Role | Why fit | Why not alone | Recommendation |
|---|---|---|---|---|
| PostgreSQL + pgvector | System-of-record + vectors | ACID + joins + ANN/exact retrieval in operational store | Not a complete search/ranking platform alone | Primary transaction store |
| OpenSearch | Hybrid + semantic retrieval | Search-grade indexing and vector/hybrid retrieval | Not ideal sole system-of-record | Secondary retrieval/search |
| Neo4j | Evidence graph | Relationship-centric modeling and traversal | Not replacement for transactional recordkeeping | Graph sidecar |

### Messaging

| Option | Best fit | Properties | Recommendation |
|---|---|---|---|
| NATS JetStream | Internal event bus with replay | Persistence, replay, replication, RAFT consistency | Default internal bus |
| Kafka | Heavy event-streaming platform | Durable streaming + broad ecosystem | Adopt only when scale/complexity justify |
| SQS | Managed cloud queue | Durable managed queue (standard/FIFO) | Good at cloud edges/simple decoupling |
| Redis Streams | Lightweight append-only eventing | Consumer groups + efficient inserts | Use for local/simple streams, not core bus |

---

## Prototype Blueprint

The first prototype should focus on five modules only:

1. Document intake
2. Evidence extraction dispatch
3. Approval state
4. Report generation
5. Connector façade

### FastAPI intake skeleton

```python
# app/main.py
from datetime import datetime
from uuid import uuid4

from fastapi import FastAPI, HTTPException
from pydantic import BaseModel, Field
import asyncio

app = FastAPI(title="Truth Engine API", version="0.1.0")

FAKE_DB = {"cases": {}, "documents": {}, "workflow_runs": {}}

class IntakeRequest(BaseModel):
    case_external_id: str = Field(..., min_length=1)
    title: str
    source_uri: str
    content_type: str = "application/pdf"
    uploaded_by: str

class IntakeResponse(BaseModel):
    case_id: str
    document_id: str
    workflow_run_id: str
    status: str

async def publish_event(topic: str, payload: dict) -> None:
    await asyncio.sleep(0)

@app.post("/v1/documents:intake", response_model=IntakeResponse, status_code=202)
async def intake_document(req: IntakeRequest) -> IntakeResponse:
    case_id = FAKE_DB["cases"].get(req.case_external_id) or str(uuid4())
    document_id = str(uuid4())
    workflow_run_id = str(uuid4())

    FAKE_DB["cases"][req.case_external_id] = case_id
    FAKE_DB["documents"][document_id] = {
        "case_id": case_id,
        "title": req.title,
        "source_uri": req.source_uri,
        "content_type": req.content_type,
        "uploaded_by": req.uploaded_by,
        "created_at": datetime.utcnow().isoformat(),
        "status": "RECEIVED",
    }

    FAKE_DB["workflow_runs"][workflow_run_id] = {
        "case_id": case_id,
        "document_id": document_id,
        "workflow_name": "document_intake",
        "status": "QUEUED",
        "created_at": datetime.utcnow().isoformat(),
    }

    await publish_event(
        "document.received",
        {
            "case_id": case_id,
            "document_id": document_id,
            "workflow_run_id": workflow_run_id,
            "source_uri": req.source_uri,
            "content_type": req.content_type,
        },
    )

    return IntakeResponse(
        case_id=case_id,
        document_id=document_id,
        workflow_run_id=workflow_run_id,
        status="QUEUED",
    )

@app.get("/v1/workflow-runs/{workflow_run_id}")
async def get_workflow_run(workflow_run_id: str):
    run = FAKE_DB["workflow_runs"].get(workflow_run_id)
    if not run:
        raise HTTPException(status_code=404, detail="workflow run not found")
    return run
```

### TypeScript intake router skeleton

```ts
// workers/intake-router.ts
import { z } from "zod";

const DocumentReceived = z.object({
  case_id: z.string().uuid(),
  document_id: z.string().uuid(),
  workflow_run_id: z.string().uuid(),
  source_uri: z.string().url(),
  content_type: z.string(),
});

type DocumentReceived = z.infer<typeof DocumentReceived>;

export async function handleDocumentReceived(raw: unknown) {
  const evt: DocumentReceived = DocumentReceived.parse(raw);

  const route =
    evt.content_type === "application/pdf"
      ? "prefect:ocr-and-extract"
      : "temporal:manual-review";

  return {
    workflowRunId: evt.workflow_run_id,
    dispatchedTo: route,
    at: new Date().toISOString(),
  };
}
```

### Minimal OpenAPI contract skeleton

```yaml
openapi: 3.1.1
info:
  title: Truth Engine API
  version: 0.1.0
paths:
  /v1/documents:intake:
    post:
      summary: Intake a document and start evidence workflow
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/IntakeRequest'
      responses:
        '202':
          description: Accepted for processing
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/IntakeResponse'
  /v1/workflow-runs/{workflowRunId}:
    get:
      summary: Get workflow run status
      parameters:
        - in: path
          name: workflowRunId
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Workflow state
components:
  schemas:
    IntakeRequest:
      type: object
      required: [case_external_id, title, source_uri, uploaded_by]
      properties:
        case_external_id: { type: string }
        title: { type: string }
        source_uri: { type: string, format: uri }
        content_type: { type: string, default: application/pdf }
        uploaded_by: { type: string }
    IntakeResponse:
      type: object
      required: [case_id, document_id, workflow_run_id, status]
      properties:
        case_id: { type: string, format: uuid }
        document_id: { type: string, format: uuid }
        workflow_run_id: { type: string, format: uuid }
        status: { type: string }
```

---

## Delivery Shape and Roadmap

### Migration strategy

Use a strangler pattern:

- Keep the existing React shell for operator continuity.
- Move system truth into new APIs/services/events.
- Dual-run and retire flows incrementally.

Suggested migration order:

1. Document intake
2. Evidence extraction
3. Approvals
4. Reporting/export
5. External syncs

### Infrastructure posture

| Option | Best fit | Tradeoff | Recommendation |
|---|---|---|---|
| ECS/Fargate | Fast pilot/prototype | Lower ops burden, less k8s flexibility | Start here for pilot |
| EKS + Argo CD | Scaled multi-service platform | More control and complexity/cost | Adopt at scale |

### Modeled monthly infra ranges (non-quote estimates)

| Environment | Footprint summary | Range |
|---|---|---|
| Prototype | API + single worker lane + small DB/storage/logging | $700–$1,500 |
| Pilot | HA API + 2–3 lanes + larger DB + search + monitoring | $2,500–$6,000 |
| Production | HA multi-lane + larger search + graph + DR/observability | $8,000–$20,000+ |

### Security/compliance priorities

- Integrate NIST SSDF practices into SDLC.
- Add AI-specific controls where model workflows apply.
- Generate SBOM continuously.
- Treat OWASP API risks as first-order design constraints.
- Instrument with OpenTelemetry for correlated trace/metric/log telemetry.

---

## Prioritized roadmap outcomes

| Milestone | Outcome | KPI |
|---|---|---|
| Platform truth map complete | Workflows/schemas/connectors/secrets ownership documented | 100% critical workflow coverage |
| First intake workflow migrated | New API/event bus production path for one flow family | 95% success on new path |
| Evidence pipeline durable | OCR/extraction/approval is replayable + auditable | 0 unrecoverable workflow losses |
| Search and graph online | Evidence searchable by keyword + semantic + relationships | Retrieval relevance improves release-over-release |
| Reporting/export hardened | Evidence-driven briefing packages and audit bundles | 60% reduction in manual reporting time |
| Legacy retirement | Old runtime high-value flows disabled | 70%+ workload on new platform |

---

## Recommended stack snapshot

| Layer | Recommended choice | Why |
|---|---|---|
| Front end | React + TypeScript + TanStack Query | Fits current reality and operator use cases |
| API | FastAPI | Strong OpenAPI-first velocity |
| Durable workflows | Temporal | Never-lose case-critical processing |
| Connector automation | n8n | Fast integration iteration |
| Research/data jobs | Prefect | Python-native orchestration |
| System of record | PostgreSQL + pgvector | Transactional truth + vectors |
| Search | OpenSearch | Hybrid/semantic retrieval |
| Graph | Neo4j | Relationship-intensive evidence modeling |
| Event bus | NATS JetStream | Replay + persistence without Kafka-level ceremony |
| Observability | OTel + Prometheus/Grafana or CloudWatch | Correlated telemetry |
| CI | GitHub Actions | Native repo ecosystem fit |
| CD | Argo CD (when on k8s) | GitOps delivery at scale |

---

## Open Questions

1. How much operational state currently lives inside Base44 vs exportable/in-repo systems?
2. Which visible workflows are production-critical versus narrative prototypes?
3. Given deadlines around **May 18, 2026**, should a temporary stabilize/export phase precede deeper extraction?
4. Should `helpful-command-flow-core.zip` become formal scope once repository boundaries are confirmed?

## Bottom Line

Truth Engine should become an evidence-grade workflow platform, not a dashboard with aspirational automation.

The repositories already point in this direction. The core requirement now is disciplined architecture convergence: one runtime truth model, durable workflows, evidence lineage, and controlled strangler migration.


---

## Sources and References

- EBSCO Research Starters: https://www.ebsco.com/research-starters
