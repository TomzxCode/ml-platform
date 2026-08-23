# Project Overview

## Purpose

ml-platform is responsible for all things machine learning: it provides a unified interface for users to do ML at commercial scale.
That includes processing data, training models, running batch inference, deploying models for online inference, and copying data from A to B.
The goal is a single surface covering anything relevant to doing machine learning at scale commercially.

## Key Stakeholders

| Stakeholder | Role | Interest |
|---|---|---|
| MLEs in companies | Primary users | Least amount of friction with the infrastructure they need to get their work done |
| ML infrastructure engineers | Platform maintainers | As little on-call load and as few support requests as possible |
| Leadership | Sponsors | MLEs unencumbered by platform concerns |

## Scope

**In scope:**
- A CLI that abstracts all ML operations (data processing, training, batch inference, online inference deployment, data transfer)

**Out of scope:**
- Server
- SDK
- DB

## Key Constraints

- None identified.
