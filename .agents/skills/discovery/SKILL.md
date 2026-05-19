---
name: discovery
description: Conducts feature/task discovery before any code is written. Investigates the codebase, surfaces business rules, edge cases, impacted files, and produces a structured Markdown discovery document with implementation plan, acceptance criteria, test strategy, and open questions. Use whenever the user invokes `/discovery`, asks to "do discovery on X", "analyze this task before coding", "levantar requisitos", "fazer descoberta", pastes a ticket ID and asks for analysis, or otherwise wants a thorough pre-implementation breakdown of a feature, bug, or change. Do NOT use for: writing the code itself (do discovery first, then implement separately), creating ADRs of already-made decisions (use create-adr), or quick one-off questions that don't need a structured document.
metadata:
  version: '1.0.0'
  author: pedromarchiori007@gmail.com
---

# Discovery — Feature/Task Pre-Implementation Analysis

You are conducting **discovery** for a software task. The goal is to understand the work deeply *before* writing code, surface risks early, and produce a single Markdown document that a developer (or a future Claude instance) can use to implement the feature confidently and consistently.

This skill is project-agnostic — it works in any codebase, framework, or language. Adapt the depth of investigation to what the project actually exposes (read the code, configs, READMEs, and recent commits — don't invent details that aren't there).

## When to Use This Skill

Use when:

- User explicitly types `/discovery` (the primary trigger)
- User says things like "do discovery on X", "analyze this task", "levantar requisitos", "fazer descoberta", "vamos planejar antes de codar"
- User pastes a ticket ID (e.g., `CU-86ah9du26`, `JIRA-1234`) and asks for analysis
- A non-trivial feature, refactor, or bug is about to be implemented and the user wants a written plan first

Do NOT use when:

- The user wants the code written immediately (no discovery doc needed) — implement directly
- The decision is already made and just needs to be recorded — use `create-adr`
- The request is trivial (rename a variable, fix a typo, format a file) — discovery would be wasted ceremony

## Operating Principle

The output is **only as valuable as the investigation behind it**. A discovery document built on assumptions is worse than no document — it gives false confidence. Spend the majority of effort *reading the codebase* and *asking the right clarifying questions*, not writing prose. Cite real files, real functions, real lines.

If something is unknown after reasonable investigation, write it as an **open question** — never invent.

## Language Adaptation

Write the document in the **same language the user is using**. Detect from the request:

- Portuguese request → Portuguese document
- English request → English document
- Spanish request → Spanish document

Section headers and prose follow the user's language. Keep code identifiers, file paths, framework names, and technical acronyms (API, JWT, ORM, etc.) in their original form.

## Workflow

### Step 1 — Capture the Task

If the user provided a description, ticket ID, or pasted requirements, start from that. If the input is thin (e.g., just `/discovery`), ask a focused question to gather enough scope to begin:

- *What feature/task should we do discovery on?*
- *Is there a ticket, RFC, or written description I should read?*
- *Any constraints I should know upfront (deadline, must-not-touch areas, dependencies)?*

Only ask what you genuinely need to start — don't front-load a questionnaire. Most clarifying questions are better asked *after* you've read the relevant code, because then they're specific ("I see two budget approval flows — which one applies?") instead of generic.

### Step 2 — Investigate the Codebase

Before writing anything, build a real mental model:

1. **Locate the domain**: grep for the feature's nouns/verbs (entity names, route paths, command words). Find the cluster of files where the work lives.
2. **Read the relevant files end-to-end**, not just snippets. Note patterns the codebase already uses (DTOs, service layers, repositories, framework conventions) — the implementation plan must match these.
3. **Trace the data flow**: where does input come in, how is it validated, what persists it, what reads it, what renders it.
4. **Check the edges**: tests near the feature (they encode business rules), config/env files, migrations, queued jobs, scheduled tasks, webhook handlers.
5. **Look at recent git history** in the affected area (`git log -p` on key files) — recent changes often reveal *why* the code is the way it is and what the user has been working on.

For broad explorations (more than ~3 targeted searches), prefer dispatching a sub-search agent if available, so the main context stays focused on synthesis.

### Step 3 — Surface Open Questions Early

If during investigation you hit something genuinely ambiguous that would change the implementation (not just a minor detail), surface it to the user **before** writing the full document. Examples:

- Two plausible places this could live, with different trade-offs
- Business rule that the code doesn't fully encode and isn't in any ticket
- Choice between "extend existing" vs "build new" that materially changes scope

A one-line clarification now saves a wrong-direction document later. If everything is clear enough, skip this step.

### Step 4 — Generate the Discovery Document

Save the document to a sensible location in the repo. Default:

```
docs/discovery/YYYY-MM-DD-{kebab-case-title}.md
```

If `docs/discovery/` doesn't exist, suggest creating it (or place the file at `docs/` and note the suggestion). If the user gave a ticket ID, include it in the filename: `2026-05-19-CU-86ah9du26-step-status-fix.md`.

Use the template below. **Adapt the depth** to the task — a small bug fix might collapse several sections into one paragraph; a large feature warrants the full breakdown. Don't pad sections with filler just to fill the template.

### Step 5 — Report to the User

After saving, give a short summary:

- Path to the document
- The 2–3 most important findings (e.g., "this touches X and Y; main risk is Z; one open question for you")
- Suggested next step (e.g., "answer the open questions, then I can implement", "review the plan and let me know if I should start with step 1")

## Document Template

```markdown
# Discovery: {Feature/Task Title}

- **Date**: YYYY-MM-DD
- **Ticket**: {ID and link, if any}
- **Author**: {user / Claude session}
- **Status**: Draft

## 1. Context & Problem

{2–5 sentences. What is the user/business trying to accomplish? What is broken or missing today? Why does this matter now? Avoid restating the ticket verbatim — synthesize.}

## 2. Goals & Non-Goals

**Goals** — what this work *will* deliver:
- {Goal 1}
- {Goal 2}

**Non-goals** — what this work explicitly will *not* address (to protect scope):
- {Non-goal 1}
- {Non-goal 2}

## 3. Requirements & Business Rules

Enumerated, testable statements. Tag each:

- **REQ-001** — {Functional requirement}
- **REQ-002** — {Functional requirement}
- **RULE-001** — {Business rule, e.g., "Budget can only advance to step N+1 if status is APPROVED"}
- **RULE-002** — {Business rule}
- **CON-001** — {Constraint: performance, security, compatibility, deadline}

Be specific. "Must be fast" is not a requirement; "Must respond in <300ms at p95 under current load" is.

## 4. Current State (as observed in the codebase)

Cite real files with `path:line` references. Describe what exists today that this task interacts with.

- **{Component/file}** (`path/to/file.php:42`) — {what it does today}
- **{Component/file}** (`path/to/file.php:120`) — {what it does today}
- **Relevant tests** — {list test files that cover this area, if any}
- **Recent changes** — {if `git log` shows recent activity here, note it}

## 5. Acceptance Criteria

Given-When-Then format where it fits; bullet form otherwise. These must be **objectively verifiable** — a reviewer should be able to check each one against the implementation.

- **AC-001** — Given {context}, When {action}, Then {expected outcome}
- **AC-002** — The system shall {behavior} when {condition}
- **AC-003** — {…}

Cover the happy path, the main alternate flows, and the failure/edge cases identified in section 7.

## 6. Test Plan

How will we verify this works and stays working?

- **Unit tests** — {what to cover, which classes/functions}
- **Integration tests** — {what to cover; real DB / real HTTP / real queue?}
- **End-to-end / manual** — {scenarios a human should click through, if applicable}
- **Regression risk** — {what existing behavior could this break, and how we'll detect it}
- **Data setup** — {fixtures, seeds, or specific records needed for testing}

If the project has a test framework already in use (PHPUnit, Pest, Jest, pytest, etc.), name it and match its conventions.

## 7. Edge Cases & Risks

Be honest and specific. A vague "handle errors" is not useful; "what happens if the user clicks Approve twice in 200ms" is.

- **Edge case** — {situation} → {expected behavior}
- **Edge case** — {situation} → {expected behavior}
- **Risk** — {technical or business risk} → {mitigation or fallback}
- **Dependency** — {external system / team / library this relies on} → {what happens if it's unavailable}

## 8. Implementation Plan

Ordered, concrete steps. Each step should be small enough that finishing it is unambiguous. Reference real files and existing patterns.

1. **{Step title}** — {what changes, in which file(s), following which existing pattern}
2. **{Step title}** — {…}
3. **{Step title}** — {…}
4. **Tests** — {add/update tests as per section 6}
5. **Documentation / migration / cleanup** — {if applicable}

Note any **ordering constraints** (e.g., "step 2 requires the migration in step 1 to have run") and any **steps that can be parallelized**.

## 9. Open Questions

Things that need a human decision before (or during) implementation. Each question should be answerable — vague questions don't get answered.

- **Q1** — {Specific question}. Context: {why it matters}. Options: {A vs B vs …}.
- **Q2** — {Specific question}. Context: {…}.

If there are no open questions, write "*None — ready to implement.*"

## 10. References

- {Ticket link}
- {Related ADRs, RFCs, prior discovery docs}
- {Relevant external docs — framework guides, RFCs, API specs}
- {Files of particular interest: `path/to/file.ext`}
```

## Adapting the Template

Not every task needs every section. Use judgment:

- **Bug fix** — sections 1, 3 (single rule), 4 (current buggy behavior), 5, 8 may suffice. Sections 2 and 6 can be a single line each.
- **Small feature** — full template but each section may be a few bullets.
- **Large feature / refactor** — every section, often with sub-bullets and multiple paragraphs. Consider whether an ADR or RFC should accompany this discovery.
- **Spike / investigation** — sections 1, 4, 7, 9 dominate; sections 5, 6, 8 may be deferred until a follow-up discovery once the spike's findings are in.

## Quality Checklist

Before reporting completion, verify:

- [ ] Every file/function reference is real and the path resolves
- [ ] Requirements are specific and testable, not aspirational
- [ ] Acceptance criteria match the requirements (no orphan requirements, no orphan ACs)
- [ ] Edge cases include at least one failure mode, not just happy-path variants
- [ ] Implementation plan steps are ordered and each is concretely scoped
- [ ] Open questions are real questions, not rhetorical
- [ ] Document is in the user's language
- [ ] File is saved at a sensible path and the user has been told where

## Anti-Patterns to Avoid

### Inventing details

**BAD**: Writing "The `BudgetService` class handles approval" without verifying the class exists.

**GOOD**: Grepping for `BudgetService`, reading it, and citing `app/Services/BudgetService.php:34`. If you can't find it, write that as an open question, not as fact.

### Restating the ticket

**BAD**: A "Context" section that copies the ticket text verbatim.

**GOOD**: Synthesizing the *why* in the developer's terms — what problem the user feels, what the business is trying to enable, what's missing today.

### Vague requirements

**BAD**: `REQ-001 — Improve performance.`

**GOOD**: `REQ-001 — Reduce p95 latency of POST /budgets/{id}/approve from current ~1.2s to <400ms under existing load (measured via logs in cloudwatch:prod-api).`

### Implementation plan without files

**BAD**: `1. Add validation. 2. Update the model. 3. Test it.`

**GOOD**: `1. Add request validator in app/Http/Requests/ApproveBudgetRequest.php (follows pattern in CreateBudgetRequest.php). 2. Extend Budget::approve() in app/Models/Budget.php:88 to call the new validation guard. 3. Add tests in tests/Feature/BudgetApprovalTest.php.`

### No open questions on a complex task

If a complex feature truly has zero open questions, you probably didn't dig hard enough. Re-read section 7 (edge cases) — usually at least one becomes a question once you sit with it.

### Premature implementation

The output of discovery is **the document**, not the code. Don't start implementing inside the discovery step — that's a separate request from the user after they've reviewed the doc.

## Example Trigger Prompts

### Portuguese
- `/discovery`
- "Faz um /discovery dessa task CU-86ah9du26"
- "Antes de codar isso, levanta os requisitos pra mim"
- "Quero uma análise estruturada antes de implementar"

### English
- `/discovery for the new export-to-CSV feature`
- "Do discovery on this ticket: TICKET-123"
- "Analyze this task before coding — I want a plan first"
- "Run a pre-implementation breakdown on the password-reset flow"

### Spanish
- "Haz un /discovery sobre esta tarea"
- "Análisis previo antes de implementar"
