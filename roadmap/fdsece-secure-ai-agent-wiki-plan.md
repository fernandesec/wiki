---
title: FDSecE Secure AI Agent Wiki Plan
---

# FDSecE Secure AI Agent Wiki Plan

This page defines the target content model for a deeply technical, hands-on wiki for Forward Deployed Security Engineers securing production AI agent deployments.

## Mission

Create a practical field manual for securing AI deployments from design to production and ongoing operations. The wiki must help a practitioner move from a customer problem to a governed deployment with admission controls, runtime isolation, traceability, evaluations, and auditor-ready evidence.

## Core sections

| Section | Purpose |
|---|---|
| Reference architectures | Secure AI agent patterns for LangGraph, Deep Agents, OpenShell, A2A, MCP, policy engines, and lakehouse evidence |
| Secure SDLC | Design, threat modeling, build, test, admission, deploy, runtime, incident response, and feedback loops |
| Deterministic evals and tests | Prompt harnesses, golden cases, rule-based assertions, trajectory checks, generate-evaluate-repair loops, and CI gates |
| Admission control | OPA, conftest, Kyverno, GitHub Actions gates, ToolRequest admission, exception lifecycle, and workload contracts |
| Runtime hardening | OpenShell isolation, network-deny defaults, filesystem boundaries, credential scoping, sandbox lifecycle, and privileged executor patterns |
| Tool governance | A2A governor, ToolRequest schema, MCP allowlists, budgets, argument gates, HITL, and structured artifacts |
| Evidence lakehouse | Admissions, evals, tool decisions, HITL approvals, A2A events, OpenShell logs, policy hashes, and exportable audit bundles |
| Field playbooks | Customer-facing deployment checklists for cloud, Kubernetes, model providers, identities, secrets, logging, and incident response |
| FDSecE operations | How to operate when the network cannot be trusted, how to validate controls on-site, and how to brief customers with evidence |

## Page types

| Page type | Example |
|---|---|
| Architecture | Production AI Agent Reference Architecture |
| Control | ToolRequest Admission Policy |
| Playbook | Deploy a Governed Coding Agent in a Restricted Namespace |
| Lab | Prove a Denied MCP Tool Call Produces Evidence |
| Eval | Deterministic Test Harness for ToolRequest and Runtime Policy |
| Evidence | Auditor Bundle by Workload, Run, Session, and Policy Hash |
| Incident | Failed Admission Bypass Attempt |
| Role guide | Forward Deployed Security Engineer Operating Model |

## Required indexes

- By role: Security Architect, Platform Engineer, FDSecE, AppSec, Auditor
- By phase: Design, Build, Test, Deploy, Runtime, Incident, Improve
- By control: Identity, network, sandbox, policy, tool governance, eval, evidence, exception
- By artifact: spec, skill, policy, eval, migration, script, workflow, wiki page
- By deployment target: Kubernetes, OpenShell, GitHub Actions, AWS, Azure, MCP, A2A

## Traceability model

Every published page should map guidance to evidence:

- Design decisions link to threat model and workload contract
- Build controls link to GitHub Actions policy gates
- Prompt and agent changes link to eval suite version, input set hash, deterministic assertion results, and accepted regression thresholds
- Admission controls link to OPA, Kyverno, and conftest outputs
- Runtime controls link to OpenShell logs, A2A events, tool decisions, and HITL approvals
- Compliance claims link to lakehouse records and export bundle manifests

## Content quality bar

Pages must be useful for real enterprise deployment, not only explanatory. A page is ready when it contains:

- A clear boundary between probabilistic model behavior and deterministic enforcement.
- A runnable or implementable test pattern.
- The expected evidence artifact for a customer, auditor, or incident commander.
- Failure modes showing what breaks when the model is trusted to self-enforce.
- A promotion rule explaining when a prompt, policy, tool, or runtime change can ship.

## Publishing rules

- Do not publish raw SEC540 course material.
- Use source material to create original synthesis, operational checklists, and architecture guidance.
- Cite public/open sources directly when external details are used.
- Keep private customer, credential, incident, and proprietary course data out of this repository.
- Treat this public repository as the release artifact, not the source of truth.
