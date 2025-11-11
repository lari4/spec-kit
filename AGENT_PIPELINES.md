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

## 4. Quality Validation Pipelines

**Description:** Parallel validation workflows that can be inserted at various stages to ensure requirement quality and cross-artifact consistency.

**When to Use:**
- When you need domain-specific validation (UX, API, security, performance)
- Before major milestones or reviews
- When you want "unit tests" for your requirements
- To catch inconsistencies early

### 4.1 Requirements Quality Validation (Checklist)

**Pipeline:** Any stage → CHECKLIST (can be run multiple times in parallel)

```
┌──────────────┐
│   SPECIFY    │
│   (Done)     │
└──────┬───────┘
       │
       ├──────────┐
       │          │
       ▼          ▼
  ┌─────────┐  ┌─────────┐
  │CHECKLIST│  │CHECKLIST│  (Can create multiple in parallel)
  │  "UX"   │  │  "API"  │
  └────┬────┘  └────┬────┘
       │            │
       ▼            ▼
checklists/    checklists/
  ux.md          api.md
```

**Example: Multi-Domain Validation**

```bash
# After specify, validate different domains in parallel
/speckit.specify "E-commerce checkout flow"

# Create domain-specific checklists (can run in any order)
/speckit.checklist "UX requirements"          # → checklists/ux.md
/speckit.checklist "API design"               # → checklists/api.md
/speckit.checklist "security and compliance"  # → checklists/security.md
/speckit.checklist "performance targets"      # → checklists/performance.md

# Continue with planning once checklists complete
/speckit.plan
```

**Data Flow:**

```
Input: spec.md, plan.md (if exists), tasks.md (if exists)
       + Domain context from command argument

Interactive: 3-5 clarifying questions about:
             - Scope refinement
             - Risk prioritization
             - Depth calibration
             - Audience framing
             - Boundary exclusion

Output: checklists/[domain].md
        - CHK001, CHK002, ... (sequential IDs)
        - Quality dimensions: [Completeness], [Clarity], [Consistency],
          [Measurability], [Coverage], [Edge Cases]
        - Traceability: [Spec §X.Y], [Gap], [Ambiguity], [Conflict]
```

**Checklist Categories:**

Common checklist domains:
- **ux.md**: Visual hierarchy, interaction states, accessibility, responsive design
- **api.md**: Error formats, rate limiting, authentication, versioning, retry logic
- **security.md**: AuthN/AuthZ, data protection, threat model, compliance, breach response
- **performance.md**: Latency targets, throughput, scalability, degradation under load
- **data.md**: Entity completeness, relationships, constraints, validation, lifecycle
- **integration.md**: External dependencies, failure modes, data formats, protocols

### 4.2 Cross-Artifact Consistency Validation (Analyze)

**Pipeline:** After TASKS → ANALYZE

```
┌──────────────┐
│    TASKS     │
│   (Done)     │
└──────┬───────┘
       │ Creates: tasks.md
       │
       ▼
┌──────────────┐
│   ANALYZE    │ (Read-only analysis)
└──────┬───────┘
       │
       │ Reads: spec.md, plan.md, tasks.md, constitution.md
       │
       ▼
┌──────────────────────────┐
│ Console Report           │
│ ─────────────────────    │
│ Findings Table           │
│ Coverage Summary         │
│ Constitution Issues      │
│ Metrics & Next Actions   │
└──────────────────────────┘
```

**Detection Passes:**

```
1. Duplication Detection
   spec.md → Find near-duplicate requirements

2. Ambiguity Detection
   spec.md, plan.md → Flag vague terms (fast, scalable, robust)
                    → Flag placeholders (TODO, ???, <placeholder>)

3. Underspecification
   spec.md → Requirements missing measurable outcome
   tasks.md → Tasks referencing undefined components

4. Constitution Alignment
   spec.md, plan.md, tasks.md → Check against constitution.md MUST rules
                               → Violations = CRITICAL severity

5. Coverage Gaps
   spec.md → tasks.md mapping → Requirements without tasks
   tasks.md → spec.md mapping → Tasks without requirements

6. Inconsistency
   All artifacts → Terminology drift
                → Conflicting requirements
                → Task ordering contradictions
```

**Example Session:**

```bash
# After generating tasks
/speckit.tasks

# Validate consistency before implementation
/speckit.analyze

# Sample output:
# ┌────┬──────────────┬──────────┬─────────────┬─────────────┬────────────────┐
# │ ID │ Category     │ Severity │ Location    │ Summary     │ Recommendation │
# ├────┼──────────────┼──────────┼─────────────┼─────────────┼────────────────┤
# │ A1 │ Duplication  │ HIGH     │ spec.md:L12 │ Two similar │ Merge phrasing │
# │ D1 │ Constitution │ CRITICAL │ plan.md:L45 │ Violates... │ Add observ...  │
# │ E1 │ Coverage     │ HIGH     │ FR-5        │ No task for │ Add task T024  │
# └────┴──────────────┴──────────┴─────────────┴─────────────┴────────────────┘

# If CRITICAL issues exist, fix before implementing
# If only MEDIUM/LOW, can proceed with caution
```

**Severity Levels:**

- **CRITICAL**: Constitution MUST violations, missing core artifact, requirement with zero coverage blocking baseline functionality
- **HIGH**: Duplicate/conflicting requirement, ambiguous security/performance, untestable acceptance criterion
- **MEDIUM**: Terminology drift, missing NFR task coverage, underspecified edge case
- **LOW**: Style/wording improvements, minor redundancy

---

## 5. Constitution Update Pipeline

**Description:** Workflow for establishing or updating project governance principles with automated template propagation.

**When to Use:**
- At project initialization (run once)
- When adding/modifying core development principles
- When changing governance or amendment procedures
- Before starting new features to ensure alignment

### ASCII Workflow Diagram

```
┌───────────────────────┐
│   CONSTITUTION        │
│   /speckit.           │
│   constitution        │
│   "[principles]"      │
└──────────┬────────────┘
           │
           │ Reads: /memory/constitution.md (template with placeholders)
           │        [PROJECT_NAME], [PRINCIPLE_1_NAME], etc.
           │
           ▼
┌───────────────────────┐
│  Collect Values       │
│  - From user input    │
│  - From repo context  │
│  - Infer defaults     │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Semantic Versioning  │
│  - MAJOR: Breaking    │
│  - MINOR: New prin.   │
│  - PATCH: Clarify     │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Propagate Changes    │
│  Check & Update:      │
│  - plan-template.md   │
│  - spec-template.md   │
│  - tasks-template.md  │
│  - commands/*.md      │
│  - README.md          │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Generate Report      │
│  - Version bump       │
│  - Modified principles│
│  - Template status    │
│  - Follow-up TODOs    │
└──────────┬────────────┘
           │
           ▼
┌───────────────────────┐
│  Write Constitution   │
│  /memory/             │
│  constitution.md      │
│  (with sync report    │
│   as HTML comment)    │
└───────────────────────┘
```

### Data Flow

```
Input:
  - /memory/constitution.md (template)
  - User input (principles, values)
  - Repo context (README, docs)

Processing:
  1. Identify placeholders: [PROJECT_NAME], [PRINCIPLE_N_NAME], etc.
  2. Fill placeholders with concrete values
  3. Determine version bump (MAJOR/MINOR/PATCH)
  4. Validate templates for alignment
  5. Generate sync impact report

Output:
  - /memory/constitution.md (filled template)
  - Sync impact report (HTML comment at top):
    * Version: 1.0.0 → 1.1.0
    * Added: Principle 4 (Observability)
    * Modified: Principle 2 (Security-first → Zero-trust)
    * Template Status:
      - ✅ /templates/plan-template.md (updated)
      - ✅ /templates/spec-template.md (updated)
      - ⚠️  README.md (needs manual update)
```

### Example Session

```bash
# Initial project setup
/speckit.constitution "Project: E-Commerce Platform
Principle 1: API-First Design - All features exposed via REST APIs
Principle 2: Zero-Trust Security - Verify every request, trust nothing
Principle 3: Data Privacy - GDPR compliance, minimal data collection
Principle 4: Observability - Comprehensive logging and metrics"

# Output:
# ✅ Constitution created (v1.0.0)
# ✅ Templates validated and updated
# ⚠️  Manual update needed: README.md, docs/quickstart.md
#
# Suggested commit: "docs: establish project constitution v1.0.0"

# Later: Update constitution
/speckit.constitution "Add Principle 5: Progressive Enhancement"

# Output:
# ✅ Constitution updated (v1.0.0 → v1.1.0)
# Added: Principle 5 (Progressive Enhancement)
# ✅ All templates remain aligned
```

### Validation Rules

Before writing constitution:
- ✅ No unexplained `[PLACEHOLDER]` tokens remain
- ✅ Version line matches report
- ✅ Dates in ISO format (YYYY-MM-DD)
- ✅ Principles are declarative and testable
- ✅ MUST/SHOULD rationale is explicit

### Downstream Impact

Constitution changes affect:
1. **SPECIFY**: Validates specs against principles
2. **PLAN**: Constitution Check gate (ERROR if violations)
3. **ANALYZE**: Constitution alignment is CRITICAL severity
4. **IMPLEMENT**: Project setup follows constitution constraints

---

## 6. Research & Planning Pipeline

**Description:** Internal multi-phase workflow within the PLAN prompt that resolves unknowns before generating design artifacts.

**When to Use:** Automatically triggered during `/speckit.plan` when technical unknowns exist.

### ASCII Workflow Diagram

```
/speckit.plan executed
         │
         ▼
┌─────────────────────────────┐
│  Load Context               │
│  - spec.md (requirements)   │
│  - constitution.md (gates)  │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Fill Technical Context     │
│  - Mark unknowns as         │
│    "NEEDS CLARIFICATION"    │
│  - Identify dependencies    │
│  - List integrations        │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ PHASE 0: Research           │
│ ─────────────────           │
│ For each unknown:           │
│   1. Generate research task │
│   2. Research best practice │
│   3. Evaluate alternatives  │
│   4. Make decision          │
│   5. Document rationale     │
└──────────┬──────────────────┘
           │
           │ Creates: research.md
           │ ─────────────────────────
           │ Decision: [what chosen]
           │ Rationale: [why chosen]
           │ Alternatives: [evaluated]
           │
           ▼
┌─────────────────────────────┐
│ PHASE 1: Design & Contracts │
│ ─────────────────────────── │
│ 1. Extract entities         │
│    → data-model.md          │
│ 2. Generate API contracts   │
│    → contracts/             │
│ 3. Create quickstart        │
│    → quickstart.md          │
│ 4. Update agent context     │
│    → [agent]/context.md     │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Re-evaluate Constitution   │
│  Check post-design          │
│  - Still aligned? ✅        │
│  - Violations? ❌ ERROR     │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Report & Stop              │
│  - Branch: [name]           │
│  - IMPL_PLAN: path          │
│  - Artifacts: list          │
│  - Next: /speckit.tasks     │
└─────────────────────────────┘
```

### Phase 0: Research (Internal)

**Trigger:** NEEDS CLARIFICATION markers in Technical Context

**Process:**

```
For each unknown:
  1. Identify question type:
     - Technology choice (e.g., "Which DB: SQL vs NoSQL?")
     - Best practice (e.g., "How to handle API versioning?")
     - Integration pattern (e.g., "Auth flow with external IdP?")

  2. Generate research task:
     Task: "Research {unknown} for {feature context}"

  3. Execute research:
     - Consider requirements from spec.md
     - Consider constitution constraints
     - Evaluate 2-3 alternatives
     - Apply decision criteria (performance, security, maintainability)

  4. Document decision in research.md:
     ### {Unknown Topic}
     **Decision**: {Chosen option}
     **Rationale**: {Why this choice}
     **Alternatives Considered**:
     - Option A: {Why rejected}
     - Option B: {Why rejected}
```

**Example research.md:**

```markdown
# Research Findings: User Authentication

### Database Choice
**Decision**: PostgreSQL with row-level security
**Rationale**:
- ACID compliance required for financial data
- Native JSON support for flexible user profiles
- Row-level security aligns with zero-trust principle

**Alternatives Considered**:
- MongoDB: Rejected due to lack of ACID guarantees
- MySQL: Rejected due to weaker JSON support

### Session Management
**Decision**: Redis-backed JWT tokens with 15min TTL
**Rationale**:
- Stateless tokens scale horizontally
- Redis allows instant revocation
- 15min TTL limits exposure window

**Alternatives Considered**:
- Server-side sessions: Rejected due to scaling constraints
- Long-lived JWTs: Rejected due to security principle
```

### Phase 1: Design (Internal)

**Prerequisites:** research.md complete (all unknowns resolved)

**Outputs:**

1. **data-model.md**: Entity extraction
   ```
   spec.md: "Users can create projects with tasks"

   → Entities:
     - User (id, email, name, created_at)
     - Project (id, user_id, title, description, created_at)
     - Task (id, project_id, title, status, due_date)

   → Relationships:
     - User --< Project (one-to-many)
     - Project --< Task (one-to-many)
   ```

2. **contracts/**: API generation
   ```
   spec.md: "Users can create a new project"

   → Endpoint:
     POST /api/v1/projects
     Request: { title, description }
     Response: 201 { id, title, ... }
     Errors: 400, 401, 403
   ```

3. **quickstart.md**: Integration scenarios
   ```
   # Quick Start: User Authentication

   ## Register New User
   POST /api/v1/auth/register
   { "email": "test@example.com", "password": "..." }

   ## Login
   POST /api/v1/auth/login
   { "email": "test@example.com", "password": "..." }
   Response: { "token": "eyJ..." }
   ```

4. **Agent context update**: Technology sync
   ```
   Run: scripts/bash/update-agent-context.sh claude

   Detects: Claude Code agent in use
   Updates: .claude/context.md

   Adds between markers:
   <!-- BEGIN AUTO-GENERATED TECH STACK -->
   - Language: Node.js 20.x
   - Framework: Express 4.x
   - Database: PostgreSQL 15
   - Cache: Redis 7.x
   - Auth: JWT with bcrypt
   <!-- END AUTO-GENERATED TECH STACK -->
   ```

### Research Decision Criteria

When resolving unknowns, consider:
1. **Constitution alignment**: Does choice align with principles?
2. **Requirements fit**: Does it satisfy functional/non-functional requirements?
3. **Risk profile**: Security, performance, maintainability implications?
4. **Team capability**: Existing expertise vs learning curve?
5. **Ecosystem maturity**: Community support, library availability?
6. **Cost**: Licensing, infrastructure, operational overhead?

---

## Summary: Pipeline Selection Guide

### Choose Main Feature Development Pipeline when:
- ✅ Production features
- ✅ Complex requirements
- ✅ Multiple stakeholders
- ✅ High quality requirements
- ✅ Team environments

### Choose Quick Feature Pipeline when:
- ✅ Simple features
- ✅ Prototypes/MVPs
- ✅ Personal projects
- ✅ Clear requirements
- ✅ Low risk

### Choose Specification Refinement when:
- ✅ Ambiguous requirements
- ✅ Stakeholder alignment needed
- ✅ High-risk features
- ✅ Requirements quality critical

### Add Quality Validation when:
- ✅ Need domain-specific checks (UX, API, security)
- ✅ Before milestones/reviews
- ✅ Want "unit tests for requirements"
- ✅ Catch inconsistencies early

### Run Constitution Update when:
- ✅ Project initialization
- ✅ Adding/modifying principles
- ✅ Governance changes
- ✅ Before new features

---

## Quick Reference: Command Sequences

```bash
# Full pipeline with all validation
/speckit.constitution "[principles]"
/speckit.specify "[feature]"
/speckit.clarify
/speckit.plan
/speckit.tasks
/speckit.analyze
/speckit.checklist "ux" && /speckit.checklist "api" && /speckit.checklist "security"
/speckit.implement

# Quick pipeline
/speckit.specify "[feature]"
/speckit.plan
/speckit.tasks
/speckit.implement

# Specification refinement
/speckit.specify "[complex feature]"
/speckit.clarify
/speckit.clarify "focus on security"
/speckit.plan

# Quality validation only
/speckit.checklist "ux"
/speckit.checklist "api"
/speckit.analyze
```

---

**End of Agent Pipelines Documentation**

For detailed prompt templates and parameters, see `PROMPTS_DOCUMENTATION.md`.
