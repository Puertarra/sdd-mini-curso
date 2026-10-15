# Spec Drift in SDD

Module 07 ended with a warning. If the spreadsheet shows an inconsistency that no rule covers, the temptation is to patch the code directly, and the right thing to do is to take the rule to the Specification. This module is about what happens when you give in to that temptation, again and again.

**Spec Drift** is the divergence between what the current requirements say and what the code does. It does not appear all at once: it builds up with every urgent fix that did not go through the Spec. Git, which we saw in the previous module, records each of those changes, but it does not warn you that the code and the Spec no longer say the same thing. For that, you need to audit.

## What role the Spec plays after implementation

Not every project asks the same thing of a specification once the code exists. Birgitta Böckeler proposed distinguishing three approaches, and the Spec Kit documentation adopts them:

| Approach | What happens to the Spec | In ProductChat |
|---|---|---|
| **Spec-first** | It guides development and can be discarded afterwards | SPEC001 would remain as a historical record of how the project started |
| **Spec-anchored** | It is kept and maintained as the reference for behavior | Every change goes through a Spec, and the current Specs describe what ProductChat does today |
| **Spec-as-source** | It is the only thing people edit; the code is generated from it | We do not use it in this course |

Since module 08, the course works **spec-anchored**: a delivered change is recorded in a new Spec that references the earlier ones, and the set of current Specs describes the current behavior. That approach has a risk of its own, and it is precisely drift: a Spec kept as a reference is only useful if it keeps telling the truth.

There is a second decision, independent of the first: when a requirement changes, where do you write the change? The Spec Kit documentation describes three ways of working. They are not tool options (the tool works the same in all three) but conventions the team agrees on:

| Mode | Where the change is written | Advantage | Risk |
|---|---|---|---|
| **Flow-forward** | In a new Spec; the earlier ones stay intact | Each Spec is a faithful record of what was requested at the time | What is current ends up spread across several Specs |
| **Living spec** | In the same `spec.md`, and the plan and tasks are regenerated | The Spec always describes the complete behavior | The reasons behind earlier decisions are lost |
| **Flow-back** | Where the problem appeared (code, tasks, or plan), and then the other artifacts are reconciled | Fast iteration | Silent drift between artifacts |

The course works **flow-forward**. In this module, SPEC004 does not repeat SPEC001: it contains only the change and declares which criterion of SPEC001 it replaces. The consequence is that ProductChat's current behavior is not in a single Spec but in the whole set read in order, and that SPEC001 still shows an AC1 that no longer applies. So that this does not confuse anyone, neither a teammate nor the agent, each new Spec explicitly declares what it replaces, and `specs/README.md` summarizes it in an index:

```text
| Spec    | Purpose           | Status      | Modifies     |
|---------|-------------------|-------------|--------------|
| SPEC001 | Catalog query     | Implemented | —            |
| SPEC002 | Home page         | Implemented | —            |
| SPEC003 | Filters           | Implemented | —            |
| SPEC004 | Hide out of stock | In progress | SPEC001: AC1 |
```

In module 09 that index was optional. Once one Spec replaces part of another, it is necessary. The index tells you which Specs exist; at the end of the module we will also generate a view with the requirements that apply today.

Böckeler, B. (2025). [*Understanding Spec-Driven-Development: Kiro, spec-kit, and Tessl*](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html). martinfowler.com. · GitHub. *Spec Kit v1.0.8*, documentation: `docs/concepts/spec-persistence.md`.

## What drift is, and what it is not

In ProductChat, a drift looks like this:

```text
Current requirement (SPEC001, AC1):  NB-1007 appears among the 32 GB notebooks
Code after a patch:                  NB-1007 does not appear, because its stock is "Yes" and not a number
                                               ↓
                                     The code no longer does what the Spec says
```

Not every difference between a document and the code is drift. It is normal for the code to have functions, variables, or structures that the Spec does not mention: the Spec describes behavior, not implementation. And a historical Spec that describes an earlier version is not wrong, just superseded. There is drift when a **current requirement** and the **observable behavior** contradict each other.

These are the usual causes:

| Situation | What it produces |
|---|---|
| An urgent change goes straight to the code | Behavior with no requirement and no test |
| The agent is asked for an adjustment after implementation | The result drifts away from the accepted scope |
| A requirement changes, but the plan and tasks do not | The code follows an obsolete decision |
| Two current Specs say different things | Nobody knows which behavior is correct |

A warning: passing tests do not rule out drift. Tests check what someone decided to test. If an acceptance criterion never became a test, the code can stop meeting it without anything failing.

## The case: hiding what is not there

The salespeople are complaining: ProductChat suggests notebooks that are not in the warehouse, and they end up offering what they cannot sell. An urgent request comes in, and someone solves it directly in the code.

To study it, we are going to cause that drift on purpose, on a practice branch:

```bash
git switch -c 004-hide-out-of-stock
```

```text
CONTROLLED EXERCISE. This prompt deliberately does not follow the course
contract: it is an urgent request like the ones that produce Spec Drift. Use it
only on the practice branch.

Urgent: salespeople are offering products we don't have. Modify the ProductChat
code directly so that it does not show any product whose stock is not a number
greater than zero. Don't touch the Specs, the plan, or the tasks; we'll add the
tests later.
```

> This prompt is in `prompts/10.1-drift-counterexample.txt`, ready to copy. If you change it, change it there: that file is the reference version.

The result looks correct: `NB-1009`, with stock 0, no longer appears, and neither does `NB-1010`, which says `sold out`. Commit the patch the way someone in a hurry would:

```bash
git add src/
git commit -m "hotfix: hide out-of-stock products"
```

Now compare it against the spreadsheet. The rule "only numeric stock greater than zero" clashes with a catalog that was never numeric:

| SKU | Stock in the spreadsheet | What SPEC001 says | What the patch does |
|---|---|---|---|
| `NB-1009` | `0` | Out of stock | Hides it |
| `NB-1010` | `sold out` | Out of stock | Hides it |
| `NB-1007` | `Yes` | Available (AC5) | **Hides it** |
| `MON-3002` | empty | Stock not reported (FR6) | **Hides it** |
| `SRV-5001`, `SRV-5002`, `SRV-5003` | empty | Service: stock does not apply (EC6) | **Hides them** |

Applied to the course's canonical query, "which notebooks have 32 GB of RAM?", the patch returns `NB-1005` and `NB-1012`. SPEC001 says there are four.

Another common form of the same patch is to hide only when the stock is `0`. That one fails the other way: it leaves `NB-1010` visible, because `sold out` is not `0`. Both variants have the same cause: the rule was written without looking at the Spec, which already defined three stock states.

## Auditing in two directions

A drift audit is always done in two directions, because each one finds a different kind of problem:

```text
Spec → code:  does the code still meet the current requirements?
Code → Spec:  does the code do things that no Spec asks for?
```

The first direction detects unmet requirements. The second detects new rules that got in without going through the Specification. An urgent patch usually produces both kinds at once.

The evidence comes from the Constitution, the current Specs with their plans and tasks, the code, the tests, and the history. This command shows everything the branch changed since it split off from `main`:

```bash
git diff main...HEAD
```

```text
ROLE
Act as the consistency auditor for ProductChat. You detect and explain
divergences; you do not modify files or decide which behavior is correct.

CONTEXT
ProductChat has several current Specs in specs/. There may be code that
changed without its requirements changing, or requirements that changed
without the code changing. The catalog, product_catalog.xlsx, is the reference
for reproducing each behavior with concrete SKUs.

INPUT
.specify/memory/constitution.md, the Specs in specs/ with their plans and
tasks, the code, the tests, CLAUDE.md, the history of the current branch (git
log and git diff main...HEAD) and, if it exists, the report of a previous
audit.

EXPECTED OUTPUT
A report with findings numbered D-01, D-02… Each one states its direction
(Spec → code or code → Spec), severity, affected requirement, observed
behavior, the SKU that reproduces it, evidence with paths and lines, and
impact. If there is a previous report, add the status of each previous
finding: resolved, pending, or not verifiable.

WHAT TO LOOK FOR
Spec → code: requirements and acceptance criteria that the code no longer
meets. Code → Spec: implemented product rules that no Spec backs. Pay special
attention to stock, prices, categories, and SKU conflicts.

CONSTRAINTS
Distinguish what you executed from what you only read in the code. If you
cannot determine which requirement is current, record it as an ambiguity.

ACCEPTANCE CRITERION
Each finding is reproduced with the SKU it cites, and none is generic, along
the lines of "it would be worth reviewing coverage".
```

> This prompt is in `prompts/10.2-drift-audit.txt`, ready to copy. If you change it, change it there: that file is the reference version.

For the patch, the audit should find at least this:

| Finding | Direction | Severity | What it finds |
|---|---|---|---|
| D-01 | Spec → code | High | AC1 of SPEC001 fails: there are two 32 GB notebooks instead of four. `NB-1007` and `NB-1009` are missing |
| D-02 | Spec → code | High | The three services disappear, against EC6 |
| D-03 | Spec → code | Medium | `MON-3002` disappears, even though FR6 distinguishes "not reported" from "out of stock" |
| D-04 | Code → Spec | High | There is a hiding rule that no Spec backs |

D-01 has a subtlety worth discussing in class. That `NB-1007` is missing is an error: it has stock. That `NB-1009` is missing may not be: it has stock 0, and if the business decides that what is not there is not shown, its absence is correct. The audit cannot decide that. It can only show that the code and the Spec no longer match.

## Resolving the drift

Resolving a drift starts by deciding which behavior you want. Neither the code nor the Spec is right by default:

| If it is decided that… | Then |
|---|---|
| The current requirement is the correct one | The code is fixed |
| The new behavior is the correct one | It is formalized in a Spec and the tests are completed |
| The need is correct but the rule is wrong | The correct rule is specified and the code is adjusted |
| There is no agreement | It is resolved before touching anything else |

Here we are in the third case. The need is legitimate: do not offer what is not there. And that decision also closes a question that the napkin spec left open from the start. OQ2 asked whether "which notebooks have 32 GB?" refers to the catalog or to the stock. The business has just answered: to the stock.

It is a new Spec, SPEC004, because it changes a behavior that SPEC001 already delivered. And it has to say so explicitly: AC1 changes from four notebooks to three.

```text
/speckit-specify

ROLE
Act as a requirements analyst. You formalize a business decision that has
already been made; you neither design it nor implement it.

CONTEXT
An urgent fix hid out-of-stock products from the chatbot without going
through SDD, and the audit found that it also hid available products and
services. The need is legitimate; the rule was wrong. This Spec also resolves
open question OQ2 from the napkin spec: when a salesperson asks what we
"have", they are asking about what can be sold today.

INPUT
The spec.md of SPEC001, the audit report, .specify/memory/constitution.md,
and product_catalog.xlsx.

EXPECTED OUTPUT
The spec.md of the new Spec, which references SPEC001 and SPEC003 and states
explicitly which acceptance criteria of SPEC001 change.

RULES ALREADY DECIDED
- Out of stock (0 or "sold out"): the product does not appear in answers.
- Available (a number greater than zero, "Yes" or "yes"): it appears.
- Stock not reported (empty cell): it appears, with a notice.
- Services: stock does not apply, and they are never hidden.
- The rules apply to all queries, with or without filters.

CONSTRAINTS
Inspect the existing code and distinguish what already meets the rule from
what needs to be fixed. Do not accept a behavior just because it is already
implemented. Out of scope: modifying the catalog and managing inventory.

ACCEPTANCE CRITERION
The Spec updates AC1 of SPEC001: "which notebooks have 32 GB of RAM?" now
returns NB-1005, NB-1007, and NB-1012. NB-1007 appears, NB-1009 does not, and
SRV-5001 still appears.
```

> This prompt is in `prompts/10.3-out-of-stock-specify.txt`, ready to copy. If you change it, change it there: that file is the reference version.

```text
/speckit-plan

ROLE
Act as an architect working on code that already contains a faulty version of
this rule. You propose the how; the final decision is mine.

CONTEXT
The branch already has a patch that hides products based on their stock. It
does not need to be redone, but fixed: the three-state stock normalization
that SPEC001 defined already exists, and it is the one to use.

INPUT
The spec.md of SPEC004, the plan.md and data-model.md of SPEC001, the audit
report, and CLAUDE.md.

EXPECTED OUTPUT
The plan.md of SPEC004, which reuses the existing stack and normalization,
describes only the adjustments to the patch, and defines the tests with the
SKUs from the acceptance criteria.

CONSTRAINTS
Do not replace the stack or the data model. Do not duplicate the stock
normalization in another component.

ACCEPTANCE CRITERION
For each audit finding, the plan states which change resolves it.
```

> This prompt is in `prompts/10.4-out-of-stock-plan.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Then come Tasks, Analyze, and Implementation, with the prompts `prompts/05.1-tasks.txt`, `prompts/06.1-analyze.txt`, and `prompts/07.1-implement.txt`. The tasks must not rebuild what already works: the patch already hides `NB-1009` and `NB-1010` correctly, and that is kept.

The expected result, verifiable against the spreadsheet:

| SKU | Stock | With SPEC004 |
|---|---|---|
| `NB-1009` | `0` | Does not appear |
| `NB-1010` | `sold out` | Does not appear |
| `NB-1007` | `Yes` | Appears |
| `MON-3002` | empty | Appears, with a stock-not-reported notice |
| `SRV-5001` | empty | Appears; its stock is not evaluated |

And the query for 32 GB notebooks returns `NB-1005`, `NB-1007`, and `NB-1012`.

Spec Kit 1.0.8 also includes `/speckit-converge`, which reviews the existing code and adds as tasks the work still missing to meet the Spec. It is useful as an extra review after implementation, but it does not replace the audit: converge looks from the Spec to the code, and drift also travels in the opposite direction.

When the implementation passes the tests, run the audit again with `10.2`, this time with the initial report as input. Every finding should be marked as resolved. Only then commit and merge:

```bash
git add src/ tests/ specs/
git commit -m "SPEC004: hide out-of-stock products per spec"
git switch main
git merge 004-hide-out-of-stock
```

The branch history tells the whole story: first the patch, then the audit, and finally the Spec that fixes it. That is traceability too.

## A view of the current requirements

With SPEC004 merged, the *flow-forward* mode shows its cost. To know what ProductChat must do today, you have to read four Specs in order and apply each replacement. A developer can do that. A Product Owner or a client who asks *"what does the system do today with out-of-stock products?"* should not have to.

The solution is not to edit SPEC001, because that would mean changing the way of working, but to generate a **consolidated view**: a derived document that gathers the current requirements and cites which Spec each one comes from. It is generated from `main` after each merge and is not edited by hand. If it says something wrong, the error is in a Spec, and it is fixed there.

```text
ROLE
Act as the technical writer for ProductChat. You consolidate requirements that
have already been decided; you do not modify them, interpret them, or resolve
contradictions.

CONTEXT
ProductChat works in flow-forward mode: each Spec in specs/ contains only its
change and states which criteria of earlier Specs it replaces. There is no
single view of what the product must do today. This view will be read by
people who do not work with Spec Kit, such as the Product Owner or the client.

INPUT
The spec.md files in specs/ on the main branch, in numbering order, and
specs/README.md.

EXPECTED OUTPUT
The file specs/current-requirements.md, with four parts:
1. A header with the date and the main commit it was generated from.
2. The current functional requirements, business rules, and acceptance
   criteria, grouped by topic and written in business language. Each one
   cites its origin (for example, SPEC004: AC1) and, if it replaces another
   one, which.
3. The replaced requirements, each with the Spec that replaced it.
4. The conflicts: points where two current Specs say different things without
   either one declaring that it replaces the other.

CONSTRAINTS
Use only what the Specs say: do not fill gaps from the plan or the code. A
requirement is replaced only if a later Spec declares so; otherwise both
remain current and, if they contradict each other, they go under conflicts.
Do not modify any spec.md. The file is derived: it is regenerated in full, not
edited by hand.

ACCEPTANCE CRITERION
Every line cites its source Spec. AC1 appears in its SPEC004 version
(NB-1005, NB-1007, and NB-1012), and the SPEC001 version appears among the
replaced requirements. No criterion from SPEC001, SPEC002, or SPEC003 that
has not been replaced is missing.
```

> This prompt is in `prompts/10.5-current-requirements.txt`, ready to copy. If you change it, change it there: that file is the reference version.

You check the result with the module's own case: AC1 must appear in its SPEC004 version, and the SPEC001 version among the replaced requirements. The conflicts section is the most valuable part of the document. If something shows up there, two current Specs say different things, one of the kinds of drift from the beginning of the module, and it is resolved with a new Spec that declares which one applies.

```bash
git add specs/current-requirements.md
git commit -m "docs: regenerate current requirements view"
```

## How to prevent it

The practice that prevents the most drift is boring: every acceptance criterion has a test. If `AC5` had been a test, the patch would have failed on the first attempt and the drift would never have reached the history.

The second is to audit after every urgent change, not at the end of the sprint. Prompt `10.2` does not depend on this case: it works just as well for any branch.

## Hands-on activity

Reproduce the whole case in your ProductChat: create the branch, apply the patch with `10.1`, check the case table against your implementation, audit, formalize SPEC004, fix, and audit again.

| What you deliver | How it is checked |
|---|---|
| Diagnosis | The initial report reproduces each finding with a SKU |
| Decision | SPEC004 declares which criteria of SPEC001 change and why |
| Fix | The expected results table is met in full |
| Verification | The second report marks every initial finding as resolved |
| Traceability | The history shows patch, Spec, and fix in that order |
| Current view | `specs/current-requirements.md` shows AC1 in its SPEC004 version and records no conflicts |

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
