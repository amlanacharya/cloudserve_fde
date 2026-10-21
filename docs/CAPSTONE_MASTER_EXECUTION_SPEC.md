# CloudServe Support Automation

## Capstone Master Execution Specification

| Field | Value |
| --- | --- |
| Status | Authoritative execution specification |
| Baseline provider | Groq |
| Baseline model | `meta-llama/llama-4-scout-17b-16e-instruct` |
| Optional optimisation model | `llama-3.1-8b-instant` for non-authoritative summarisation only |
| Delivery | Individual capstone submission |
| Completion standard | All evidence gates in this document are satisfied; no result is invented or inferred. |

## 1. Purpose and definition of done

Build a trustworthy support-triage and retrieval service for CloudServe. It must answer only documentation-backed, safe tickets; escalate all other tickets with context that makes the human agent faster; and produce auditable evidence of every decision.

This document is the one place to run the capstone from. It does **not** replace the assessed deliverables: the five completed workbooks, effort log, repository, report, and video must still be produced and submitted in their prescribed formats. It tells you exactly what each must contain and when it can honestly be marked complete.

### Final completion conditions

The capstone is complete only when all of the following are true:

1. The repository clears acceptance criteria A1-A12 from a clean checkout.
2. The full evaluation input is processed unattended by one documented command, producing the decision log and metrics report.
3. The evaluation results, governance controls, fairness audit, risks, and limitations are recorded honestly.
4. The PRD has been revised at least once and the revision log explains why.
5. All five workbooks, effort log, report, video, and correctly structured archive are present.

## 2. Product decision

CloudServe does not need a general chatbot. It needs a controlled retrieval-and-triage service.

The system receives a ticket from one of four channels, normalises it, classifies intent and urgency, retrieves reviewed documentation, then makes one deterministic decision:

```text
safe + confident + grounded -> automated cited response
uncertain / no useful source / human-only topic -> escalation package
guardrail failure -> response blocked and escalated
```

The design is evidence-led. In the development set, 357 of 500 tickets (71.4%) are documentation-answerable and 311 (62.2%) are expected to auto-respond; 87 tickets are flagged as never suitable for automated response. The opportunity is therefore to make reviewed knowledge findable and make escalation useful, not to automate every customer interaction.

## 3. Authoritative requirements

The full, testable product requirements live in [FINAL_PRODUCT_REQUIREMENTS_DOCUMENT.md](FINAL_PRODUCT_REQUIREMENTS_DOCUMENT.md). That document contains FR-01 to FR-12, NFR-01 to NFR-09, traceability evidence, routing policy, success measures, and release criteria.

The requirements which may never be weakened are:

- All four channels must be normalised.
- Every ticket must receive a logged terminal outcome: auto-response, escalation, or block.
- Cited claims must resolve to the retrieved documentation passages that support them.
- Security/account compromise, billing disputes, commitments, data-location/compliance, roadmap/feature requests, no-hit, low-confidence, and guardrail-failing cases must not auto-respond.
- Every generated response must be validated before release.
- The hidden-set-compatible harness must accept input and output paths; no ticket may be skipped or manually repaired during a run.

## 4. Technical architecture

### Required components

| Component | Responsibility | Evidence produced |
| --- | --- | --- |
| Ingest | Accept email, chat, documentation-comment, and forum tickets; create one normalised ticket object. | Normalised ticket / fallback record. |
| Classifier | Produce intent, urgency, calibrated confidence, and ranked alternatives. | Structured classification record. |
| Retriever | Search only reviewed `documentation.json` articles and return ranked identifiable passages. | Passage IDs, scores, and source text. |
| Router | Apply fixed, documented policy and threshold. | Deterministic route and human-readable reason. |
| Generator | Produce a concise, source-grounded draft with citations. | Structured response draft. |
| Validator | Enforce PII, grounding, instruction-integrity, tone/scope, commitment, and confidence controls. | Guardrail results and block/escalation decision. |
| Persistence | Store decision records, metrics inputs, and evaluation outputs. | Reconstructable audit trail. |
| Evaluation harness | Process any compatible input unattended and write metrics. | Results and metrics report. |

### Recommended implementation choices

| Layer | Baseline choice | Why |
| --- | --- | --- |
| Language | Python 3.10+ | Required-compatible, accessible stack. |
| API | FastAPI | Provides a testable HTTP interface and automatic documentation. |
| Orchestration | LangChain and LangGraph | Separates retrieval, routing, generation, and recovery paths. |
| Retrieval | Chroma plus `all-MiniLM-L6-v2` embeddings | Local, free, and sufficient for the supplied 29-article corpus. |
| Decision store | SQLite | Simple persistent local audit log at capstone scale. |
| Provider | Groq | Fast, OpenAI-compatible API with a free-tier path. |
| Monitoring | Prometheus and Grafana | Required monitoring evidence. |
| Automation | GitHub Actions | Required reproducible test signal. |

Keep model access behind one provider interface. The rest of the pipeline must not know the Groq SDK or API details. This makes provider failure handling and later substitution testable.

## 5. Groq setup and model policy

### Create and protect the API key

1. Go to the Groq Console, sign in, open **API Keys**, and create a new key.
2. Copy it immediately; do not paste it into chat, source code, screenshots, commits, or this document.
3. Create a local `.env` file from `.env.example` with `GROQ_API_KEY=your_key_here`.
4. Add `.env` to `.gitignore`; commit `.env.example` only, with no real values.
5. Fail startup with a clear message if `GROQ_API_KEY` is absent. Never silently use an embedded fallback key.
6. Before submission, search the full repository and commit history for keys or tokens.

Groq supports the environment variable with both its Python client and the OpenAI-compatible endpoint at `https://api.groq.com/openai/v1`. Verify currently active model IDs in your own account with the Models endpoint before beginning a benchmark, because provider availability can change.

### Model configuration

| Use | Model | Policy |
| --- | --- | --- |
| Classification, routing, structured escalation package, and baseline answer generation | `meta-llama/llama-4-scout-17b-16e-instruct` | Required MVP baseline. It supports strict JSON-schema structured outputs in Groq documentation, reducing malformed records. |
| Optional low-latency summarisation experiment | `llama-3.1-8b-instant` | May be evaluated only after the baseline passes. It cannot override routing, safety policy, or a validator decision. |
| Safety controls | Deterministic application code first | Regex/format checks, source-claim checks, route exclusions, and schema validation remain authoritative; no model is the sole guardrail. |

Use model IDs as environment-configurable settings:

```dotenv
GROQ_API_KEY=
GROQ_BASE_URL=https://api.groq.com/openai/v1
GROQ_PRIMARY_MODEL=meta-llama/llama-4-scout-17b-16e-instruct
GROQ_SUMMARY_MODEL=llama-3.1-8b-instant
```

Do not use the optional summary model until the baseline system passes the full validation workflow. If the primary model is unavailable or rate-limited, record the failure and safely escalate; do not switch to a weaker model invisibly during evaluation.

## 6. Data rules

| Dataset | Permitted use | Prohibited use |
| --- | --- | --- |
| `development_tickets.json` | Discovery, classifier/router development, retrieval and prompt iteration. | Treating historical outcomes as the desired future outcome. |
| `validation_tickets.json` | Repeated validation during development, clearly labelled as validation. | Presenting it as a hidden/final assessment result. |
| `ground_truth_responses.json` | Development comparison of completeness and unsafe claims. | Treating it as the retrieval corpus or copying it as customer-facing evidence. |
| `documentation.json` | The only approved customer-answer grounding corpus. | Assuming every ticket has an answer; no-hit is valid. |
| Hidden assessment input | One unattended post-submission run by assessors. | Hardcoding a filename, inspecting it, or tuning against it. |

## 7. Routing and guardrail policy

### Mandatory escalation or block

Never auto-respond to security or account-compromise requests, billing disputes, refunds/service-credit or contractual commitments, data-location/compliance questions, feature/roadmap requests, absent documentation, low-relevance retrieval, low/calibration-failing confidence, or any case with unsafe output.

### Automated response gateway

An automated answer can be released only if all conditions pass:

1. The ticket is not an excluded class.
2. A reviewed passage clears the relevance threshold.
3. Calibrated classification/routing confidence clears the evaluated threshold.
4. The answer’s factual claims map to those retrieved passages.
5. PII, instruction integrity, tone/scope, and commitment checks pass.

### Guardrails that must run on every response

| Guardrail | On failure |
| --- | --- |
| Private data | Block; escalate; never redact and send. |
| Grounding | Block; escalate with unsupported claim identified. |
| Instruction integrity | Block; escalate with suspicious customer input logged. |
| Tone and scope | Block any commitment, inappropriate tone, or out-of-scope response. |
| Confidence floor | Escalate; a missing score is never high confidence. |

## 8. Decision-log contract

Persist a record for every stage/terminal action. At minimum include:

```json
{
  "decision_id": "...",
  "timestamp": "...",
  "ticket_id": "...",
  "stage": "classification | routing | generation | validation",
  "input_summary": "...",
  "model": {"name": "...", "version": "..."},
  "prediction": {"value": "...", "confidence": 0.0},
  "alternatives": [{"value": "...", "confidence": 0.0}],
  "sources_used": [{"doc_id": "...", "score": 0.0}],
  "threshold_applied": 0.0,
  "action_taken": "auto_respond | escalate | block",
  "reason": "human-readable explanation",
  "guardrail_results": {"pii": "pass", "grounding": "pass"},
  "prompt_version": "...",
  "requirement_ids": ["FR-01"]
}
```

The number of processed tickets and terminal decision records must reconcile exactly.

## 9. Delivery plan and evidence gates

### Stage 1 — Discovery

Complete `Stage_1_Discovery_Workbook.docx` before designing. Record all five stakeholders, disagreements, development-data counts, effort map, client success measures, data constraints, risks, and evidence-supported problem statement.

**Gate:** Every claim in the problem statement points to an evidence row. Do not write requirements first.

### Stage 2 — Requirements

Use `FINAL_PRODUCT_REQUIREMENTS_DOCUMENT.md` to complete `Stage_2_PRD_Template.docx`. Preserve requirement IDs, priorities, evidence links, measurable acceptance criteria, scope exclusions, assumptions, success measures, and open decisions.

**Gate:** An engineer who did not attend discovery could build and test every requirement.

### Stage 3 — Prompt library

Create versioned prompts/specifications for classification, routing, answer generation, validation, and evaluation. Each prompt must name the requirement IDs it serves, expected input/output schema, examples, failure handling, version, and test cases.

**Gate:** Every functional requirement maps to at least one prompt/specification and test.

### Stage 4 — Sprint plan

Create a backlog with owner, estimate, dependency, definition of done, and planned day. Protect the end-to-end harness early; cut nonessential scope rather than the unattended run.

**Gate:** Every A1-A12 criterion has an owner, a build item, and a test item.

### Stage 5 — Build, evaluate, revise

Build in this order:

1. Normalise one ticket from each channel.
2. Index the documentation and prove retrieved passages resolve to the corpus.
3. Classify, calibrate, route deterministically, and write logs.
4. Generate cited drafts and prove a guardrail blocks.
5. Run the full chain unattended and produce metrics.
6. Run fairness, privacy, citation, and hallucination checks.
7. Revise the PRD and complete the revision log using actual evidence.

**Gate:** All A1-A12 conditions pass from a clean checkout; metrics and logs reconcile; no false performance claims.

### Stage 6 — Submission

Produce a 20-minute video, a 20-30 page report excluding appendices, completed workbooks/effort log, and complete source repository. Package exactly four top-level folders: `01_Video`, `02_Report`, `03_Workbooks`, and `04_Source_Code`.

**Gate:** Rehearse clean checkout and submission checklist at least one day before submission.

## 10. Acceptance-criteria implementation matrix

| Criterion | Build proof | Required test/evidence |
| --- | --- | --- |
| A1 | README setup/run works from a new directory. | Clean-checkout rehearsal. |
| A2 | Four input adapters produce one schema. | Four-channel test fixtures. |
| A3 | Intent, urgency, and 0-1 confidence are returned. | Schema and fallback tests. |
| A4 | Sources are real retrieved corpus passages. | Source-resolution test. |
| A5 | Same input gets same route under recorded threshold. | Determinism test. |
| A6 | Each citation supports its associated claim. | Citation-audit sample. |
| A7 | At least one live guardrail blocks a response. | Adversarial ticket test. |
| A8 | Persistent records cover all automated decisions. | Count reconciliation. |
| A9 | Full input processes once without intervention. | Harness run log/results. |
| A10 | Metrics write automatically after the run. | Generated metrics file. |
| A11 | No-hit, outage, rate limit, malformed input continue safely. | Failure-injection suite. |
| A12 | One documented test command passes. | CI and local command output. |

## 11. Evaluation plan

The evaluation harness must accept `--input` and `--output` (or equivalent), process every record, and write a machine-readable results file plus report without manual calculation.

| Tier | Required measures |
| --- | --- |
| Business | Tickets processed, automatic answers, escalations, blocks, first-contact resolution, mean/median response time, escalation rate, satisfaction proxy, repeat-contact proxy where supportable. |
| Technical | Precision and recall per intent, confusion matrix, retrieval hit rate, citation accuracy, hallucination rate, median/p95 latency. |
| Governance | Decision-log reconciliation, guardrail activations by type, private-data detections, confidence calibration, and outcome differences by tier, region, and language fluency. |

Do not claim an achieved result until a dated, versioned run produces it. In the report, state the dataset, sample size, system/prompt/model version, run count, method, and limitations. The report must explain what a result means for customer experience and workload, not merely list numbers.

## 12. Governance and release controls

Before any live-deployment claim, provide:

1. Risk register with likelihood, impact, concrete mitigation, and named owner.
2. Fairness audit by customer tier, region, language fluency, and relevant ticket complexity; variation must be under five percentage points or deployment is blocked pending mitigation.
3. Decision-log reconciliation proving complete coverage.
4. Automated scan for private data plus manual response sample; required result is zero disclosures.
5. Incident procedure: detect, contain, assess, notify, remediate, review.
6. Tested kill switch that disables automated replies immediately without deployment; define operator, activation time, handling of in-flight tickets, and test evidence.

## 13. Repository standard

```text
README.md
requirements.txt
.env.example
src/
prompts/
tests/
evaluation/
docs/
data/                 # small samples only
.github/workflows/
```

The README must be written and tested as an external user would follow it: environment creation, dependency installation, `.env` setup, API launch, one-ticket run, unattended evaluation run, and tests. Include expected outputs, error/fallback behaviour, and an attribution declaration for generated or external code.

## 14. Report and video outline

### Report

Use this order: executive summary; problem; discovery findings; requirements; architecture and design; implementation; evaluation; governance and risk; PRD revision; conclusions; appendices. Every performance statement needs its method and data source. Include the required evaluation-caution statement.

### Video

Use this approximate flow: problem (0-2 min); discovery (2-5); system (5-7); live demonstration including response, escalation, block, and unattended-run evidence (7-14); results and limitations (14-17); governance/risk (17-18); next steps and revision lessons (18-20). Be visible for introduction and close.

Use the Hyperframes plugin for video generation and editing where it improves production quality, such as titles, transitions, captions, visual explainers, and post-production. It must not replace the required live system demonstration or fabricate evaluation evidence. Preserve the original demonstration recording, distinguish generated visuals from actual system output, and include the required declaration of AI tool use in the submission.

## 15. Final pre-submission checklist

- [ ] Clean checkout follows README exactly.
- [ ] A1-A12 all pass and evidence is retained.
- [ ] Full evaluation input completed unattended; output and decision log reconcile.
- [ ] No credentials appear in current files or Git history.
- [ ] Groq model IDs are configured, documented, and active in the account used for the benchmark.
- [ ] All prompt versions map to requirements and tests.
- [ ] Fairness, privacy, grounding, citations, and calibration are measured and reported honestly.
- [ ] PRD revision log contains a substantive evidence-led change.
- [ ] Five workbooks and effort log are genuinely completed.
- [ ] Report follows its required order and length.
- [ ] Video includes live response, escalation, guardrail block, and unattended-run evidence.
- [ ] Hyperframes-generated or edited material is disclosed, while live system evidence remains unaltered and traceable to the original recording.
- [ ] Archive has exactly the required four top-level folders and correct naming.

## 16. Current status ledger

| Work item | Current status | What remains |
| --- | --- | --- |
| Product requirements | Specified | Transfer into Stage 2 workbook when preparing submission. |
| Master execution plan | Specified | Follow it; it is not evidence of implementation. |
| Groq setup | Requires user-owned console action | Create key, add local `.env`, verify model availability. |
| Software implementation | Not started | Build and test components. |
| Evaluation and governance results | Not started | Run, measure, review, and document actual results. |
| Workbooks, report, video, effort log | Not completed | Produce progressively as evidence is available. |
| Submission archive | Not created | Package only after all deliverables are complete. |
