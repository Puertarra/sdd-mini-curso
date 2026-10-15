# Consistency Analysis in SDD
## What is Analyze?

Once the tasks have been generated and before implementation begins, it is advisable to add an **additional quality gate** through **Analyze**.

In **GitHub Spec Kit** it is run with:

```text
/speckit-analyze

ROLE
Act as a consistency auditor. Detect and explain problems; do not fix the
artifacts on your own.

CONTEXT
ProductChat, before implementation begins.

INPUT
.specify/memory/constitution.md, spec.md, plan.md, data-model.md and tasks.md.

EXPECTED OUTPUT
A findings report, each finding with a severity (CRITICAL, HIGH, MEDIUM, LOW),
the artifact and the section where the problem is, and a concrete
recommendation.

WHAT TO LOOK FOR
Requirements or acceptance criteria with no associated task. Tasks that
implement something that does not appear in the Specification. Duplicate tasks.
Plan decisions that contradict the Constitution. Refined stories with an
implementation task but no verification task. And, in particular, any point
where an artifact has gone back to assuming that the catalog is clean.

ACCEPTANCE CRITERION
Each finding points to a specific file and section, and I can check it by
opening that file. There are no generic findings of the kind "it would be
worth reviewing the coverage".
```

> This prompt is in `prompts/06.1-analyze.txt`, ready to copy. If you change it, change it there: that file is the reference version.


Analyze is not a stage: it is a **quality gate**, just like Clarify. It does not produce a new artifact; it reviews the ones that already exist. And it serves a different purpose from Clarify. While **Clarify reviews the Specification on its own**, Analyze checks the **consistency and coverage across the different artifacts of the project**.

Its goal is to answer the question: **is everything we have defined so far consistent, covered and compliant with the Constitution?**

## What does it analyze?

Analyze reviews the main artifacts generated during the process together:

```text
Constitution
     ↓
Specification
     ↓
Plan
     ↓
Tasks
```

Depending on the project, it can also consider other documents generated during planning, such as the data model or the API contracts.

The analysis mainly looks for problems of **consistency, coverage and traceability**. For example:

- A requirement in the Specification that has no associated task.
- A task that implements a feature that does not exist in the Specification.
- Two tasks that needlessly implement the same thing.
- A Plan decision that contradicts the Constitution.
- A user story that was only partially covered by the tasks.
- An acceptance criterion for which there is no clear way to verify it.

## Difference between Clarify and Analyze

Although both work as quality controls, they act at different moments and at different levels.

`Clarify = reviews the quality of the Specification`

`Analyze = reviews the coherence across all the artifacts`

For example, **Clarify** could detect that the ProductChat Specification does not say whether, given two conflicting prices for the same SKU, it should report both or apply a tie-breaking rule. That ambiguity is resolved directly in the Specification.

**Analyze**, on the other hand, works on what has already been decided. Once BR5 and AC3 establish that the conflict must be reported, Analyze can detect that task `US4.4` implements that behavior but that no task in the validation phase verifies it. The Specification is fine; what fails is its propagation.

## Traceability

One of the most important aspects of Analyze is checking the traceability between requirements, user stories, technical decisions and tasks.

For ProductChat we could have:

```text
BR5 / AC3
   ↓
US4: Resolve an inconsistent catalog
   ↓
US4.1: Category equivalences
US4.2: Unit conversion
US4.3: Conditional attributes
US4.4: Conflicts between duplicate SKUs
   ↓
Technical plan
   ↓
Implementation tasks
   ↓
Verification tasks
```

Analyze lets you detect breaks in this chain. If `US4.4` is defined and has an implementation task, but no task verifies it, acceptance criterion `AC3` is not covered even though everything *seems* to be done. That is the kind of gap you cannot see by reading one artifact on its own.

It can also detect the opposite problem: a task meant to implement personalized recommendations when that feature was never defined in the Specification. In that case there is **unspecified scope**.

## Running it with GitHub Spec Kit

Once `tasks.md` has been reviewed, we run the previous prompt. There is no need to add information about the case: the prompt names the artifacts as input and Spec Kit reads them.

Findings can be classified by severity, for example:

```text
CRITICAL
HIGH
MEDIUM
LOW
```

A critical or high problem should be reviewed before implementation begins. Medium or low problems may represent minor inconsistencies, traceability problems or aspects worth reviewing.

## Example findings

Let's run Analyze on the artifacts we have been building. The `tasks.md` from the previous module has three real problems, and those are the ones Analyze should find.

The first is the most serious, because it affects an acceptance criterion:

```text
[HIGH] Refined story US4.4 (report duplicate SKUs with conflicting values)
has an implementation task in Phase 3, but there is no verification task in
Phase 4. Acceptance criterion AC3 depends directly on this behavior and would
remain unchecked.

Recommendation:
Add to Phase 4 a validation task associated with US4.4, verifiable
against SKU NB-1001 in the catalog.
```

The second is a duplication:

```text
[MEDIUM] Tasks US2.1 ("query the available attributes of a product") and
US2.2 ("get the available features of a product") describe the same
behavior on the same entity. The Specification does not distinguish an
attribute from a feature.

Recommendation:
Merge both into a single refined story, or specify in the Specification
how they differ.
```

The third is a minor coverage gap:

```text
[MEDIUM] US3.2 (inform the user when no products are found) has no
associated verification task.

Recommendation:
Add the corresponding validation, or justify why it is covered by the
verification of US3.1.
```

Analyze can also detect the reverse problem, a task that implements something nobody asked for:

```text
[HIGH] There is a task to correct and rewrite the Excel file with the
normalized data. This feature does not appear in the Specification and
contradicts the constitutional principle "the chatbot reads, it does not correct".

Recommendation:
Remove the task. If the company wants a clean version of the catalog,
it must be specified as a separate product.
```

This last case is instructive: it is not just unspecified scope, it is a violation of the Constitution. An agent that finds messy data naturally tends to want to clean it, and the Constitution exists precisely to prevent that without a human decision in between.

## Correcting inconsistencies

Analyze should be used mainly to **detect and explain problems**, not to modify the artifacts automatically without review.

When it finds an inconsistency, a concrete correction can be proposed. Once it has been reviewed and accepted, the corresponding artifact is updated.

The correction must be made at the level where the problem really is:

`Problem in the what → correct the Specification`

`Problem in the how → correct the Plan`

`Problem in the execution → correct the Tasks`

`Problem with a global rule → review it against the Constitution`

After making important changes, you can run `/speckit-analyze` again to check that the artifacts are aligned.

## Analyze as the gate before implementation

Analyze works as the last global review before code generation begins. Up to this point we have worked mainly on documents and decisions, so correcting an inconsistency is still relatively easy.

The complete flow before implementation becomes:

```text
Constitution
     ↓
Specification
     ↓
Clarify
     ↓
Plan
     ↓
Tasks
     ↓
Analyze
     ↓
Implementation
```

In this way, **Clarify** reduces the ambiguities of the Specification and **Analyze** then checks that this Specification has been correctly propagated to the Plan and the Tasks. Only after passing these reviews does implementation begin.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
