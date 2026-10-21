# CloudServe Support Automation

## Final Product Requirements Document

| Document control | Value |
| --- | --- |
| Version | 2.0 Final specification |
| Status | Ready for build and evaluation |
| Product | CloudServe Support Automation |
| Primary owner | Head of Customer Support |
| Decision date | 22 October 2026 |

## 1. Executive summary

CloudServe should build a support automation service that retrieves answers from its reviewed knowledge base, responds automatically only when evidence and safety checks support that decision, and otherwise escalates the ticket with useful context. It is not a general-purpose chatbot. Its purpose is to reduce avoidable support work and waiting time without sending unsupported, unsafe, or misleading answers to customers.

The development dataset supports this focus: 357 of 500 tickets (71.4%) are answerable from existing documentation, while 311 (62.2%) are expected to be suitable for automated response. Historic handling escalated 281 tickets (56.2%). The system must therefore make documentation findable and make safe escalation valuable, rather than attempting to automate every interaction.

## 2. Problem statement

CloudServe receives more than 500 support tickets each week and takes eight to twelve hours to provide a first reply, despite a two-hour service commitment. Much of this work is repetitive and already covered by reviewed documentation, but agents cannot reliably find the relevant material; they recreate answers from memory or escalate tickets that could have been resolved. Customers wait for answers that often already exist, while tier-two engineers receive raw escalations with no summary or evidence. Automating an answer when the system is uncertain, handling a safety-sensitive request, or cannot ground its claims would worsen the problem. CloudServe needs a trustworthy triage and retrieval service that gives customers fast, cited answers only when it is safe to do so and gives human agents a clear, evidence-backed escalation otherwise.

## 3. Evidence and traceability key

| ID | Evidence | Requirement implication |
| --- | --- | --- |
| E1 | Head of Support: first reply is 8-12 hours; first-contact resolution is 42%; a wrong automated answer is unacceptable. | Safe automation, measurable outcomes, auditable decisions. |
| E2 | Tier-one agent: most questions repeat; time is spent finding and rewriting known answers; security is always escalated. | Retrieval, citations, mandatory escalation paths. |
| E3 | Tier-two engineer: around half of escalations are resolvable with confidence or findability; raw escalations lack context. | Escalation summary, sources, and uncertainty explanation. |
| E4 | Technical writer: 29 reviewed articles are accurate; keyword search fails when customer wording differs; private answer files are unreviewed. | Retrieve only from the supplied reviewed documentation corpus. |
| E5 | Customer: fast, cited, disclosed automation is acceptable; a confidently wrong answer is not; urgent deployment failures matter most. | Clear disclosure, safe grounding, urgency-aware routing. |
| E6 | Development-ticket analysis: 357/500 (71.4%) answerable from documentation; 311/500 (62.2%) expected to auto-respond; 87/500 must never auto-respond. | Retrieval-first solution with explicit safety blocks. |
| E7 | Development-ticket analysis: 120/500 (24.0%) are from non-fluent writers; channels are email 212, chat 155, documentation comments 78, forum 55. | Four-channel normalisation and fairness measurement. |
| E8 | Build Specification A1-A12 and Evaluation Framework. | Runnable, deterministic, measurable and governed system. |

## 4. Users and outcomes

| User | Need | Outcome |
| --- | --- | --- |
| Customer raising a ticket | A prompt, accurate answer or transparent hand-off. | Cited response or a clear escalation without false certainty. |
| Tier-one support agent | Less searching and repetition; a safe draft when automation cannot send. | Faster handling with a summary, relevant sources, and reason for escalation. |
| Tier-two or specialist agent | Escalations that preserve context and show what has been tried. | Less thread reconstruction and fewer repeat questions. |
| Head of Support | Lower queue pressure without higher customer risk. | Traceable results against resolution, response-time, escalation, and quality targets. |
| Technical writer | Feedback on whether failures arise from the knowledge base or from the system. | Source identifiers and retrieval evidence for each decision. |

## 5. Product scope

### In scope

1. Ingest and normalise tickets from email, live chat, documentation comments, and the community forum.
2. Classify intent and urgency with calibrated numeric confidence and alternatives considered.
3. Retrieve ranked, identifiable passages from the 29 supplied reviewed knowledge-base articles.
4. Route deterministically to automated answer, human escalation, or block.
5. Generate citations-backed responses from retrieved passages only.
6. Provide an escalation package containing summary, sources, uncertainty, and draft where appropriate.
7. Run guardrails for private data, grounding, instruction integrity, tone and commitments, and confidence.
8. Persist auditable decision records; produce an unattended-run metrics report.

### Out of scope

| Exclusion | Reason |
| --- | --- |
| Autonomous security, account-compromise, billing-dispute, contractual, refund, data-location, or roadmap decisions | These require human judgment or have defined safety/compliance risks. |
| Using private agent snippet files or historical answers as answer sources | They are unreviewed and may be outdated. |
| Updating the knowledge base automatically | Content ownership and review remain with the technical writer. |
| Customer-facing promises about refunds, service credits, timelines, or product roadmap | The automation has no authority to make commitments. |
| Replacing human support or changing service entitlements | The objective is safe workload reduction, not headcount or policy change. |

## 6. Functional requirements

| ID | Requirement | Priority | Evidence | Acceptance criteria |
| --- | --- | --- | --- | --- |
| FR-01 | The system shall accept email, chat, documentation-comment, and forum tickets and normalise each into one internal record while retaining original content and channel. It shall handle missing fields, unusual characters, and empty bodies without failing. | Must | E7, E8 | A ticket from every channel is processed into the same schema; malformed input produces a logged fallback rather than a crash. |
| FR-02 | The system shall classify every ticket with an intent, urgency, numeric confidence from 0 to 1, and ranked alternatives. It shall return a defined fallback when classification is unavailable. | Must | E1, E5, E8 | Output contains all fields for every ticket; class precision is evaluated per class and confidence is calibrated. |
| FR-03 | The system shall search only the supplied reviewed documentation corpus and return ranked passages with document and passage identifiers, relevance scores, and no result when no passage clears the relevance threshold. | Must | E2, E4, E6, E8 | Returned sources resolve to real corpus passages; irrelevant or unsupported questions return no retrieval result. |
| FR-04 | The system shall route every ticket deterministically using a documented threshold and safety rules to `auto_respond`, `escalate`, or `block`, with a human-readable reason. | Must | E1, E2, E3, E6, E8 | Repeating the same input yields the same route; the log records threshold and route reason. |
| FR-05 | The system shall never auto-respond to tickets labelled or detected as security/account compromise, billing dispute, contractual commitment, data-location/compliance, roadmap/feature request, or other no-document/unsafe requests. | Must | E2, E3, E4, E6 | Safety cases are escalated or blocked; no must-not-auto-respond ticket is sent automatically. |
| FR-06 | For an auto-response, the system shall generate a concise answer grounded only in retrieved passages; attach resolvable citations to factual claims; disclose that the response is automated; and state uncertainty rather than inventing information. | Must | E1, E4, E5, E8 | Each citation resolves to a retrieved passage that supports its claim; unsupported claims trigger a block/escalation. |
| FR-07 | For an escalation, the system shall create a package with the original ticket, classification and urgency, concise summary, retrieved sources, draft if available, and a specific statement of what prevented automated resolution. | Must | E2, E3 | An agent can inspect the package without reconstructing the thread; the route reason is present. |
| FR-08 | The system shall validate every generated response before release for private data, grounding, instruction integrity, tone/scope, prohibited commitments, and confidence floor. A failed check shall block release and create an escalation record. | Must | E1, E3, E5, E8 | A deliberately unsafe ticket is blocked; validation runs on every response and is recorded. |
| FR-09 | The system shall persist an auditable record for each automated decision, including ticket ID, time, stage, summarised input, model/version, prediction/confidence/alternatives, sources, threshold, route, reason, guardrail results, prompt version, and requirement IDs. | Must | E1, E3, E4, E8 | Decision records reconcile exactly with processed tickets and can reconstruct the decision. |
| FR-10 | The system shall expose one documented unattended evaluation command accepting input and output paths, process every ticket without manual intervention, and produce a decision log and metrics report. | Must | E8 | The full supplied set completes in one run; a differently named/unseen-schema-compatible input can be processed. |
| FR-11 | The system shall handle no retrieval result, model-provider timeout/outage, rate limiting, and malformed input without crashing or silently dropping a ticket. | Must | E8 | Each induced failure produces a logged fallback, escalation, or block and the run continues. |
| FR-12 | The system shall provide a single documented test command and a clean-checkout setup path with no committed credentials. | Must | E8 | README instructions work from a clean checkout; tests execute; credential scan finds none. |

## 7. Non-functional requirements

| ID | Category | Requirement | Verification |
| --- | --- | --- | --- |
| NFR-01 | Response time | The p95 end-to-end response latency shall be under 3 seconds for normal auto-response processing; any provider delay shall degrade to a logged escalation rather than stall the run. | Automated metrics report and induced-provider-failure test. |
| NFR-02 | Business outcome | The design target is first-contact resolution at or above 60%, median first reply below 5 minutes, customer-satisfaction proxy at or above 4.0/5, and escalation rate at or below 30%, compared with 42%, 8-12 hours, 3.2/5, and 58% baseline respectively. | Report baseline, achieved value, method, date, sample, and limitations. |
| NFR-03 | Classification quality | Intent precision shall be at least 85% and recall/precision shall be reported per class with a confusion matrix. | Held-out development evaluation or validation report. |
| NFR-04 | Grounding | Hallucination rate shall be at most 5% and citation accuracy at least 95%. | Two-assessor review of at least 50 responses; citation check against source passages. |
| NFR-05 | Privacy and security | Outbound responses shall contain zero private-data disclosures. Customer text shall not be able to alter system instructions. | Automated full-output scan, manual sample review, and adversarial-input test. |
| NFR-06 | Fairness | Resolution quality and citation accuracy shall differ by less than five percentage points across customer tier, region, and language-fluency groups; findings outside this condition block deployment pending mitigation. | Segmented fairness audit, method and sample sizes documented. |
| NFR-07 | Auditability | Every automated decision shall be reconstructable and logged; logged and processed-ticket counts shall match exactly. | Reconciliation in the metrics report. |
| NFR-08 | Availability and resilience | The system shall continue safely through no-hit retrieval, timeout/outage, rate limit, and malformed input, and shall never silently omit a ticket. | Failure-injection test and unattended-run results. |
| NFR-09 | Cost and portability | The system shall use free/open-source components or free-tier access, run on Python 3.10+, and not depend on a user-specific path or uncommitted secret. | Clean-checkout rehearsal and dependency review. |

## 8. Routing policy

Automated response is permitted only when all conditions below are true:

1. The ticket is not a mandatory-escalation or prohibited-commitment case.
2. A reviewed documentation passage exceeds the documented relevance threshold.
3. Intent confidence is calibrated and meets the documented auto-response threshold.
4. Every factual claim in the generated answer is supported by a retrieved passage.
5. Private-data, instruction-integrity, tone/scope, and commitment checks pass.

Otherwise, the ticket is escalated with its evidence package. If a validation guardrail identifies a release risk, the response is blocked and escalated. Threshold values must be selected using development/validation evidence and recorded with their precision, automation-rate, and safety trade-off; the illustrative 0.80 threshold in the brief is not a mandated value.

## 9. Success measures and reporting

| Measure | Baseline | Target | Accountable role | Reporting method |
| --- | --- | --- | --- | --- |
| First-contact resolution | 42% | >= 60% | Head of Support | Automated run report; interpret alongside escalation rate. |
| Time to first reply | 8-12 hours | Median < 5 minutes | Head of Support | Mean, median, and p95 from timestamps. |
| Customer satisfaction | 3.2/5 | >= 4.0/5 proxy | Head of Support | Clearly state proxy and its limitation. |
| Escalation rate | 58% | <= 30% | Head of Support | Escalated / total tickets. |
| Intent precision | Not established | >= 85% | Engineering owner | Per-class precision/recall and confusion matrix. |
| Hallucination rate | Not established | <= 5% | Engineering owner | Two-assessor review of >= 50 responses. |
| Citation accuracy | Not established | >= 95% | Technical writer / engineering owner | Claim-to-passage check. |
| p95 latency | Not established | < 3 seconds | Engineering owner | End-to-end automated timing. |
| Private-data disclosures | Not established | 0 | Governance owner | Full-output scan plus manual sample. |
| Cross-group variation | Not established | < 5 points | Governance owner | Segmented fairness audit. |

## 10. Assumptions, risks, and open decisions

| Assumption or risk | Consequence if false | Control or resolution |
| --- | --- | --- |
| Reviewed documentation remains current and covers routine issues. | Stale or insufficient evidence can make answers unsafe. | Retrieve only reviewed articles; log sources; escalate no-hit/low-confidence cases; route article gaps to the technical writer. |
| A threshold can separate defensible automation from unsafe automation. | Over-automation harms customers; under-automation yields little operational benefit. | Calibrate on development data and select threshold using precision, safety, and escalation trade-offs. |
| Non-fluent tickets can retrieve relevant content comparably. | Customers may receive systematically worse outcomes. | Measure by fluency; mitigate and block deployment if variation exceeds 5 points. |
| Model providers may fail or rate-limit. | An unattended run may fail or response time may degrade. | Retry/backoff where safe, cache development results, then fall back to logged escalation. |
| Generated output can be validated for grounding and privacy. | A flawed guardrail risks unsafe release. | Block on uncertainty; retain logs and sampled human review. |
| Product/team policy gives a named operator authority to disable automation. | Incidents cannot be contained quickly. | Define and test a kill switch before deployment; confirm owner and in-flight behaviour. |

## 11. Acceptance and release criteria

The build is ready for assessment only when it satisfies A1-A12 in the Build Specification: clean-checkout startup; four-channel normalisation; classified tickets with confidence; identifiable retrieval; deterministic threshold routing; resolvable citations; a blocking guardrail; complete persistent decision logging; one unattended full-set run; automatic metrics; graceful failure handling; and a one-command passing test suite.

The build is not fit for live deployment unless, in addition, governance conditions hold: zero private-data disclosures, under-five-point quality variation across customer groups, complete log coverage, calibrated confidence within five points of observed accuracy, a documented incident procedure, and a tested kill switch.

## 12. Revision record

| Version | Change | Rationale |
| --- | --- | --- |
| 1.0 | Initial PRD structure. | Stage-two baseline. |
| 2.0 | Finalised the product as evidence-grounded retrieval and triage; added mandatory escalation policy, validation guardrails, exact unattended-run requirements, decision-log schema requirements, fairness controls, and measurable acceptance criteria. | Discovery and the Build/Governance/Evaluation frameworks show that a generic chatbot would not safely address CloudServe's actual bottleneck. |
