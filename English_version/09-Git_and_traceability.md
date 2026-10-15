# Version Control in SDD

Part one ended with ProductChat up and running. From here on the product starts to change, and that raises a question we did not need to answer until now: **how do you prove that a line of code exists because of a requirement?**

In part one we claimed many times that SDD is traceable, that every part of the code can be followed back to the specification that asked for it. Without version control, that claim is rhetoric: you can say it, but you cannot check it. Git makes it verifiable, because it stores the requirement and the code that implements it together, in the same commit.

This module uses Git only as far as SDD needs it. It is not a Git course: one branch per Spec, commits per stage and a context file for the agent are enough. If Git is new to you, or you need to recall a command, the annex [`A-Git_mini_manual.md`](A-Git_mini_manual.md) collects every command the course uses, with ProductChat examples.

## What goes under version control

In an SDD project, the code and the documents that explain why the code is the way it is are versioned together. For ProductChat:

```text
ProductChat/
├── .specify/memory/constitution.md
├── specs/
│   ├── 001-catalog-query/          ← SPEC001, part one
│   └── 002-home-page/              ← SPEC002, module 08
├── src/
├── tests/
├── product_catalog.xlsx
├── napkin_spec_productchat.md
├── CLAUDE.md
└── .gitignore
```

Spec Kit generates the names of the folders in `specs/` from the description of each Spec; yours may differ.

One file deserves a second look: the catalog. In a real project, a client's data is not uploaded to a repository. Here it is, for two reasons. The catalog is fictional. And the acceptance criteria of the Specification cite its SKUs: if the file changes without the change being recorded, `AC1` can no longer be verified and nobody knows why. Versioning it is what keeps the criteria honest.

## Setting up the repository

Spec Kit 1.0.8 **does not create a Git repository** when you run `specify init`. If you did not create one yourself at the end of part one, now is the time:

```text
ROLE
Act as the person responsible for the ProductChat repository. You set up version
control; you do not modify code or artifacts.

CONTEXT
ProductChat is already implemented: part one of the course ended with its
code, its SDD artifacts, product_catalog.xlsx and napkin_spec_productchat.md
inside the project. Spec Kit 1.0.8 does not create a Git repository when it
initializes a project.

INPUT
The root of the ProductChat project.

EXPECTED OUTPUT
A Git repository with the main branch, a reviewed .gitignore and a first commit
that includes the code, the tests, .specify/memory/constitution.md, the specs/
folder, product_catalog.xlsx and napkin_spec_productchat.md.

CONSTRAINTS
If Git is already initialized, do not reinitialize it: report its status.
Exclude virtual environments, caches and generated files. Do not exclude the
catalog: its SKUs are the reference for the acceptance criteria.

ACCEPTANCE CRITERION
git status shows no pending changes, and git show --stat HEAD lists the
catalog and the files in specs/.
```

> This prompt is in `prompts/09.1-git-init.txt`, ready to copy. If you change it, change it there: that file is the reference version.

If you prefer to do it by hand, it takes three commands:

```bash
git init -b main
git add .
git commit -m "ProductChat: base implementation with SDD artifacts"
```

Before the `add`, check the `.gitignore`: the virtual environment and the caches stay out; the catalog stays in.

## One branch per Spec

Each new Spec is developed on its own branch and merged into `main` only once it has been verified. The course convention is that the branch has the same name as the Spec's folder:

```bash
git switch -c 003-catalog-filters
```

This is our convention, not Spec Kit's. In version 1.0.8, `/speckit-specify` **does not create branches**: that only happens if you install the `git` extension, which we leave out of the course. Spec Kit does not need branches to work either: it remembers which Spec is active in `.specify/feature.json`, not by the branch name. That is why you create the branch yourself, before running `/speckit-specify`.

Inside the branch, make one commit for each step forward that makes sense on its own. That is what later lets you reconstruct the reasoning:

```text
SPEC003: specify and clarify filters      ← spec.md
SPEC003: plan and tasks                   ← plan.md, tasks.md
SPEC003: implement and test filters       ← code, tests and artifacts
```

## The agent's technical context

The agent starts every session with no memory. It can read the Constitution and the Specs, but it does not know where the catalog normalizer lives or which command runs the tests. That information goes in a context file at the root of the project: `CLAUDE.md` in Claude Code, `AGENTS.md` in Codex.

It is not one more place to write rules. Each thing has its place:

| Document | Answers | Example in ProductChat |
|---|---|---|
| Constitution | Which rules always apply? | Catalog inconsistency is shown, not hidden |
| Specification | What does this capability do? | With a maximum price of 34.99, `PER-4001` appears |
| Plan | How is this capability built? | The filters are applied to the already normalized catalog |
| `CLAUDE.md` | How do you work in this repository? | The normalizer is in `src/catalog/`; the tests run with `pytest` |

If a line in `CLAUDE.md` could be in the Constitution, it is in the wrong place.

```text
ROLE
Act as the technical maintainer of ProductChat. You document how work is done in
the repository; you do not change the system's behavior.

CONTEXT
ProductChat has a Constitution, specifications and plans, but the agent
starts each session without knowing how the code is organized or how it is
tested. That information lives in a context file: CLAUDE.md in Claude Code,
AGENTS.md in Codex.

INPUT
.specify/memory/constitution.md, the plan.md files in specs/ and the project's
code.

EXPECTED OUTPUT
CLAUDE.md (or AGENTS.md) at the root, one page at most, with: the system's
purpose in two lines, the current stack, how the code is organized, where the
catalog is normalized, verified commands to run and test, and the branch and
commit convention per Spec.

CONSTRAINTS
Reference the Constitution and the specs/ folder; do not copy their content. If
you could not verify a command, mark it as unverified. Do not make up data.

ACCEPTANCE CRITERION
Every command in the file runs without errors in the project, and the file does
not repeat any rule that is already in the Constitution.
```

> This prompt is in `prompts/09.2-agent-context.txt`, ready to copy. If you change it, change it there: that file is the reference version.

## The cycle with Git: the SPEC003 filters

With the repository ready, we go through the full cycle for a new capability. The salesperson wants to filter by category and maximum price. This is SPEC003, because SPEC002 was the home page from module 08.

```text
main ──┬────────────────────────────────────────────┬── main
       │                                       merge │
       └── 003-catalog-filters                       │
             Specification → Clarify → commit        │
             Plan → Tasks → Analyze  → commit        │
             Implementation → tests  → commit ───────┘
```

Before you start, `git status` must be clean: a pending change from something else would end up mixed into the SPEC003 commits.

### Specify

```text
/speckit-specify

ROLE
Act as a requirements analyst. You specify a new ProductChat capability; you do
not design or implement it.

CONTEXT
ProductChat queries product_catalog.xlsx, a catalog with categories written
in several ways, prices stored as text or left empty, and duplicate SKUs with
conflicting prices. The basic query (SPEC001) and the home page (SPEC002) are
already implemented. The salesperson now needs to filter by category and
maximum price.

INPUT
.specify/memory/constitution.md, the spec.md of SPEC001 and product_catalog.xlsx.

EXPECTED OUTPUT
The spec.md of the new Spec, with user stories, functional requirements,
business rules, acceptance criteria and edge cases.

RULES ALREADY DECIDED
- The category and maximum price filters can be combined.
- The maximum price includes the limit value.
- The category uses the same equivalences as the basic query.
- A product without a usable price does not meet a price filter: it is excluded
  and the response says so.
- A SKU with conflicting prices is reported as SPEC001 establishes: no price is
  chosen silently.
- A negative maximum price is invalid input, and the user is warned.

CONSTRAINTS
Do not change the behavior of SPEC001. Out of scope: sorting results,
filtering by other attributes, exporting, and modifying the catalog.

ACCEPTANCE CRITERION
The criteria cite real SKUs from the catalog. "Peripherals with a maximum price
of 34.99" returns PER-4001 and PER-4002, and "Notebooks with a maximum price
of 1,000" excludes NB-1009, reporting that it has no price.
```

> This prompt is in `prompts/09.3-filters-specify.txt`, ready to copy. If you change it, change it there: that file is the reference version.

The expected results are checked against the real catalog:

| Query | Expected result | Why |
|---|---|---|
| Peripherals, maximum 34.99 | `PER-4001`, `PER-4002` | `PER-4001` costs exactly 34.99: it tests the inclusive limit. The two are stored as `Peripherals` and `Peripheral`, so the query only works if the category is normalized |
| Notebooks, maximum 1,000 | `NB-1010`, `NB-1015`, and `NB-1001` with a conflict warning | Both prices of `NB-1001`, 899.99 and 949.99, meet the filter. `NB-1009` is left out because it has no price, and the response says so |
| Peripherals, maximum -1 | Invalid input warning | A negative price is not a filter |

Now run Clarify with the prompt `prompts/03.2-clarify.txt` from module 03. There is one case you want it to surface: with a maximum of 920, one of the two prices of `NB-1001` meets the filter and the other does not. The Spec says the conflict is reported, but it does not say whether the product appears. If Clarify does not catch it, you will find it during implementation, and the next section explains what to do then.

With the Spec clarified, record the progress:

```bash
git add specs/
git commit -m "SPEC003: specify and clarify filters"
```

### Plan

```text
/speckit-plan

ROLE
Act as an architect working on a codebase that already works. You propose the
how of SPEC003; the final decision is mine.

CONTEXT
ProductChat already normalizes categories, prices and SKU conflicts when it
reads the catalog. The filters must be applied to that normalized catalog, not
redo the normalization.

INPUT
The spec.md of SPEC003, the plan.md and data-model.md of SPEC001,
.specify/memory/constitution.md and CLAUDE.md.

EXPECTED OUTPUT
The plan.md of SPEC003, reusing the stack and data model of SPEC001 and
describing only what changes: where the filters are applied, how a missing
price is handled and how a SKU in conflict is handled against a maximum price.

CONSTRAINTS
Do not replace the stack or the data model. If any command or convention
changes, include a task to update CLAUDE.md, recording only what will be
useful to future Specs.

ACCEPTANCE CRITERION
The plan lists the components it does not touch, and every new component is
justified by a requirement of SPEC003.
```

> This prompt is in `prompts/09.4-filters-plan.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Then run Tasks and Analyze with the prompts `prompts/05.1-tasks.txt` and `prompts/06.1-analyze.txt`. They work unchanged, because they name the artifacts of the active Spec, and Spec Kit knows which one that is from `.specify/feature.json`. Fix whatever Analyze finds in the artifact where the problem lies, and record:

```bash
git add specs/
git commit -m "SPEC003: plan and tasks"
```

### Implement and merge

Implement with `prompts/07.1-implement.txt`. When the three queries in the table give the expected result, record the code, the tests and the updated artifacts in a single commit:

```bash
git add src/ tests/ specs/ CLAUDE.md
git diff --cached
git commit -m "SPEC003: implement and test filters"
```

`git diff --cached` shows exactly what is going into the commit. It is the last point where an accidental change is visible before it is recorded.

To merge, go back to `main` and bring in the branch:

```bash
git switch main
git merge 003-catalog-filters
```

After the merge, run the tests again. Git reporting no conflicts only means that the texts were combined; it does not guarantee that the behavior is still correct.

## Checking traceability

Now traceability can be demonstrated, not just claimed. For the requirement "the maximum price includes the limit value", the full chain is:

| Link | Evidence |
|---|---|
| Requirement | SPEC003 `spec.md`: a product whose price equals the maximum meets the filter |
| Decision | `plan.md`: the comparison is `price <= maximum` |
| Task | `tasks.md`: implement the filter and verify the limit |
| Test | `PER-4001`, priced at 34.99, appears with a maximum price of 34.99 |
| Commit | `SPEC003: implement and test filters` |

Git lets you walk that chain backward from any point:

```bash
git log --oneline -- specs/003-catalog-filters/
git log -p -- specs/003-catalog-filters/spec.md
```

The first lists the commits that touched the Spec. The second shows how each requirement changed and when.

```text
ROLE
Act as a traceability auditor. You review; you do not modify files.

CONTEXT
SPEC003 is implemented and merged into main. You need to show that a specific
requirement can be followed from the specification to a test and a commit.

INPUT
The SPEC003 folder in specs/ (spec.md, plan.md, tasks.md), the code, the
tests and the Git history.

EXPECTED OUTPUT
A table for the requirement "the maximum price includes the limit value" with
five rows: requirement, plan decision, task, test and commit, each with its
real path, name or hash.

CONSTRAINTS
Cite only paths, test names and hashes that exist. If a link is missing, mark
it as missing instead of assuming it.

ACCEPTANCE CRITERION
I can open every path and run git show with every hash, and the cited test
verifies that PER-4001, priced at 34.99, appears with a maximum price of 34.99.
```

> This prompt is in `prompts/09.5-traceability.txt`, ready to copy. If you change it, change it there: that file is the reference version.

## Changing a Spec or creating a new one

Not every change needs a new Spec. The course criterion is simple: **if it clarifies something in a Spec that is still open, you change that Spec; if it adds a capability, or changes something already delivered, you create a new one.**

| Situation in ProductChat | What to do |
|---|---|
| During SPEC003 the case of `NB-1001` with a maximum of 920 comes up | Refine SPEC003 |
| `PER-4001` does not appear with a maximum of 34.99 | Fix the code: the requirement is already correct |
| The salesperson wants to export the filtered results | New Spec that references SPEC003 |
| You want to change how a price conflict is shown in every query | New Spec: it changes behavior already delivered in SPEC001 |

The first case is the one we left open when specifying. This is how you refine it:

```text
ROLE
Act as a requirements analyst working on a Spec that is still in development.
You refine a requirement; you do not expand the scope.

CONTEXT
During the implementation of SPEC003 a case came up that the Spec does not
resolve: NB-1001 has two conflicting prices, 899.99 and 949.99, and with a
maximum price of 920 one meets the filter and the other does not. The
decision has already been made: the product appears, with the conflict warning
and an indication of which of its prices meets the filter.

INPUT
The spec.md, plan.md and tasks.md of SPEC003.

EXPECTED OUTPUT
spec.md updated with that rule and its acceptance criterion, plus a list of the
changes it requires in plan.md, tasks.md and the tests, separating what is
already done from what is pending.

CONSTRAINTS
Keep the identifier and the folder: do not create a new Spec. Do not delete
current plan decisions or tasks already completed.

ACCEPTANCE CRITERION
The diff only touches the SPEC003 folder, and spec.md contains a verifiable
criterion with NB-1001 and a maximum price of 920.
```

> This prompt is in `prompts/09.6-refine-existing-spec.txt`, ready to copy. If you change it, change it there: that file is the reference version.

After refining the Spec, review the artifacts that depend on it. Do not regenerate the plan and the tasks from scratch: they contain decisions and work that are still valid.

## Practical activity

Implement SPEC003 on your ProductChat from part one, following the cycle in this module.

The deliverable is the repository, with:

| What | How it is checked |
|---|---|
| Consistency | Spec, plan, tasks and code describe the same behavior |
| History | The branch has one commit per stage, and each one references SPEC003 |
| Context | `CLAUDE.md` describes the current project and its commands work |
| Verification | The three queries in the table give the expected result |
| Traceability | The table from the previous section, completed with real paths and hashes |

If your project accumulates several Specs, you can add a `specs/README.md` with one line per Spec: identifier, purpose and status. With three Specs that do not step on each other it is not necessary; module 10 shows when it becomes necessary.

> **For later: the Spec Kit `git` extension.** Spec Kit 1.0.8 includes an optional extension that automates part of what you did by hand in this module. You install it with `specify extension add git`, and it works through *hooks*, actions that run on their own before or after each operation: it creates the `NNN-name` branch before each `/speckit-specify` and, if you configure it, makes a commit before or after each stage. It is an advanced feature, and you should use it only once you have mastered the manual workflow: creating the branch, deciding what goes into each commit and reading the resulting history. Automating a process you do not yet understand produces a history you will not be able to read either. Also keep in mind that the automatic messages, such as `[Spec Kit] Add tasks`, do not say which Spec they belong to: you would have to configure them to keep the `SPEC003: …` convention. And the extension does not solve the parallel work of module 11: it creates branches, but in a single working folder.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
