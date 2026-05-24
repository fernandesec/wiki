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

## Evals Are The Harness, Not The Control

Evals prove behavior and catch regressions. They do not replace runtime enforcement.

A useful enterprise eval stack has three tiers:

| Tier | Use | Example |
|---|---|---|
| Deterministic assertions | Hard rules with known expected outcomes | Tool name must match allowlist; JSON schema must validate; restricted tool without approval must deny |
| Programmatic scoring | Domain rules that can be calculated | Schedule violates zero hard constraints; generated policy contains required fields; evidence bundle has required rows |
| LLM or human judgment | Soft quality and ambiguous behavior | Is the answer useful, complete, and appropriately cautious? |

Use deterministic assertions wherever the rule is crisp. Use LLM-as-judge for quality, not as a synchronous security gate.

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
