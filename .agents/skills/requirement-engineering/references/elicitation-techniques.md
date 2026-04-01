# Elicitation Techniques Reference

## Interview Patterns

### Structured Interview

- Prepare a fixed question set in advance
- Best for: regulated domains, consistency across many stakeholders
- Key questions:
  - "What does the system need to do for you?"
  - "What are the biggest pain points in the current process?"
  - "What does success look like in 6 months?"
  - "What would cause you to reject this system?"

### Semi-Structured Interview

- Core questions + follow-up probing
- Use laddering: "Why is that important?" → drill to root need
- Use the "5 Whys" for problem discovery

### Contextual Inquiry

- Observe users in their natural work environment
- Ask "talk-aloud" questions while they perform tasks
- Uncovers implicit/tacit knowledge users cannot articulate

---

## Workshop Facilitation

### Requirements Workshop (JAD — Joint Application Development)

- Participants: business owners, end users, developers, BA facilitator
- Outputs: agreed feature list, initial use cases, constraint list
- Techniques:
  - Whiteboard storming → affinity grouping
  - Dot-voting for prioritization
  - "How might we..." framing for feature ideation

### Impact Mapping

- Goal → Actors → Impacts → Deliverables
- Connects business goals to system features
- Prevents scope creep by always linking to a goal

### Story Mapping (Agile)

- Activities (horizontal) → Tasks (vertical priority)
- Creates narrative flow across user journeys
- Helps identify MVP slice vs. later iterations

---

## Document Analysis

- Review existing: process manuals, legacy system specs, regulatory docs, contracts
- Extract: implicit rules, data definitions, exception handling, SLAs already committed
- Always cross-validate with stakeholders — documents go stale

---

## Observation Techniques

- **Shadowing**: follow a user through their full workflow silently
- **Think-aloud protocol**: user narrates what they're doing and why
- **Field study**: extended time in the work environment

---

## Prototyping for Elicitation

- Use wireframes or low-fi mockups to surface hidden requirements
- "Show don't tell" — stakeholders react better to visuals than abstract descriptions
- Two types:
  - **Throwaway prototypes**: explore and discard
  - **Evolutionary prototypes**: refine into final product

---

## Common Elicitation Pitfalls

| Pitfall | Prevention |
|---------|-----------|
| Stakeholders describe solutions, not problems | Ask "what problem does that solve?" |
| Missing silent stakeholders | Stakeholder mapping exercise upfront |
| Gold-plating | Enforce MoSCoW; link every feature to a business goal |
| Requirements from HiPPO (Highest Paid Person's Opinion) | Validate against real user data |
| Scope creep through verbal agreements | Document and baseline all decisions |
