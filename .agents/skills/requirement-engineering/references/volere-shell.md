# Volere Requirement Shell Template

The Volere shell provides a complete, self-contained record for a single requirement.
Use it when maximum traceability and rigor are needed (regulated, safety-critical, or large-scale projects).

---

## Volere Shell

| Field                  | Content |
|------------------------|---------|
| **Requirement #**      | [Unique ID, e.g., REQ-042] |
| **Requirement Type**   | [Functional / NFR — Performance / Security / Usability / etc.] |
| **Event / Use Case #** | [Link to triggering use case or event] |
| **Description**        | [One clear, testable sentence using SHALL] |
| **Rationale**          | [Why this requirement exists; business justification] |
| **Source**             | [Stakeholder name/role who raised this] |
| **Fit Criterion**      | [Measurable pass/fail condition — this is the test] |
| **Customer Satisfaction** | [1–5: how happy if included] |
| **Customer Dissatisfaction** | [1–5: how unhappy if omitted] |
| **Priority**           | [MoSCoW: Must / Should / Could / Won't] |
| **Conflicts**          | [IDs of any conflicting requirements] |
| **Dependencies**       | [IDs of requirements this depends on] |
| **Supporting Material**| [References, diagrams, examples] |
| **History**            | [Date created, author, change log] |

---

## Example: Completed Volere Shell

| Field                  | Content |
|------------------------|---------|
| **Requirement #**      | REQ-017 |
| **Requirement Type**   | Non-Functional — Performance |
| **Event / Use Case #** | UC-003 (User submits search query) |
| **Description**        | The system SHALL return search results within 300ms at the 95th percentile under a load of 5,000 concurrent users. |
| **Rationale**          | User research shows abandonment rate doubles if search exceeds 500ms. Revenue impact estimated at $200K/month per 100ms added latency. |
| **Source**             | Sarah Chen, Head of Product |
| **Fit Criterion**      | Performance test: 5,000 virtual users, 300ms p95 latency on search endpoint. Test passes if ≥ 95% of requests complete within 300ms. |
| **Customer Satisfaction** | 4 |
| **Customer Dissatisfaction** | 5 |
| **Priority**           | Must |
| **Conflicts**          | None |
| **Dependencies**       | REQ-015 (Search indexing pipeline), REQ-016 (Caching layer) |
| **Supporting Material**| Lighthouse performance budget doc v2.1 |
| **History**            | 2025-01-10 created by J. Reyes; 2025-02-03 threshold updated from 500ms to 300ms based on A/B test results |

---

## MoSCoW Prioritization Guide

| Priority | Meaning | Guidance |
|----------|---------|---------|
| **Must** | Non-negotiable; system fails without it | ~60% of requirements at most |
| **Should** | Important but a workaround exists | Include if time allows |
| **Could** | Nice to have; low cost of exclusion | Defer to later iterations |
| **Won't (this time)** | Explicitly out of scope now | Document to prevent scope creep |

---

## Fit Criterion Writing Rules

A fit criterion is the key differentiator of a Volere shell from a weak requirement.
It must be:

1. **Quantitative** — contain a number, threshold, or binary pass/fail
2. **Independent** — a tester who didn't write the requirement can execute it
3. **Unambiguous** — no interpretation required

Bad: "System should be fast"
Good: "95th percentile response time ≤ 200ms at 1,000 concurrent users (measured by load test)"

Bad: "Users should find the interface easy"
Good: "≥ 85% of first-time users complete the onboarding flow without requesting help (usability test, n=20)"
