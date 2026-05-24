---
title: Fernandesec Wiki
---

# Fernandesec Wiki

This site is the public, synthesized knowledge layer for secure AI platform engineering. It is designed for hands-on security architects, platform engineers, and Forward Deployed Security Engineers securing high-stakes AI deployments in regulated environments.

The private source workspace contains the raw SEC540-derived corpus, specs, skills, policies, evals, and lakehouse evidence tooling. This public wiki publishes curated guidance only.

## Start here

- [FDSecE secure AI agent wiki plan](./roadmap/fdsece-secure-ai-agent-wiki-plan.md)
- [Deterministic governance and evals for agentic systems](./concepts/deterministic-governance-and-evals.md)

## Publishing model

- Public repository: `fernandesec/wiki`
- Public site: `https://fernandesec.github.io/wiki/`
- Private source: SEC540 platform workspace and raw course notes
- Release rule: publish synthesized architecture, controls, labs, and evidence patterns; never publish raw proprietary training material

## Target users

- Security architects defining secure-by-default AI platforms
- Platform engineering leaders building governed runtime foundations
- Forward Deployed Security Engineers working directly with customers
- AppSec and cloud security engineers validating agent deployments
- Auditors who need evidence of design, admission, runtime, and incident controls

## Operating thesis

Foundation models can reason probabilistically, but enterprise actions must be authorized deterministically. The platform must convert model intent into typed requests, policy decisions, runtime constraints, eval results, and evidence records that can be replayed without trusting the model's explanation.
