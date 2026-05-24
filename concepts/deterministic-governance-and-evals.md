---
title: Deterministic Governance and Evals for Agentic Systems
---

# Deterministic Governance and Evals for Agentic Systems

Agentic systems are useful because models can reason, plan, and adapt. They are risky because those same behaviors are probabilistic. Enterprise security cannot depend on the model deciding to be compliant. Compliance has to be enforced by deterministic system boundaries.

The core pattern is simple:

1. Let the model propose intent.
2. Convert intent into a typed request.
3. Evaluate that request with deterministic policy.
4. Execute only through constrained runtime boundaries.
5. Record the decision, inputs, policy version, runtime result, and evidence hash.
6. Use evals and deterministic tests to prove the control still works after every change.

## Reference Architecture

```text
User / API
   |
   v
Agent harness
   |  prompt version, model version, input set hash
   v
Foundation model
   |  probabilistic plan or proposed action
   v
Typed intent boundary
   |  ToolRequest, DataAccessRequest, ArtifactHandoff, ApprovalRequest
   v
Deterministic control plane
   |  schema validation, OPA/conftest, Kyverno, authorization, quotas, residency policy
   |------------------- deny -------------------> evidence lakehouse
   |------------------- pause ------------------> durable HITL approval
   |------------------- allow ------------------> constrained runtime
                                                   |
                                                   v
                                           OpenShell / sandbox /
                                           privileged executor
                                                   |
                                                   v
                                           filtered result / artifact
                                                   |
                                                   v
                                      A2A events + audit evidence
                                                   |
                                                   v
                                           agent final answer
```

The foundation model sits inside the harness. It does not sit inside the trusted computing base for authorization. The trusted computing base is the deterministic control plane plus the constrained runtime and evidence store.

## Where Probabilistic Reasoning Ends

The model may choose a plan, draft text, identify a tool, or propose a remediation. The model may not be the final authority for:

- Whether a tool is allowed.
- Whether a data source may be accessed.
- Whether sensitive data may leave a boundary.
- Whether a restricted action has approval.
- Whether an inter-agent message can be trusted.
- Whether a runtime has enough isolation for the requested task.
- Whether a prompt or policy change is safe to deploy.

Those decisions belong to deterministic components: policy engines, authorization services, schema validators, admission controllers, sandbox runtimes, CI gates, and evidence exporters.

## Enforcement Layers

| Layer | Probabilistic input | Deterministic enforcement | Evidence |
|---|---|---|---|
| Prompt and harness | Model behavior changes across prompts, models, and context | Golden cases, regression thresholds, schema assertions, rule-based checks | Eval run, case hash, prompt version, pass/fail result |
| Tool request | Model proposes a tool call | ToolRequest schema, allowlist, argument gates, budgets, category quotas | Tool decision, policy hash, argument hash |
| Data access | Model asks for context or retrieval | Identity, purpose, data classification, ACL, residency, retention policy | Access decision, dataset ID, requester, purpose |
| Inter-agent communication | One agent hands work to another | Typed artifacts, artifact allowlist, provenance hash, role boundary | A2A event, artifact hash, source agent, target agent |
| Runtime execution | Agent requests shell, code, browser, MCP, or network | OpenShell or sandbox policy, egress rules, filesystem scope, secret scope, resource limits | Runtime session, sandbox policy, logs, exit status |
| Human approval | Agent reaches restricted action | Durable HITL record, expiry, approver identity, request hash, resume token hash | Approval record, approval log, resumed command |
| Deployment | Team changes prompt, policy, workflow, or runtime | CI policy engines, migration tests, eval gates, Pages/export leak scanner | GHA run, policy results, eval summary, release artifact |

## Component Control Matrix

| Component | Responsibility | Deterministic guarantee | Test or eval |
|---|---|---|---|
| Agent harness | Owns prompt, context assembly, model call, and eval runner | Same input set, prompt version, and model config can be replayed | Golden set regression, schema assertion, trajectory check |
| ToolRequest contract | Converts model intent into typed action | Unknown fields, missing purpose, invalid tool, and invalid arguments fail before execution | JSON schema validation, argument-boundary tests |
| A2A governor | Makes allow, deny, or pause decisions | Every tool call passes through one enforcement point | Allow/deny/pause fixtures, policy hash replay |
| OPA/conftest | Evaluates CI and request policies | Policy decision is independent of model text | Rego unit tests, admission fixture tests |
| Kyverno or cluster admission | Enforces workload and runtime deployment policy | Non-compliant pods, images, and namespaces cannot deploy | Kyverno CLI tests, bad manifest denial |
| OpenShell or sandbox runtime | Constrains code, shell, filesystem, network, and process execution | Runtime cannot exceed declared policy even if agent asks | Egress denial test, filesystem boundary test, secret access test |
| HITL service | Handles restricted action approval | Approval is durable, expiring, identity-bound, and tied to request hash | Denied-without-approval test, expired approval test, replay mismatch test |
| Evidence lakehouse | Stores decisions, runtime events, evals, and artifacts | Auditors can reconstruct what happened without trusting the model | Evidence bundle completeness tests |
| GitHub Actions | Runs policy engines and eval gates before merge or deploy | Unsafe changes cannot enter production branch without passing gates | Required checks, migration tests, eval thresholds |

## Evals Are The Harness, Not The Control

Evals prove behavior and catch regressions. They do not replace runtime enforcement.

A useful enterprise eval stack has three tiers:

| Tier | Use | Example |
|---|---|---|
| Deterministic assertions | Hard rules with known expected outcomes | Tool name must match allowlist; JSON schema must validate; restricted tool without approval must deny |
| Programmatic scoring | Domain rules that can be calculated | Schedule violates zero hard constraints; generated policy contains required fields; evidence bundle has required rows |
| LLM or human judgment | Soft quality and ambiguous behavior | Is the answer useful, complete, and appropriately cautious? |

Use deterministic assertions wherever the rule is crisp. Use LLM-as-judge for quality, not as a synchronous security gate.

## Deterministic Test Pack

An enterprise agent should ship with a minimum deterministic test pack before any qualitative evals are considered:

| Test pack | Required cases |
|---|---|
| Schema tests | Valid request, missing field, extra field, invalid enum, malformed artifact |
| Tool admission tests | Allowed discovery tool, denied unknown tool, denied restricted tool without approval, approved restricted tool, quota exhaustion |
| Argument gate tests | Oversized query, wildcard delete, path traversal, excessive result count, invalid location or region |
| Data access tests | Allowed tenant data, denied cross-tenant access, denied purpose mismatch, denied residency mismatch |
| Runtime tests | Denied egress, denied host mount, denied secret access, CPU/memory limit, sandbox TTL expiry |
| HITL tests | Approval required, approval denied, approval expired, request hash mismatch, replay token mismatch |
| Evidence tests | Every decision has policy hash, schema hash, request hash, actor, timestamp, outcome, and artifact reference |
| Regression tests | Historical production failures and adversarial cases remain fixed |

These tests are deterministic because their expected result is known. If any of them require an LLM judge to decide whether the security control worked, the control boundary is in the wrong place.

## Prompt, Harness, Evals

The practical development loop should be:

1. Create a baseline prompt and harness.
2. Build an eval suite before changing the prompt.
3. Include control cases, historical failures, edge cases, and adversarial cases.
4. Version the prompt and record why each instruction exists.
5. Run deterministic tests before qualitative judging.
6. Promote only when regressions are understood and thresholds are met.

This matters because prompt changes can help one case while breaking another. A defensive instruction written for an older model can become an over-constraint on a newer model. A vague instruction such as "calculate accurately" does not add capability; the correct pattern is to give the model a tool and test whether it calls that tool correctly.

## Generate, Evaluate, Repair

For complex tasks, avoid one giant prompt that tries to do everything. Split the workflow:

```text
generate -> evaluate -> repair -> validate -> admit -> execute
```

The evaluation step should produce structured findings. The repair step should address those findings. The final validation step should be deterministic wherever possible.

For a regulated agent deployment, that means:

- Generate: model proposes plan or artifact.
- Evaluate: harness checks policies, schemas, threat-model requirements, and known failure cases.
- Repair: model fixes the precise violations.
- Validate: deterministic tests confirm the artifact now satisfies the contract.
- Admit: policy engine decides whether the action can proceed.
- Execute: sandbox or privileged executor runs under constrained policy.

## Worked Example: Restricted Tool Call

Scenario: an agent wants to run a SerpApi/MCP search tool against a regulated customer task.

| Step | Event | Deterministic control | Evidence |
|---|---|---|---|
| 1 | Model proposes `SEARCH` with arguments | Harness wraps proposal as `ToolRequest` | `tool_request` row with request hash |
| 2 | Schema validates request shape | Missing purpose, malformed args, or unknown artifact type fail | `validation_result` or policy decision |
| 3 | Governor classifies tool as restricted execution | Category comes from registry, not model text | `tool_decision` with policy hash |
| 4 | Policy checks budget, tenant, data class, and approval state | No approval exists, so action pauses | `approval_request` artifact |
| 5 | External approver reviews request | Approval includes approver identity, expiry, and request hash | `hitl_approval` row |
| 6 | Graph resumes with approval token | Token hash and request hash must match | `approval_log` and resumed command |
| 7 | Privileged executor calls tool under runtime constraints | Agent never receives MCP credentials | `tool_execution` and sandbox log |
| 8 | Result is filtered into allowed artifact types | Raw result is not silently leaked | `agent_artifact` with artifact hash |
| 9 | Agent receives summary and produces answer | Final answer is separated from raw tool output | A2A status and result summary |
| 10 | Evidence bundle exports run | Auditor can replay policy and reconstruct trace | Bundle manifest with policy/schema hashes |

The model chose the action. The platform authorized the action. Those are different responsibilities.

## Mapping Governance Requirements To Controls

| Governance requirement | Enforceable system control |
|---|---|
| Least privilege | Per-agent identity, tool allowlist, scoped credentials, runtime service account |
| Data minimization | Retrieval filters, field-level redaction, artifact allowlist, retention policy |
| Purpose limitation | Workload contract, approved use case, data-access purpose field |
| Residency | Provider/region allowlist, data classification, routing policy |
| Human oversight | Durable HITL approval with expiry and replay protection |
| Auditability | Policy hash, schema hash, request hash, runtime log, evidence bundle |
| Safety constraints | Tool category quotas, argument gates, deny-by-default policy |
| Change control | CI eval gate, policy engine tests, migration tests, release provenance |

## Architecture Patterns

| Pattern | What it separates | Why it matters |
|---|---|---|
| Intent-to-request boundary | Model text from system action | Policy can evaluate structured data instead of prose |
| Governor/executor split | Authorization from execution | Credentials and privileged APIs stay outside the agent |
| Registry-backed tool categories | Tool identity from naming heuristics | Execution tools cannot masquerade as discovery tools |
| Durable HITL | Approval from conversation state | High-risk decisions survive time, restarts, and audits |
| Artifact filtering | Raw tool output from agent-visible output | Sensitive data is controlled before re-entering the model context |
| Sandbox lifecycle | Agent plan from runtime capability | Shell, network, filesystem, and secrets are constrained by policy |
| Eval-before-prompt-change | Prompt iteration from subjective judgment | Teams can measure whether behavior improved or regressed |
| Evidence lakehouse | Runtime event from audit claim | Compliance can be proven from records, not model explanation |

## Failure Modes

| Failure mode | Why it fails | Deterministic fix |
|---|---|---|
| The prompt says "do not call dangerous tools" | The model can ignore, misunderstand, or be manipulated | Deny dangerous tools in ToolRequest admission |
| The model is asked to classify sensitive data by itself | Classification can drift or be bypassed | Use data labels, ACLs, and policy-bound retrieval |
| HITL approval is console input | Approval is not durable, auditable, or resumable | Store approval with request hash, approver, expiry, and resume token hash |
| Evals only test happy paths | Regressions and adversarial paths pass unnoticed | Include historical failures, edge cases, and adversarial cases |
| LLM-as-judge is used as a runtime guard | The guard is probabilistic and can be attacked | Use deterministic policy for authorization; use judge offline for quality |
| MCP discovery is treated as authorization | Discovering a tool does not make it safe | Registry-backed allowlist and category policy |
| Tool output is trusted directly | Tool can return unexpected or sensitive content | Validate, redact, summarize, and filter artifacts before handoff |

## Enterprise Promotion Rule

A prompt, tool, policy, or agent runtime change should not ship unless it has:

- Versioned input and output contracts.
- Deterministic tests for hard constraints.
- Eval coverage for known and expected user domains.
- Regression summary against the previous release.
- Admission-policy pass result.
- Evidence of runtime behavior under sandbox constraints.
- Traceable policy hash, schema hash, and artifact hash.
- Rollback or exception path with owner and expiry.

## CI Gate Shape

A production branch should require these checks for any change that touches prompts, tools, policies, runtime, evals, workflows, or evidence schemas:

```text
validate contracts
  -> run deterministic eval fixtures
  -> run OPA/conftest policy tests
  -> run Kyverno/admission tests
  -> run lakehouse migration tests
  -> run evidence bundle completeness tests
  -> publish signed/hashed release artifact
```

For regulated deployments, a green CI run is not "the model is safe." It means the deterministic controls that contain the model have been tested against known failure modes.

## Field Checklist

- Can a denied tool call be reproduced with the same policy hash?
- Can an auditor see which prompt, model, policy, and schema produced a run?
- Can a customer verify that restricted actions required durable approval?
- Can the team prove that data access followed identity, purpose, residency, and retention rules?
- Can CI block a prompt or policy change that breaks deterministic tests?
- Can runtime logs show that sandbox egress and filesystem boundaries held?
- Can eval results distinguish model quality regressions from policy/admission failures?

## Sources

- Anthropic Code w/ Claude session: [The prompting playbook](https://claude.com/code-with-claude/session/ldn-the-prompting-playbook)
- Public transcript summary: [The Prompting Playbook, Margot van Laar](https://gist.github.com/eiraho/263910a5b85f6a79540c00611035f577)
- Related eval framing: [Evals Are Not Unit Tests - Evaluating Non-Deterministic AI Systems](https://www.classcentral.com/course/youtube-evals-are-not-unit-tests-ido-pesok-vercel-v0-473894)
