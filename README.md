# SPM Agent Skills Registry

This repository serves as the centralized registry for AI agent skills used across the Site Package Manager (SPM) ecosystem. Storing agent instructions, roles, and validation protocols here ensures consistency, reusability, and easy updates across all ecosystem repositories.

## Registry Contents

The following agent skills are currently defined:

### 1. [`qa-doc-audit`](qa-doc-audit/SKILL.md)
- **Role:** Senior Documentation QA Architect & Parallel Subagent Controller
- **Purpose:** Audits documentation coverage, API contract consistency, and Veneer Spec compilation success across modernization environments. Performs static analysis and parses `.vnr` source files into manifests using `spm compile`.

### 2. [`qa-runtime`](qa-runtime/SKILL.md)
- **Role:** Senior Runtime QA Architect & Parallel Subagent Controller
- **Purpose:** Conducts empirical, browser-less validation of compiled manifests against live HTML snapshots, verifying CSS selector matching, data extraction pipelines, and DOM modifications.

### 3. [`component-developer`](component-developer/SKILL.md)
- **Role:** Senior Frontend & Component Architect
- **Purpose:** Provides strict development guidelines for building React components (`spm-components`) that are Shadow DOM compatible, highly configurable, responsive, and robust against broken page states.

---

## How to Integrate and Use Skills

When configuring development workspaces or prompting AI agents, point them to this repository to load the latest instructions and rules. 

### Local Reference
Agents can directly reference the `.md` skill files via local paths when working on the respective projects (e.g. referencing `component-developer/SKILL.md` when building UI elements).

### Updates Protocol
To update agent behaviors:
1. Make changes to the respective `SKILL.md` in this repository.
2. Commit and push the updates.
3. Reference the updated commit hash or main branch in other pipelines/projects.
