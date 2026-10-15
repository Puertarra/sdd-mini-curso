# Spec-Driven Development Mini Course

**Andrés Leiva-Araos, Ph.D.** · 
<br>School of Engineering, Universidad del Desarrollo

Versión en español: [`../README.md`](../README.md)

Material for a mini course on **Spec-Driven Development (SDD)** for undergraduate engineering students. The course uses **GitHub Spec Kit** as the method layer and **Claude Code** or **Codex** as the agent layer, and it has two parts:

- **Introduction to SDD (modules 00 to 08):** the full SDD cycle, from the project rules to the code.
- **SDD — Advanced Topics (modules 09 to 12):** what happens once the product exists and starts to change.

## The case

The whole course works on a single case that moves forward module by module.

**ProductChat** is an internal chatbot for **Distribuidora Andes**, a company that sells IT equipment. Its catalog of products and services lives in an Excel spreadsheet maintained by hand, with duplicate SKUs that contradict each other, categories spelled thirteen different ways, attributes without a consistent unit, prices that are sometimes text, and data hidden in notes columns. Nobody is going to clean it up before the chatbot is built.

That mess is not an accident. It is what forces the Specification to decide things that a Specification over clean data would never have to decide, and it is the thread that runs through all thirteen modules. In the second part, the same catalog keeps producing the hard cases: a price sitting exactly on the boundary of a filter, a stock patch that hides products that are available, and two capabilities that interpret the same duplicate SKU differently.

## Contents

### Part one: Introduction to SDD

| Module | Contents |
|---|---|
| [`00-Work_environment.md`](00-Work_environment.md) | The two layers, installing uv and Spec Kit 1.0.8, creating the project |
| [`01-SDD.md`](01-SDD.md) | What SDD is, SDD versus Vibe Coding, the course case, vocabulary and the prompt contract |
| [`02-Constitution.md`](02-Constitution.md) | Stage 1: the permanent rules of the project |
| [`03-Specification.md`](03-Specification.md) | Stage 2: what the product must do. Includes the Clarify gate |
| [`04-Plan.md`](04-Plan.md) | Stage 3: how it will be built |
| [`05-Tasks.md`](05-Tasks.md) | Stage 4: small, verifiable units of work |
| [`06-Consistency_analysis.md`](06-Consistency_analysis.md) | Analyze gate: consistency across all the artifacts |
| [`07-Implementation.md`](07-Implementation.md) | Execution: from tasks to code, and manual validation |
| [`08-Post-implementation_changes.md`](08-Post-implementation_changes.md) | A second iteration without falling back into Vibe Coding |

### Part two: SDD — Advanced Topics

| Module | Contents |
|---|---|
| [`09-Git_and_traceability.md`](09-Git_and_traceability.md) | Version control in the service of traceability: one branch per Spec, commits per stage, and context for the agent |
| [`10-Spec_Drift.md`](10-Spec_Drift.md) | When the code and the Spec stop saying the same thing: provoking drift, auditing it in both directions and resolving it. How Specs evolve, and a view of the current requirements |
| [`11-Parallel_work.md`](11-Parallel_work.md) | Two agents, two capabilities, and the conflicts of meaning that Git cannot see |
| [`12-Reverse_Specs.md`](12-Reverse_Specs.md) | Rebuilding the Spec of an existing system. Final activity of the course |

### Annex

| File | Contents |
|---|---|
| [`A-Git_mini_manual.md`](A-Git_mini_manual.md) | The Git commands the course uses, with ProductChat examples, conventions, and how to undo mistakes |

The modules are meant to be read in order. Module `01` contains the vocabulary and the prompt contract that all the others use, so don't skip it. The second part starts from the ProductChat you have implemented by the end of the first part.

## Case files

| File | What it is |
|---|---|
| [`napkin_spec_productchat.md`](napkin_spec_productchat.md) | The Specification written by hand, before using any tool. It is the input to `/speckit-specify`, not its output |
| `product_catalog.xlsx` | The Distribuidora Andes catalog, defects included. Sheet `Catalog`, header in row 3 |
| [`prompts/`](prompts/) | The 34 prompts of the course, ready to copy |

Copy the first two into your Spec Kit project before you start, because the prompts refer to them by their path. The acceptance criteria throughout the course cite SKUs from the spreadsheet: if you modify it, they no longer match.

## The cycle

```text
Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation
```

Four **stages** that produce artifacts, two **quality gates** that review the existing artifacts without producing any new one, and one **execution** that turns everything before it into code. Module `01` develops this distinction.

## The prompts

Every prompt in the course states five things: the **role**, the **context** of the case, the **input** named by its path, the **expected output**, and an **acceptance criterion** you can verify without reading code. And they chain together: the output of each one is the declared input of the next. That property is what turns SDD into a method rather than a series of unrelated requests.

There is a single exception, and it is deliberate: `10.1` represents the urgent request without method that produces drift, and it says so in its first line.

The number of each prompt indicates the module where it is used and its order within that module: `03.2-clarify.txt` is the second prompt of module 03. The modules reproduce each prompt in full, but the reference version is always the file in `prompts/`.

| Prompt | Module | Purpose |
|---|---|---|
| [`02.1-constitution.txt`](prompts/02.1-constitution.txt) | 02 | Create the ProductChat Constitution |
| [`03.1-specify.txt`](prompts/03.1-specify.txt) | 03 | Formalize the napkin spec as SPEC001 |
| [`03.2-clarify.txt`](prompts/03.2-clarify.txt) | 03 | Detect pending decisions in a Spec. Reused in the second part |
| [`04.1-plan.txt`](prompts/04.1-plan.txt) | 04 | Plan, including how to handle messy data |
| [`05.1-tasks.txt`](prompts/05.1-tasks.txt) | 05 | Break the plan down into verifiable tasks. Reused in the second part |
| [`06.1-analyze.txt`](prompts/06.1-analyze.txt) | 06 | Audit consistency across all the artifacts. Reused in the second part |
| [`07.1-implement.txt`](prompts/07.1-implement.txt) | 07 | Implement the tasks without modifying the catalog. Reused in the second part |
| [`08.1-change-specify.txt`](prompts/08.1-change-specify.txt) | 08 | Template: specify a change to a product that is already implemented |
| [`08.2-checklist.txt`](prompts/08.2-checklist.txt) | 08 | Template: thematic checklist |
| [`08.3-change-plan.txt`](prompts/08.3-change-plan.txt) | 08 | Template: plan a change without redesigning what exists |
| [`09.1-git-init.txt`](prompts/09.1-git-init.txt) | 09 | Create the repository and its first commit |
| [`09.2-agent-context.txt`](prompts/09.2-agent-context.txt) | 09 | Create `CLAUDE.md` or `AGENTS.md` |
| [`09.3-filters-specify.txt`](prompts/09.3-filters-specify.txt) | 09 | Specify the category and price filters (SPEC003) |
| [`09.4-filters-plan.txt`](prompts/09.4-filters-plan.txt) | 09 | Plan SPEC003 on top of the existing code |
| [`09.5-traceability.txt`](prompts/09.5-traceability.txt) | 09 | Demonstrate the requirement → plan → task → test → commit chain |
| [`09.6-refine-existing-spec.txt`](prompts/09.6-refine-existing-spec.txt) | 09 | Refine an open Spec without creating another one |
| [`10.1-drift-counterexample.txt`](prompts/10.1-drift-counterexample.txt) | 10 | Provoke drift on purpose. It does not follow the contract, and says so |
| [`10.2-drift-audit.txt`](prompts/10.2-drift-audit.txt) | 10 | Audit in both directions. Also verifies the resolution |
| [`10.3-out-of-stock-specify.txt`](prompts/10.3-out-of-stock-specify.txt) | 10 | Formalize the stock rule (SPEC004) |
| [`10.4-out-of-stock-plan.txt`](prompts/10.4-out-of-stock-plan.txt) | 10 | Fix the patch from the plan |
| [`10.5-current-requirements.txt`](prompts/10.5-current-requirements.txt) | 10 | Generate the consolidated view of the current requirements, for people who do not read the Specs |
| [`11.1-distribution.txt`](prompts/11.1-distribution.txt) | 11 | Split the work and assign an owner to each shared rule |
| [`11.2-export-specify.txt`](prompts/11.2-export-specify.txt) | 11 | Specify the CSV export (SPEC005) |
| [`11.3-summary-specify.txt`](prompts/11.3-summary-specify.txt) | 11 | Specify the email summary (SPEC006) |
| [`11.4-cross-review.txt`](prompts/11.4-cross-review.txt) | 11 | Detect conflicts of meaning between Specs |
| [`11.5-agent-plan.txt`](prompts/11.5-agent-plan.txt) | 11 | Plan a single capability within the team |
| [`11.6-integration.txt`](prompts/11.6-integration.txt) | 11 | Integrate branch by branch, with tests after each merge |
| [`12.1-explore.txt`](prompts/12.1-explore.txt) | 12 | Map of the system and the flow of a query |
| [`12.2-rules.txt`](prompts/12.2-rules.txt) | 12 | Rebuild the rules with evidence |
| [`12.3-anomalies.txt`](prompts/12.3-anomalies.txt) | 12 | Record and classify odd behaviors |
| [`12.4-interview.txt`](prompts/12.4-interview.txt) | 12 | Prepare and record the business interview |
| [`12.5-characterization.txt`](prompts/12.5-characterization.txt) | 12 | Pin down the current behavior with tests |
| [`12.6-reverse-spec.txt`](prompts/12.6-reverse-spec.txt) | 12 | Write the Spec of the existing system |
| [`12.7-spec-pack.txt`](prompts/12.7-spec-pack.txt) | 12 | Assemble the final deliverable |

The second part reuses `03.2`, `05.1`, `06.1` and `07.1` unchanged for Clarify, Tasks, Analyze and Implementation: they name the artifacts of the active Spec, and Spec Kit knows which one that is.

## Requirements

- A coding agent: Claude Code or Codex.
- [uv](https://docs.astral.sh/uv/), to install Spec Kit.
- Git 2.28 or later, from module 09 onward. Annex `A` summarizes the commands you need.
- GitHub Spec Kit **v1.0.8**:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v1.0.8
```

> ⚠️ **Check the version before installing.** All the material is pinned to v1.0.8 and
> tested against it. Spec Kit evolves quickly, and the way its operations are invoked is not
> stable: in 1.0.8 it depends on the agent, and it does not match the notation used in its own
> documentation. Review the repository releases before installing. If you choose a version
> other than v1.0.8, the prompts in `prompts/` may need adjustments; module `00` explains
> how to check the actual names in your installation.

In v1.0.8, operations are invoked like this:

| Agent | Syntax |
|---|---|
| Claude Code | `/speckit-specify` |
| Codex | `$speckit-specify` |

The course uses the Claude Code syntax. With Codex, just replace the slash with a dollar sign. The dot notation, `/speckit.specify`, is the one the official documentation uses as a reference; it is not what you type in the agent.

If you teach the course, repeat this check before each offering and update the pinned version if needed, instead of assuming that what worked last semester still works.

## Authorship

Material developed by **Andrés Leiva-Araos**, School of Engineering, Universidad del Desarrollo.

## License

This material is licensed under [Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/) (CC BY-SA 4.0). You may use, adapt and redistribute it, including for commercial purposes, as long as you credit the author and distribute derivative works under the same license. See [LICENSE](LICENSE).

Suggested citation:

> Leiva-Araos, A. (2026). *Spec-Driven Development Mini Course*. School of Engineering, Universidad del Desarrollo. CC BY-SA 4.0.
