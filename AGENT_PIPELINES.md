# Agent Pipelines Documentation - Spec Kit

This document describes all possible agent workflows and pipelines in the Spec Kit system. Each pipeline shows which prompts call which, what data flows between them, and when to use each workflow.

**Last Updated:** 2025-11-11

---

## Table of Contents

1. [Pipeline Overview](#pipeline-overview)
2. [Main Feature Development Pipeline](#1-main-feature-development-pipeline) ✓
3. [Quick Feature Pipeline](#2-quick-feature-pipeline) _(Coming next)_
4. [Specification Refinement Pipeline](#3-specification-refinement-pipeline) _(Coming next)_
5. [Quality Validation Pipelines](#4-quality-validation-pipelines) _(Coming next)_
6. [Constitution Update Pipeline](#5-constitution-update-pipeline) _(Coming next)_
7. [Research & Planning Pipeline](#6-research--planning-pipeline) _(Coming next)_

---

## Pipeline Overview

The Spec Kit implements a multi-phase, AI-powered development workflow with 8 core prompts that can be composed into different pipelines based on your needs:

- **constitution**: Establish project governance principles
- **specify**: Create feature specifications from natural language
- **clarify**: Resolve specification ambiguities interactively
- **plan**: Generate technical design and contracts
- **tasks**: Create dependency-ordered implementation tasks
- **checklist**: Validate requirement quality ("unit tests for English")
- **analyze**: Check cross-artifact consistency
- **implement**: Execute implementation tasks

**Key Concepts:**
- **Sequential Pipelines**: Prompts that must run in order (e.g., specify → plan → tasks)
- **Optional Steps**: Prompts that can be skipped (clarify, analyze, checklist)
- **Parallel Branches**: Multiple checklists can be created independently
- **Data Flow**: Each prompt produces artifacts consumed by downstream prompts
- **Quality Gates**: Validation checkpoints that can block progression

---

## 1. Main Feature Development Pipeline

**Description:** The complete, recommended workflow for production features with all quality gates and validation steps.

**When to Use:**
- Production features requiring high quality
- Features with complex requirements or multiple stakeholders
- When you want maximum validation and traceability
- Team environments with review processes

**Pipeline Stages:** 7 stages (5 required + 2 optional)

### ASCII Workflow Diagram

```
┌─────────────────┐
│  constitution   │ (Optional: Run once per project)
│  /speckit.      │
│  constitution   │
└────────┬────────┘
         │
         │ Establishes: Project principles, governance rules
         │ Outputs: /memory/constitution.md
         │
         ▼
┌─────────────────┐
│  1. SPECIFY     │ *** REQUIRED ***
│  /speckit.      │
│  specify        │
│  "[description]"│
└────────┬────────┘
         │
         │ Inputs: Feature description (natural language)
         │ Reads: constitution.md (for validation)
         │ Outputs: - specs/[N]-[name]/spec.md
         │          - specs/[N]-[name]/checklists/requirements.md
         │          - New branch: [N]-[feature-name]
         │ Data: User scenarios, functional requirements,
         │       acceptance criteria, success metrics,
         │       key entities, assumptions
         │
         ▼
┌─────────────────┐
│  2. CLARIFY     │ (Optional but Recommended)
│  /speckit.      │
│  clarify        │
└────────┬────────┘
         │
         │ Inputs: Existing spec.md
         │ Interactive: Up to 5 targeted questions with AI recommendations
         │ Reads: spec.md (ambiguity scanning across 10+ categories)
         │ Updates: spec.md (adds Clarifications section, updates requirements)
         │ Data: - Functional scope clarifications
         │       - Data model decisions
         │       - UX flow specifications
         │       - Non-functional requirement targets
         │       - Integration assumptions
         │       - Edge case handling
         │
         ▼
┌─────────────────┐
│  3. PLAN        │ *** REQUIRED ***
│  /speckit.plan  │
└────────┬────────┘
         │
         │ Inputs: spec.md, constitution.md
         │ Reads: - spec.md (requirements, entities, user stories)
         │        - constitution.md (validation gates)
         │ Phase 0: Research unknowns → research.md
         │ Phase 1: Design & contracts → data-model.md, contracts/*, quickstart.md
         │ Agent Context Update: Updates .claude/, .gemini/, etc. with tech stack
         │ Outputs: - plan.md (tech stack, architecture)
         │          - research.md (decisions, rationale, alternatives)
         │          - data-model.md (entities, relationships, validation)
         │          - contracts/ (OpenAPI/GraphQL schemas)
         │          - quickstart.md (integration scenarios)
         │          - [agent]/context.md (updated with new technology)
         │ Data: - Technical context (language, frameworks, tools)
         │       - Architecture decisions
         │       - Entity definitions with relationships
         │       - API contracts per user action
         │       - Integration patterns
         │
         ▼
┌─────────────────┐
│  4. TASKS       │ *** REQUIRED ***
│  /speckit.tasks │
└────────┬────────┘
         │
         │ Inputs: plan.md, spec.md
         │ Reads: - spec.md (user stories with P1/P2/P3 priorities)
         │        - plan.md (tech stack, structure)
         │        - data-model.md (entities) [if exists]
         │        - contracts/ (endpoints) [if exists]
         │        - research.md (decisions) [if exists]
         │ Outputs: tasks.md (dependency-ordered task list)
         │ Data: - Task phases (Setup, Foundational, User Stories, Polish)
         │       - Task IDs (T001, T002, ...)
         │       - Parallel markers [P]
         │       - Story labels [US1], [US2], [US3]
         │       - File paths for each task
         │       - Dependency graph
         │       - Parallel execution opportunities
         │
         ▼
┌─────────────────┐
│  5. ANALYZE     │ (Optional but Recommended)
│  /speckit.      │
│  analyze        │
└────────┬────────┘
         │
         │ Inputs: spec.md, plan.md, tasks.md, constitution.md
         │ Reads: All artifacts (read-only analysis)
         │ Detection Passes:
         │   - Duplication (near-duplicate requirements)
         │   - Ambiguity (vague terms, placeholders)
         │   - Underspecification (missing details)
         │   - Constitution Alignment (MUST violations = CRITICAL)
         │   - Coverage Gaps (requirements without tasks)
         │   - Inconsistency (terminology drift, conflicts)
         │ Outputs: Analysis report (console output, not file)
         │ Data: - Findings table (ID, Category, Severity, Location, Recommendation)
         │       - Coverage summary (requirements → tasks mapping)
         │       - Constitution alignment issues
         │       - Unmapped tasks
         │       - Metrics (total reqs, tasks, coverage %, issue counts)
         │       - Next actions
         │
         ▼
┌─────────────────┐
│ 6. CHECKLIST(S) │ (Optional: Can create multiple)
│  /speckit.      │
│  checklist      │
│  "[domain]"     │
└────────┬────────┘
         │
         │ Inputs: spec.md, plan.md, tasks.md, domain context
         │ Interactive: 3-5 clarifying questions about focus areas
         │ Reads: Relevant portions of spec/plan/tasks based on domain
         │ Can be run multiple times for different domains:
         │   - /speckit.checklist "UX requirements" → checklists/ux.md
         │   - /speckit.checklist "API design" → checklists/api.md
         │   - /speckit.checklist "security" → checklists/security.md
         │ Outputs: checklists/[domain].md (e.g., ux.md, api.md, security.md)
         │ Data: - Requirement quality checks (CHK001, CHK002, ...)
         │       - Traceability markers [Spec §X.Y], [Gap], [Ambiguity]
         │       - Quality dimensions (Completeness, Clarity, Consistency, etc.)
         │       - Domain-specific validation items
         │
         ▼
┌─────────────────┐
│ 7. IMPLEMENT    │ *** REQUIRED ***
│  /speckit.      │
│  implement      │
└────────┬────────┘
         │
         │ Inputs: All artifacts (tasks.md, plan.md, spec.md, etc.)
         │ Checklist Gate: Validates all checklists are complete
         │   - If incomplete: Prompts user to proceed or wait
         │   - If complete: Proceeds automatically
         │ Reads: - tasks.md (execution plan)
         │        - plan.md (tech stack, architecture)
         │        - data-model.md, contracts/, research.md, quickstart.md [if exist]
         │        - checklists/* (validation status)
         │ Project Setup: Auto-creates ignore files based on tech stack
         │   - .gitignore, .dockerignore, .eslintignore, etc.
         │ Execution: Phase-by-phase task execution
         │   - Respects dependencies (sequential vs parallel [P])
         │   - Marks tasks complete [X] in tasks.md
         │ Outputs: - Implemented features
         │          - Updated tasks.md (tasks marked [X])
         │          - Created/updated ignore files
         │ Data: Working implementation per specification
         │
         ▼
    ┌─────────┐
    │ SUCCESS │
    │ Feature │
    │ Complete│
    └─────────┘
```

### Data Flow Summary

```
Stage 1 (SPECIFY):
  IN:  Feature description (user input)
       constitution.md (validation)
  OUT: spec.md → [functional requirements, user scenarios, acceptance criteria,
                  success metrics, entities, assumptions]
       checklists/requirements.md → [spec quality checklist]

Stage 2 (CLARIFY):
  IN:  spec.md
  OUT: spec.md (updated) → [+ Clarifications section, + resolved ambiguities]

Stage 3 (PLAN):
  IN:  spec.md (requirements)
       constitution.md (gates)
  OUT: plan.md → [tech stack, architecture, file structure]
       research.md → [decisions, rationale, alternatives]
       data-model.md → [entities, fields, relationships, validation]
       contracts/ → [API schemas per endpoint]
       quickstart.md → [integration scenarios]
       [agent]/context.md → [updated tech stack]

Stage 4 (TASKS):
  IN:  plan.md (tech details)
       spec.md (user stories + priorities)
       data-model.md, contracts/, research.md (optional)
  OUT: tasks.md → [task phases, IDs, parallel markers, story labels,
                   file paths, dependencies]

Stage 5 (ANALYZE):
  IN:  spec.md, plan.md, tasks.md, constitution.md
  OUT: console report → [findings, coverage map, metrics, next actions]

Stage 6 (CHECKLIST):
  IN:  spec.md, plan.md, tasks.md (domain-filtered)
  OUT: checklists/[domain].md → [requirement quality checks, traceability]

Stage 7 (IMPLEMENT):
  IN:  tasks.md (execution plan)
       plan.md, spec.md, data-model.md, contracts/, research.md
       checklists/* (gate validation)
  OUT: implemented features
       tasks.md (updated with [X])
       ignore files (.gitignore, etc.)
```

### Critical Decision Points

1. **After SPECIFY**: Do requirements need clarification?
   - **YES** → Run CLARIFY (reduces downstream rework)
   - **NO** → Skip to PLAN

2. **After TASKS**: Validate consistency before implementation?
   - **Recommended** → Run ANALYZE (catches issues early)
   - **Skip** → Proceed to CHECKLIST or IMPLEMENT

3. **Before IMPLEMENT**: Need domain-specific validation?
   - **YES** → Run CHECKLIST for each domain (UX, API, security, etc.)
   - **NO** → Proceed to IMPLEMENT

4. **At IMPLEMENT Gate**: Checklists incomplete?
   - **User Decision** → Proceed anyway or fix issues first

### Example Session

```bash
# 1. Establish project constitution (once per project)
/speckit.constitution "Data privacy, Security first, API-first design"

# 2. Start new feature
/speckit.specify "Add user authentication with OAuth2 and JWT tokens"
# → Creates specs/1-user-auth/spec.md
# → Creates specs/1-user-auth/checklists/requirements.md
# → Asks 0-3 clarification questions if needed

# 3. Resolve ambiguities (optional but recommended)
/speckit.clarify
# → Asks up to 5 targeted questions with AI recommendations
# → Updates spec.md incrementally after each answer

# 4. Generate implementation plan
/speckit.plan
# → Creates plan.md, research.md, data-model.md, contracts/, quickstart.md
# → Updates .claude/context.md with new tech stack

# 5. Generate task list
/speckit.tasks
# → Creates tasks.md with phases organized by user story

# 6. Validate consistency (optional)
/speckit.analyze
# → Shows findings table, coverage map, metrics

# 7. Create domain checklists (optional, can run multiple)
/speckit.checklist "security requirements"  # → checklists/security.md
/speckit.checklist "API design"             # → checklists/api.md

# 8. Execute implementation
/speckit.implement
# → Validates checklists (if any exist)
# → Creates .gitignore, .dockerignore, etc.
# → Executes tasks phase-by-phase
# → Marks tasks complete with [X]
```

### Success Criteria

- ✅ All required artifacts created (spec, plan, tasks)
- ✅ Constitution gates passed
- ✅ Requirements mapped to tasks (>80% coverage)
- ✅ All checklists complete (if created)
- ✅ Implementation matches specification
- ✅ Tests pass (if tests were requested)

---

## 2. Quick Feature Pipeline

**Description:** Streamlined workflow for simple features, prototypes, or when speed is prioritized over extensive validation.

**When to Use:**
- Simple, well-understood features
- Prototypes and MVPs
- Personal projects or exploratory work
- Features with low risk or limited scope
- When requirements are already clear

**Pipeline Stages:** 4 stages (all required, no optional steps)

### ASCII Workflow Diagram

```
┌─────────────────┐
│  1. SPECIFY     │ *** REQUIRED ***
│  /speckit.      │
│  specify        │
└────────┬────────┘
         │ Creates: spec.md, checklists/requirements.md
         │ Branch: [N]-[feature-name]
         │
         ▼
┌─────────────────┐
│  2. PLAN        │ *** REQUIRED ***
│  /speckit.plan  │
└────────┬────────┘
         │ Creates: plan.md, research.md, data-model.md
         │          contracts/, quickstart.md
         │
         ▼
┌─────────────────┐
│  3. TASKS       │ *** REQUIRED ***
│  /speckit.tasks │
└────────┬────────┘
         │ Creates: tasks.md (dependency-ordered)
         │
         ▼
┌─────────────────┐
│ 4. IMPLEMENT    │ *** REQUIRED ***
│  /speckit.      │
│  implement      │
└────────┬────────┘
         │ Implements: All tasks in tasks.md
         │
         ▼
    ┌─────────┐
    │ SUCCESS │
    └─────────┘
```

### Skipped Steps

**Not included (but can be added later if needed):**
- ❌ CLARIFY - Assumes requirements are clear from specify step
- ❌ ANALYZE - No consistency validation
- ❌ CHECKLIST - No domain-specific requirement validation

### Example Session

```bash
# Quick 4-step workflow
/speckit.specify "Add export to CSV functionality"
/speckit.plan
/speckit.tasks
/speckit.implement
```

### Trade-offs

**Advantages:**
- ✅ Faster time to implementation
- ✅ Fewer interactive prompts
- ✅ Simpler workflow

**Risks:**
- ⚠️ May miss ambiguities in requirements
- ⚠️ No cross-artifact consistency checks
- ⚠️ Limited quality validation
- ⚠️ Higher rework risk if requirements misunderstood

### When to Upgrade to Full Pipeline

If during implementation you discover:
- Requirements were ambiguous → Go back and run `/speckit.clarify`
- Inconsistencies between artifacts → Run `/speckit.analyze`
- Need domain validation → Run `/speckit.checklist [domain]`

---

## 3. Specification Refinement Pipeline

**Description:** Iterative workflow focused on perfecting requirements before planning begins.

**When to Use:**
- Complex or ambiguous requirements
- Features with multiple stakeholders requiring alignment
- High-risk features needing careful specification
- When requirements quality is critical

**Pipeline Stages:** Iterative specify → clarify loop

### ASCII Workflow Diagram

```
     ┌─────────────────┐
     │    SPECIFY      │
     │  /speckit.      │
     │  specify        │
     └────────┬────────┘
              │ Creates: spec.md (initial)
              │
              ▼
     ┌─────────────────┐
  ┌─►│    CLARIFY      │
  │  │  /speckit.      │
  │  │  clarify        │
  │  └────────┬────────┘
  │           │ Updates: spec.md
  │           │ Asks: Up to 5 questions
  │           │
  │           ▼
  │  ┌──────────────────┐
  │  │  Review Updated  │
  │  │  Specification   │
  │  └────────┬─────────┘
  │           │
  │           ├─── More ambiguities? ───┐
  │           │                          │
  │           │ YES                      │ NO
  │           │                          │
  └───────────┘                          ▼
                              ┌──────────────────┐
                              │ Specification    │
                              │ Ready for        │
                              │ Planning         │
                              └────────┬─────────┘
                                       │
                                       ▼
                              Continue to PLAN stage
```

### Data Flow

```
Iteration 1:
  SPECIFY → spec.md (v1) → CLARIFY → spec.md (v1.1)

Iteration 2:
  spec.md (v1.1) → CLARIFY → spec.md (v1.2)

Iteration N:
  spec.md (v1.N-1) → CLARIFY → spec.md (v1.N) [Ready for PLAN]
```

### Example Session

```bash
# Initial specification
/speckit.specify "Complex authentication system with multiple providers"
# → spec.md created with some [NEEDS CLARIFICATION] markers

# First clarification pass
/speckit.clarify
# → Resolves 5 questions about OAuth providers, token storage, session management
# → Updates spec.md

# Review and second clarification pass (if needed)
/speckit.clarify "Focus on security and compliance"
# → Resolves 5 more questions about data protection, audit logging, MFA
# → Updates spec.md

# Continue when satisfied
/speckit.plan
# ...rest of pipeline
```

### Exit Criteria

Specification is ready for planning when:
- ✅ All [NEEDS CLARIFICATION] markers resolved
- ✅ Ambiguity scan finds no critical gaps
- ✅ Functional and non-functional requirements complete
- ✅ Edge cases identified
- ✅ Dependencies and assumptions documented

---

_Specification Refinement Pipeline complete. Quality Validation Pipelines coming next..._
