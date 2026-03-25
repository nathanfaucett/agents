---
name: agent-ready
description: |
   Analyzes a codebase to identify gaps, inconsistencies, and friction points that hinder
   autonomous agents or developers from effectively understanding, modifying, and
   extending the project. Invoke when assessing repository readiness for autonomous
   agents or onboarding developers.
---

## Summary

Analyze a codebase to identify gaps, inconsistencies, and friction points that would hinder autonomous agents (or developers) from effectively understanding, modifying, and extending the project. Provide concrete, prioritized recommendations to improve agent operability.

---

### Inputs

* Project files (full or partial repository)
* Optional:

  * Project guidelines / contributing docs
  * CI/CD configs
  * Issue tracker or roadmap
  * Target agent capabilities (e.g., codegen, refactor, test-writing)

---

### Outputs

Structured report with:

1. **Summary**

   * High-level assessment of agent readiness
   * Key risks and missing capabilities

2. **Gap Analysis**

   * Missing or unclear structure
   * Incomplete abstractions
   * Poor naming or discoverability
   * Hidden coupling / implicit behavior
   * Inconsistent patterns

3. **Agent Friction Points**

   * Areas where intent is unclear
   * Non-local reasoning required
   * Lack of type safety or contracts
   * Dynamic or implicit behavior
   * Missing or weak test coverage
   * Unclear side effects or state flow

4. **Documentation Deficiencies**

   * Missing high-level architecture overview
   * Missing module/service boundaries
   * Lack of “how to extend” guidance
   * Missing API or schema definitions

5. **Tooling & Automation Gaps**

   * Missing linting / formatting rules
   * Weak or absent CI checks
   * No type enforcement
   * Missing code generation or scaffolding tools

6. **Recommendations (Prioritized)**

   * Ordered by impact vs effort
   * Each includes:

     * Problem
     * Why it matters for agents
     * Concrete fix
     * Example (if applicable)

7. **Quick Wins**

   * Small, high-impact changes

8. **Long-Term Improvements**

   * Architectural or systemic changes

---

### Evaluation Heuristics

#### 1. Clarity & Explicitness

* Are behaviors explicit vs implicit?
* Are types/interfaces well-defined?
* Can intent be inferred locally?

#### 2. Consistency

* Naming conventions
* File/module structure
* Patterns (hooks, services, data access, etc.)

#### 3. Composability

* Are components modular and reusable?
* Are boundaries well-defined?

#### 4. Discoverability

* Can an agent find:

  * Entry points?
  * Core logic?
  * Data models?
  * Extension points?

#### 5. Determinism

* Are side effects controlled and predictable?
* Is behavior testable and reproducible?

#### 6. Testability

* Unit/integration coverage
* Mockability
* Isolation of logic

#### 7. Type Safety / Contracts

* Strong typing or schema validation
* Clear input/output contracts

#### 8. Documentation Quality

* Architecture overview
* Module responsibilities
* Contribution patterns

---

### Detection Patterns

Flag when:

* Logic spans multiple unrelated files without clear linkage
* Magic strings / implicit contracts are used
* Dynamic typing obscures structure
* Side effects are hidden (I/O, mutation, globals)
* Functions/classes exceed reasonable complexity
* Inconsistent abstractions exist for similar tasks
* Tests are missing for core logic
* Naming does not reflect intent

---

### Recommendation Patterns

Generate fixes such as:

* Introduce typed interfaces or schemas
* Extract pure functions from side-effect-heavy code
* Standardize patterns (e.g., data access layer, service layer)
* Add index/entry files for discoverability
* Introduce linting/formatting rules
* Add test scaffolding and examples
* Create architecture and extension docs
* Replace implicit behavior with explicit configuration

---

### Output Format (Example)

```md
## Summary
Project is moderately agent-friendly but suffers from implicit behavior and weak structure.

## Key Gaps
- Missing clear service boundaries
- Inconsistent data access patterns
- Lack of type contracts in core flows

## Agent Friction
- Requires cross-file reasoning for simple changes
- Hidden side effects in utility functions

## Recommendations

### 1. Introduce Service Layer (High Impact / Medium Effort)
Problem: Business logic scattered across routes and utils  
Fix: Extract into `/services/*` with explicit interfaces  

### 2. Add Type Contracts (High Impact / Low Effort)
Problem: Unclear data shapes  
Fix: Define shared types or schemas  

## Quick Wins
- Add ESLint + Prettier config
- Add README with architecture overview

## Long-Term
- Refactor toward modular domain structure
```

---

### Behavior Guidelines

* Be opinionated but practical
* Prefer concrete fixes over abstract advice
* Optimize for agent comprehension, not just human readability
* Avoid over-engineering recommendations
* Assume the goal is **autonomous modification with minimal context**
