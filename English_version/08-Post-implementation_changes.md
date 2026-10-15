# Post-Implementation Changes in SDD

## Iterating after the first version

The first implementation does not end the **Spec-Driven Development** process. When you test the product, new needs, improvements, or features that were not originally planned may appear.

The fundamental idea is:

> **A new need is not implemented directly in the code. First it is added to the specification, and then the SDD cycle is run again.**

Suppose ProductChat is already working, but after using it we want to add two improvements:

1. A home page that gives access to the chatbot's main functions.
2. A more professional and consistent interface.

The second improvement is deliberately phrased the way a user would say it: it is vague and cannot be verified. We leave it that way so we can see later how Clarify and the checklist turn it into testable requirements. Instead of asking the AI directly to modify the code, we start a new iteration.

> **A note on identifiers.** Spec Kit creates one directory per specification, numbered in the order they are generated. In this material we call the initial ProductChat specification `SPEC001` and the one for this second iteration `SPEC002`. The exact directory name depends on your Spec Kit version; check how yours names them and replace the reference where needed.

## 1. Create the new specification

We use again:

```text
/speckit-specify
```

```text
/speckit-specify

ROLE
Act as a requirements analyst working on a product that is already implemented.

CONTEXT
[PRODUCT NAME] is working. I want to add changes without breaking what is
already there.

INPUT
The existing specification, [PATH TO THE PREVIOUS SPEC].

REQUESTED CHANGES
1. [CHANGE 1]
2. [CHANGE 2]
3. [CHANGE N]

EXPECTED OUTPUT
A new, separate specification that describes only these changes and
explicitly states which existing behavior remains intact.

CONSTRAINTS
Keep unchanged any existing behavior that is not explicitly affected by these
requirements. Do not rewrite the previous specification.

ACCEPTANCE CRITERION
The new specification can be read alongside the previous one without
contradicting it, and its acceptance criteria can be verified without reading
code.
```

> This prompt is in `prompts/08.1-change-specify.txt`, ready to copy. If you change it, change it there: that file is the reference version.

### ProductChat example

```text
/speckit-specify

ROLE
Act as a requirements analyst working on a product that is already implemented.

CONTEXT
ProductChat is working on top of the Distribuidora Andes Excel catalog.
I want to improve how it is used, without touching how it reads the data.

INPUT
The existing specification, SPEC001/spec.md.

REQUESTED CHANGES
1. A home page from which the user can access the chatbot and look up
   basic information about the catalog.
2. A consistent, clear, and professional visual presentation.

EXPECTED OUTPUT
A new, separate specification, SPEC002, that describes only these changes.

CONSTRAINTS
Keep product search, catalog normalization, and the handling of conflicting
SKUs unchanged. Do not rewrite SPEC001.

ACCEPTANCE CRITERION
The acceptance criteria of SPEC001 are still met without modification.
```

Spec Kit can create a new specification, for example `SPEC002`, keeping the new feature separate from the original implementation.

## 2. Clarify the new requirements

Before planning, we run Clarify with the prompt in `prompts/03.2-clarify.txt`, adjusting its context block to the new requirements:

```text
/speckit-clarify
```

Clarify lets us detect ambiguous expressions such as *"professional interface"* or *"clear presentation"* and turn them into more concrete requirements.

When the change affects a domain that requires specific criteria, you can also use `/speckit-checklist`, an additional quality gate that generates a themed checklist (usability, security, accessibility, or another domain) to review the new requirements before planning:

```text
/speckit-checklist
```

```text
/speckit-checklist

ROLE
Act as a reviewer specializing in [UX / security / accessibility / other].

CONTEXT
[Which part of ProductChat is being reviewed and why.]

INPUT
spec.md, in particular the requirements added in this iteration.

EXPECTED OUTPUT
A [domain] checklist, where each item is answered with yes or no by using the
application.

ACCEPTANCE CRITERION
No item on the list requires reading code to be answered, and none is
subjective: "the interface looks professional" is not a valid item.
```

> This prompt is in `prompts/08.2-checklist.txt`, ready to copy. If you change it, change it there: that file is the reference version.

### ProductChat example

```text
/speckit-checklist

ROLE
Act as a reviewer specializing in UX.

CONTEXT
The new ProductChat home page and the redesign of how results are presented
are being reviewed. The chatbot often shows several products and, sometimes,
warnings about catalog inconsistencies.

INPUT
SPEC002/spec.md.

EXPECTED OUTPUT
A UX checklist where each item is answered with yes or no by using the
application.

ACCEPTANCE CRITERION
No item requires reading code or making an aesthetic judgment.
```

## 3. Create the Plan while respecting the existing implementation

The new Plan must reuse the existing technical decisions unless the change explicitly requires modifying them.

```text
/speckit-plan

ROLE
Act as an architect working on an existing codebase.

CONTEXT
This feature modifies [AFFECTED AREA] of an application that already works.

INPUT
The new specification, plus [PATH TO THE PREVIOUS PLAN] and [PATH TO THE
PREVIOUS DATA MODEL], from which you must take the technology stack, the
conventions, the data model, and the project structure.

EXPECTED OUTPUT
A plan that reuses the existing technical decisions and describes only what
changes.

CONSTRAINTS
Do not replace or redesign the stack, the conventions, or the data model unless
the new requirements make it essential. Limit the changes to what the new
specification asks for.

ACCEPTANCE CRITERION
The plan explicitly lists which parts of the system it does not touch. No
component that already worked appears redesigned without a written
justification.
```

> This prompt is in `prompts/08.3-change-plan.txt`, ready to copy. If you change it, change it there: that file is the reference version.

### ProductChat example

```text
/speckit-plan

ROLE
Act as an architect working on an existing codebase.

CONTEXT
This feature modifies the presentation layer of ProductChat.

INPUT
SPEC002/spec.md, plus SPEC001/plan.md and SPEC001/data-model.md, from which
you must take the stack, the conventions, the data model, and the structure.

EXPECTED OUTPUT
A plan that reuses those decisions and describes only what changes.

CONSTRAINTS
Do not replace or redesign catalog access or the normalized internal model.
Limit the changes to the home page and the interface.

ACCEPTANCE CRITERION
The plan lists which parts of the system it does not touch.
```

This instruction is important because it prevents a localized improvement from triggering an unnecessary redesign of parts of the system that already work.

## 4. Generate, analyze, and implement

From here on, we repeat the normal cycle:

```text
/speckit-tasks
        ↓
/speckit-analyze
        ↓
/speckit-implement
        ↓
Validation
```

`Tasks` breaks the change down into executable work. `Analyze` checks that the new specification, the Plan, and the tasks are consistent with the existing artifacts. Finally, `Implement` makes the changes and verifies the tasks.

If new needs appear during validation, we start another iteration:

```text
Implemented product
        ↓
New need
        ↓
Specification
        ↓
Clarify / Checklist
        ↓
Plan
        ↓
Tasks
        ↓
Analyze
        ↓
Implementation
        ↓
Validation
        ↺
```

The advantage is that ProductChat can evolve without losing traceability of **what was requested, why it was changed, how it was decided to implement it, and which tasks finally produced the change in the code**.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
