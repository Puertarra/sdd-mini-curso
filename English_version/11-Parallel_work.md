# Parallel Work in SDD

Up to this point, one agent did all the work, one Spec after another. With two independent capabilities (exporting a query result to CSV and summarizing it as text for an email to the customer), the natural temptation is to open two sessions and let each agent move forward on its own. It can work, but it pays to know in advance where it breaks.

The main risk is not the conflicts Git detects. It is the ones Git cannot see: two agents that understand the same catalog rule differently and produce results that contradict each other, even though each one passes its own tests. This module is about how to avoid that.

A warning before you start: adding agents does not guarantee finishing sooner. Parallelizing saves time only if the tasks are truly independent and if coordinating and integrating costs less than doing them in sequence. In a project the size of ProductChat, the lesson is worth more than the time saved.

## What can be parallelized and what cannot

Independence is not judged by the names of the tasks, but by what they share. Two tasks with different names can depend on the same rule, the same component or the same file:

| Work | In parallel? |
|---|---|
| Deciding how to handle a SKU with two prices | No. It is a common rule: it is decided once, before distributing the work |
| Writing two Specs that consume the same result | Yes, with a cross-review before implementing |
| Implementing two capabilities in separate files | Yes, if each agent works in its own directory |
| Modifying the catalog normalizer | No. A single owner, because everything depends on it |

## The two capabilities

ProductChat is adding two ways to take the result of a query out of the chat:

| Spec | Capability | Owner |
|---|---|---|
| SPEC005 | Export the result to CSV, to work on it in a spreadsheet | Agent A |
| SPEC006 | Summarize the result as text, to paste it into an email to the customer | Agent B |
| | Common rules, cross-review and integration | You, as coordinator |

Both capabilities consume the same result, and that result already applies rules that belong to other Specs: price conflicts and duplicates belong to SPEC001, filters to SPEC003 and stock to SPEC004. Neither of the two new capabilities should redefine them. That is the point where parallel work usually breaks, so the first step is to put it in writing:

```text
ROLE
Act as the coordinator of a team of two agents. You distribute work and
assign owners; you do not modify files.

CONTEXT
ProductChat is adding two capabilities that consume the same query result:
exporting it to CSV (SPEC005) and summarizing it as text for an email to the
customer (SPEC006). That result already applies rules defined in other Specs:
price conflicts and duplicates (SPEC001), filters (SPEC003) and stock
(SPEC004). The two capabilities will be implemented by different agents.

INPUT
The Specs in specs/, the code, the tests and CLAUDE.md.

EXPECTED OUTPUT
Two tables. The first, of common rules: each rule that both capabilities
consume and the single Spec that owns that rule. The second, of work: Spec,
owning agent, branch, own files, shared files with their single owner,
dependencies and required tests.

CONSTRAINTS
Distinguish between reading a Spec and having its code available. If a common
component needs changes, assign it to a single agent.

ACCEPTANCE CRITERION
Each common rule has exactly one owning Spec, and no file has two owners.
```

> This prompt is in `prompts/11.1-distribution.txt`, ready to copy. If you change it, change it there: that file is the reference version.

## Why the coordinator creates the Specs

It seems natural for each agent to create its own Spec. With Spec Kit 1.0.8 that brings two problems, and both are concrete.

The first is the **active Spec**. Spec Kit stores in `.specify/feature.json` which Spec is in progress, and there is a single file per working directory. If two agents run `/speckit-specify` in the same directory, the second one overwrites the first one's active Spec, and the first agent's next commands end up working on the wrong Spec.

The second is **numbering**. Spec Kit assigns each Spec the next free number it finds in the `specs/` folder of the branch where it runs. A branch created from `main` does not contain the Specs that live on other branches, so two Specs created on different branches get the same number, 005. The collision only shows up at integration time.

The simplest way out is for the coordinator to create both Specs, one after the other, in the main directory, and to set each one's directory in the prompt instead of letting Spec Kit choose the number. Version 1.0.8 respects the directory if you give it as `SPECIFY_FEATURE_DIRECTORY`, and both prompts do so. Each Spec, on its own branch:

```bash
git switch -c 005-csv-export
```

```text
/speckit-specify

ROLE
Act as a requirements analyst. You specify a capability that consumes rules
already defined; you do not redefine them.

CONTEXT
The salesperson wants to download the result of a ProductChat query as CSV
to work on it in a spreadsheet. The result already applies the rules of
SPEC001 (price conflicts and duplicates), SPEC003 (filters) and SPEC004
(stock).

INPUT
The spec.md files of SPEC001, SPEC003 and SPEC004, and product_catalog.xlsx.

EXPECTED OUTPUT
The spec.md of SPEC005, in the directory specs/005-csv-export (use it as
SPECIFY_FEATURE_DIRECTORY): a CSV with one row per product in the result and
the columns SKU, name, category, price, stock status and warning.

CONSTRAINTS
Export exactly the result shown: do not query the catalog again or select
products again. If a rule from the earlier Specs is not enough to decide how
to export a case, record it as an open question; do not invent a new rule.
An empty result generates no file, and the user is notified.
Out of scope: other formats, sending the file and modifying the catalog.

ACCEPTANCE CRITERION
The Spec defines how NB-1001, with two conflicting prices, and SRV-5001,
whose stock does not apply, are exported, citing the source rule of each case.
```

> This prompt is in `prompts/11.2-export-specify.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Clarify the Spec with `prompts/03.2-clarify.txt`, commit it and go back to `main` for the second one:

```bash
git add specs/
git commit -m "SPEC005: specify CSV export"
git switch main
git switch -c 006-email-summary
```

```text
/speckit-specify

ROLE
Act as a requirements analyst. You specify a capability that consumes rules
already defined; you do not redefine them.

CONTEXT
The salesperson replies to customers by email and today copies by hand the
data ProductChat gives them. They want a text summary of a query result,
ready to paste. The result already applies the rules of SPEC001 (price
conflicts and duplicates), SPEC003 (filters) and SPEC004 (stock).

INPUT
The spec.md files of SPEC001, SPEC003 and SPEC004, and product_catalog.xlsx.

EXPECTED OUTPUT
The spec.md of SPEC006, in the directory specs/006-email-summary (use it as
SPECIFY_FEATURE_DIRECTORY): plain text with one line per product in the
result, with name, SKU, price and availability, plus any applicable
warnings.

CONSTRAINTS
Summarize exactly the result shown: do not query the catalog again or select
products again. If a rule from the earlier Specs is not enough to decide how
to present a case, record it as an open question; do not invent a new rule.
Out of scope: sending the email, writing the full message and modifying the
catalog.

ACCEPTANCE CRITERION
The Spec defines how NB-1001, with two conflicting prices, and MON-3001 and
MON-3004, which are the same monitor under two SKUs, appear, citing the
source rule of each case.
```

> This prompt is in `prompts/11.3-summary-specify.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Clarify and commit it the same way as the previous one. With both Specs on their branches, they can now be reviewed in parallel.

## Text conflicts and meaning conflicts

A **text conflict** happens when Git cannot combine the changes to a file: two branches modified the same lines. Git detects it and forces you to resolve it.

A **meaning conflict** happens when two Specs, or two implementations, interpret the same rule differently. Git does not detect it, because the files do not overlap. The merge comes out clean and the result is wrong.

In ProductChat they appear easily, because the catalog is full of cases that force interpretation:

| SKU | Situation | If the CSV… | And the summary… |
|---|---|---|---|
| `NB-1001` | Two conflicting prices, 899.99 and 949.99 | exports only the first one | says "price to be confirmed" |
| `MON-3001`, `MON-3004` | The same monitor under two SKUs | exports both rows | shows a single line "to avoid repetition" |
| `SRV-5001` | Service: stock does not apply | writes stock 0 | does not mention stock |

In all three cases, the customer receives an email that does not match the salesperson's spreadsheet. And in all three, one of the two capabilities redefined a rule that already had an owner: BR5 and EC5 of SPEC001, and the services rule of SPEC004.

To review the two Specs you do not need to switch branches. Git lets you read a file from another branch directly:

```bash
git show 005-csv-export:specs/005-csv-export/spec.md
```

Keep in mind that `git show` only sees what has been recorded in a commit: an uncommitted change in another directory does not show up.

```text
ROLE
Act as a consistency reviewer across Specs. You detect conflicts; you do not
edit documents or decide how to resolve them.

CONTEXT
SPEC005 (CSV export) and SPEC006 (email summary) were written separately
and consume the same query result. The rules that result applies belong to
SPEC001, SPEC003 and SPEC004.

INPUT
The spec.md files of SPEC005 and SPEC006, read from their branches with
git show BRANCH:PATH, and the spec.md files of SPEC001, SPEC003 and SPEC004
on main. Record the commit of each branch you review.

EXPECTED OUTPUT
A table of findings with: concept, what each consuming Spec says, what the
Spec that owns the rule says, the SKU that reproduces it and a proposed
resolution.

WHAT TO LOOK FOR
Rules redefined in a consuming Spec. Cases that one Spec handles and the
other does not. In particular: conflicting prices, empty stock, services and
products duplicated under two SKUs.

CONSTRAINTS
Do not switch branches or edit files. Distinguish a confirmed conflict from
a risk that cannot be confirmed yet.

ACCEPTANCE CRITERION
Each finding is reproduced with a SKU from the catalog, and the proposed
resolution points to the Spec that must change.
```

> This prompt is in `prompts/11.4-cross-review.txt`, ready to copy. If you change it, change it there: that file is the reference version.

## Every rule has an owner

The resolution follows a simple principle: **a common rule lives in a single Spec, and the others reference it**. The Spec that redefined it gets corrected; the owning Spec does not change.

If the CSV draft exports only the first price of `NB-1001`, the owner of SPEC005 corrects their Spec to apply BR5 of SPEC001: both prices, with the conflict warning. They can do it with Clarify or by editing the Spec, and then they review the effects on their plan and their tests, if those already exist. They do not touch SPEC006 or SPEC001: each agent corrects only its own document.

If, while reviewing, you discover that both capabilities need something the query result does not deliver yet, for example a field that gives the already-interpreted stock status, neither of the two agents resolves it. It is a change to the common component, and it is done first, with a single owner, before the consumers use it.

## Planning and implementing

Dependencies determine what can move forward at the same time:

```text
SPEC001 + SPEC003 + SPEC004, integrated and verified on main
                        ↓
               Common query result
                ┌───────┴───────┐
                ↓               ↓
           SPEC005           SPEC006
          CSV export      email summary
                └───────┬───────┘
                        ↓
           Integration and joint tests
```

With the Specs reviewed, each agent plans its own:

```text
/speckit-plan

ROLE
Act as the architect of a single capability within a team. You plan your
Spec; you do not touch the other agent's work.

CONTEXT
You work on the branch and in the folder the coordinator assigned to you.
Your capability consumes the query result that ProductChat already builds.

INPUT
Your Spec's spec.md, as revised after the cross-review, the coordinator's
distribution table, the plan.md of SPEC001 and CLAUDE.md.

EXPECTED OUTPUT
Your Spec's plan.md, which reuses the existing stack and query result, lists
the files you will create or modify and defines the tests with the SKUs from
your acceptance criteria.

CONSTRAINTS
Modify only the files the table assigns to you. If you need to change a
shared component, inform the coordinator instead of doing it. Do not
reimplement product selection.

ACCEPTANCE CRITERION
Every file in the plan is in your column of the distribution table.
```

> This prompt is in `prompts/11.5-agent-plan.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Then each agent continues with Tasks, Analyze and Implementation using `prompts/05.1-tasks.txt`, `prompts/06.1-analyze.txt` and `prompts/07.1-implement.txt`, on its own Spec. When it finishes, it hands over its commits, the files it modified, the tests it ran and what is still pending. Saying "done" without that evidence does not clear the way for integration.

### One directory or two

A branch is not a copy of the files. Two agents working in the same directory share the files and the active Spec, and any branch switch by one affects the other.

With a single directory, the sensible approach is to parallelize the review, which is read-only, and implement one capability after the other. You already get a good part of the benefit: the Specs were reviewed together before any code was written.

To really implement in parallel, each agent needs its own directory. Git allows this with **worktrees**: separate folders of the same repository, each with its own branch. And since each worktree has its own `.specify/feature.json`, each agent keeps its own active Spec. From the main directory, on `main` and with no pending changes:

```bash
git worktree add ../ProductChat-csv 005-csv-export
git worktree add ../ProductChat-summary 006-email-summary
git worktree list
```

Each agent opens its session in its own folder. This is an optional extension: the whole activity can be completed with a single directory.

## Integrating in order of risk

The two branches do not depend on each other, so risk decides the order. The summary only produces text; the export writes files to disk, which adds names, paths and permissions that can fail. It is best to integrate the lower-risk one first, verify, and only then the next:

```bash
git switch main
git merge 006-email-summary
```

Run the tests. If they pass:

```bash
git merge 005-csv-export
```

And run the tests again, now with both capabilities together. If the second merge produces conflicts, resolve them according to the Specs, keeping the valid work from both branches. Never by picking one branch's version wholesale, and never by deleting tests to make them pass.

```text
ROLE
Act as the integration coordinator. You integrate branches that are already
verified, one at a time; you do not expand the scope of any of them.

CONTEXT
SPEC005 and SPEC006 are implemented on their branches, each with its tests
passing. They need to be brought into main without breaking what already
works.

INPUT
The branches of SPEC005 and SPEC006, the distribution table and each
agent's test reports.

EXPECTED OUTPUT
An integration order justified by dependencies and risk, the integration
branch by branch with the tests run after each merge, and a report with
commits, resolved conflicts and pending items.

CONSTRAINTS
Confirm that main is clean before you start. Do not integrate the second
branch until the first one passes the tests. If a conflict appears, resolve
it according to the Specs; do not pick one branch's version wholesale or
delete tests to make them pass.

ACCEPTANCE CRITERION
On main, the query "Notebooks with a maximum price of 1,000" produces
the same result in the CSV and in the summary: NB-1001 with its two prices
and the conflict warning, and NB-1015.
```

> This prompt is in `prompts/11.6-integration.txt`, ready to copy. If you change it, change it there: that file is the reference version.

If you used worktrees, remove them with `git worktree remove` only when they have no pending changes.

## Hands-on activity

Split SPEC005 and SPEC006 between two agent sessions. As coordinator, create the Specs, run the cross-review, resolve the conflicts you find and then integrate.

The result is verified by comparing the two outputs for the same queries:

| Query | CSV and summary must show |
|---|---|
| Notebooks, maximum 1,000 | `NB-1001` with its two prices and the conflict warning, and `NB-1015`. `NB-1010` does not appear: it is sold out |
| Services, maximum 100 | `SRV-5001`, without presenting it as "out of stock" |
| Monitors, maximum 150 | `MON-3001` and `MON-3004`, separately |

| Deliverable | How it is checked |
|---|---|
| Distribution | Each common rule has a single owning Spec |
| Cross-review | Each conflict found is reproduced with a SKU |
| Resolution | The consuming Specs reference the rule instead of redefining it |
| Integration | One merge at a time, with tests after each one |
| Consistency | The CSV and the summary match on all three queries |

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
