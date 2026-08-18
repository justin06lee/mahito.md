---
name: mahito
description: "Run a full autonomous cybersecurity audit and hardening pass over the current project. It profiles the codebase, selects every applicable skill from the security library, fans out one subagent per selected skill to find and fix issues, verifies, then consolidates a report. Use when the user runs /mahito, or asks for a complete/full security audit, end-to-end hardening, or to 'audit and fix everything' on an app, repo, or service they own."
domain: cybersecurity
subdomain: devsecops
tags:
  - security-audit
  - orchestration
  - hardening
  - devsecops
  - subagents
version: 1.0.0
author: justin06lee
license: Apache-2.0
---

# mahito

**Mahito is the conductor.** It does not itself contain security techniques — it profiles the project, picks the right skills out of the 817-skill library, and dispatches a fleet of `mahito-auditor` subagents that each wield exactly one skill to find and fix problems. You (the main agent) never need to read any individual skill's body: every technique is loaded by the subagent that invokes it, in its own context.

Invoked by `/mahito` (optionally `/mahito <path-or-scope>` to narrow the target).

## Authorization

`/mahito` operates on the **current working project only** — the user owns it and invoked the audit, which is the authorization. The whole pass is **defensive**: find weaknesses in the user's own code and fix them. Never point any part of this at third-party or production systems you were not handed, never launch live attacks against external targets, and never weaken an existing control or add a backdoor. If the working directory does not look like the user's own project, confirm scope before proceeding.

Run end-to-end without stopping to ask — the user asked for one command. Only pause for a genuinely destructive or out-of-scope decision.

## The pipeline

### Phase 0 — Load the catalog (never read skill bodies)

Read `references/skills-catalog.jsonl` (bundled with this skill): one JSON object per line, `{name, description}`, for all 817 library skills. The `description` carries each skill's "use when…" triggers — that is all you need to match. **Do not open any skill's `SKILL.md`.** If the live repo `index.json` is reachable and newer, you may prefer it; otherwise the bundled catalog is authoritative.

### Phase 1 — Profile the target

Build a project profile by inspecting the working tree (not by guessing from names):
- Languages & package managers (manifests: `package.json`, `requirements.txt`, `go.mod`, `pom.xml`, `Cargo.toml`, `Gemfile`, …).
- App shape: web server / HTTP routes, REST/GraphQL API, CLI, library, background jobs.
- Auth & identity, session handling, secret storage, cryptography usage.
- Data stores and how they are queried.
- Infrastructure: `Dockerfile`, compose, Kubernetes manifests, Terraform/CloudFormation, cloud SDKs (AWS/Azure/GCP).
- CI/CD (`.github/workflows`, etc.), dependency posture, SBOM presence.
- Special domains: mobile, smart contracts, LLM/AI features (prompts, embeddings, MCP, agents).
- Any stated compliance target (SOC 2, PCI, HIPAA, CMMC, GDPR).

Emit a short profile so the user can see what you detected.

### Phase 2 — Select applicable skills

Using the profile and `references/selection-guide.md`, choose the **applicable** subset:
- Favor **defensive / auditing / hardening** skills; use offensive skills only as review lenses on the user's own code.
- Match each catalog entry's triggers against the profile. Skip skills whose only purpose is responding to an external compromise, reversing third-party malware, host forensics, or threat-intel collection — **unless** the project itself implements those workflows.
- Produce a selected list, grouped by security area, each with a one-line rationale. Print it. Also print what you **considered and skipped**, with reasons — never let a subset read as full coverage.

Scale sensibly: a small app matches a handful of skills; a large multi-service repo may match many. There is no need to force all 817 — applicability is the filter.

### Phase 3 — Fan out the auditor fleet

For each selected skill, spawn a **`mahito-auditor`** subagent (via the Agent/Task tool; fall back to `general-purpose` if that type is unavailable). Give each subagent:
- `SKILL` — the one skill name it must invoke.
- `SCOPE` — the specific paths/components from the profile relevant to that skill.
- `PROFILE` — the project profile.

Each subagent invokes its skill, audits its lens, applies safe defensive fixes, and returns a structured report (see `agents/mahito-auditor.md`). **Run in bounded waves** (respect the platform's parallel-subagent cap; ~6–10 at a time is a good default) rather than launching everything at once.

**Avoid write collisions.** Group the waves so skills likely to edit the same files do not run concurrently. For heavy parallel file-mutating work, prefer worktree isolation (`isolation: "worktree"`) and merge results between waves. Review each wave's reports before starting the next.

### Phase 4 — Consolidate

Collect every subagent report and produce a single audit document, **`MAHITO-AUDIT.md`**, at the project root:
- Executive summary + the project profile.
- Skills applied (and skills considered-but-skipped, with reasons).
- Findings grouped by severity (critical → info), each with `path:line`.
- Fixes applied (file + one-line what/why), deduplicated across auditors.
- Residual risks & manual follow-ups that need a human decision.
- Optional framework coverage note (MITRE ATT&CK / NIST) if useful.

### Phase 5 — Verify & hand off

- If the project has tests/build/lint wired up, run them to confirm the fixes did not break anything. Report the result honestly — if something fails, say so with the output.
- Summarize for the user: what was hardened, what still needs their decision, and where the report lives.
- **Git:** the fleet leaves edits in the working tree. Commit the consolidated result following good hygiene (a dedicated branch for a change this size, atomic commits, an annotated tag) — but only actually push or commit per the session's git rules and the user's wishes. If the working tree started dirty, checkpoint first so nothing is lost.

## Guardrails (non-negotiable)

- **Own-project, defensive only.** No live attacks on external systems; no exfiltration of code, secrets, or findings; no weakening of existing controls.
- **Fix, don't break.** Minimal, targeted, style-preserving changes. If a fix is risky or behavior-changing, record it as a manual follow-up instead of applying it.
- **No silent caps.** Whatever you skip or defer, name it and why in the report.
- **You never read skill bodies.** Selection uses only the catalog metadata; techniques live only inside the subagents that invoke them.
