# Implementation in SDD

## What is Implementation?

**Implementation (Implement)** is the step in which the artifacts developed during Spec-Driven Development are finally turned into executable code. It is not a fifth stage: the four stages produce artifacts and are already finished; this is the execution of what those artifacts define.

Unlike traditional development, we do not start programming directly from an idea or a prompt. Before getting here, we have progressively built the context we need:

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

At this stage, implementation can largely be delegated to the **AI agent**, because the main decisions have already been made and documented.

## Implementation with GitHub Spec Kit

In **GitHub Spec Kit**, we start this step with:

```text
/speckit-implement

ROLE
Act as a developer. Implement what is specified, no more and no less.

CONTEXT
ProductChat, with the artifacts already reviewed by Analyze and corrected.

INPUT
tasks.md as the main guide, plus spec.md, plan.md, data-model.md, and
.specify/memory/constitution.md. The data file is product_catalog.xlsx.

EXPECTED OUTPUT
The application code, with the tasks in tasks.md checked off as they are
completed. For each task: implement it, run its verification, and fix it
until it passes.

CONSTRAINTS
Do not modify product_catalog.xlsx under any circumstances. Do not implement
anything that does not have a task in tasks.md; if you notice that something
is missing, stop and tell me instead of adding it. If a task turns out to be
ambiguous, ask before deciding.

ACCEPTANCE CRITERION
The application starts. Every acceptance criterion in spec.md can be checked
manually. The file product_catalog.xlsx has the same modification date as
before you started.
```

> This prompt is in `prompts/07.1-implement.txt`, ready to copy. If you change it, change it there: that file is the reference version.


The agent uses the artifacts generated earlier, especially:

```text
spec.md
plan.md
tasks.md
data-model.md
contracts/
research.md
quickstart.md
```

Not every project will necessarily have all of these files.

The central element is `tasks.md`. The agent works through the defined tasks, respects their dependencies, and progressively starts building the application.

## Executing the tasks

For ProductChat, we could have tasks that come from refining our user stories:

```text
[US1.1] Search products by name.
[US1.2] Search products by category.
[US1.3] Search products by features.
[US1.4] Return multiple results.

[US2.1] Look up the available attributes of a product.
[US2.3] Report when a requested attribute is not available.

[US3.1] Detect queries about nonexistent products.
[US3.2] Inform the user when no products are found.
[US3.3] Avoid answers with information that does not exist in the source.

[US4.1] Apply the category equivalences defined in BR2.
[US4.2] Convert memory units according to BR3.
[US4.3] Discard conditional attributes according to BR4.
[US4.4] Report duplicate SKUs with conflicting values according to BR5.
[US4.5] Indicate the source SKU in every answer.
```

This list is no longer the one `/speckit-tasks` generated: it includes the corrections that Analyze recommended in the previous module. `US2.2` is gone, merged into `US2.1`, and the missing validations for `US3.2` and `US4.4` were added to the corresponding phase. That is the effect of the quality gate: you implement the corrected version, not the first one that came out.

During implementation, the agent completes these units of work and checks them off in `tasks.md`.

The traceability we built during the previous stages then reaches all the way to the code:

```text
Requirement
   ↓
User Story
   ↓
Refined story
   ↓
Plan
   ↓
Task
   ↓
Code
   ↓
Verification
```

## The agentic loop

An important feature of this stage is that the agent should not just generate code once. For each task, it can run a small **agentic loop**:

```text
  Analyze the task
         ↓
       Plan
         ↓
     Implement
         ↓
   ┌→   Verify
   │       ↓
   │   Does it pass?
   │    ↙      ↘
   │   No        Yes  →  Task completed
   │    ↓
   └─  Fix
```

For example, for:

```text
[US1.2] Implement product search by category.
```

the agent can identify the affected components, implement the functionality, run the corresponding tests, and check that the behavior satisfies the defined requirements. In ProductChat, that verification is concrete and has a known correct answer: the search by category must also return the machines entered as `Laptops` and `Laptop`, not only those entered as `Notebooks`.

If the verification fails, it goes back to the implementation until the problem is solved.

That is why the quality of the previous stages is essential. The clearer the Specification, the Plan, and the Tasks are, the fewer decisions the agent will have to improvise during these cycles.

## Parallel execution

When `tasks.md` identifies independent tasks, some of them can be executed in parallel.

For example:

```text
[P] [US1.1] Implement search by name.
[P] [US1.2] Implement search by category.
[P] [US1.3] Implement search by features.
```

The `[P]` marker indicates that those tasks *can* be executed in parallel, not that they will be. Spec Kit does not execute anything on its own: `/speckit-implement` gives the agent the implementation workflow, and it is the agent that decides whether to delegate work to subagents. Claude Code and Codex both support subagents, but in both cases you have to ask for delegation explicitly, by adding an instruction like this one to the implementation prompt:

```text
Before phase 3, review the tasks marked [P]: check that they do not
depend on each other or modify the same files, and delegate to
subagents only the groups that are truly independent. Wait for all of
them to finish, run the tests, and check the acceptance criteria
before moving on to phase 4.
```

The main agent is still responsible for the result: it coordinates the work, waits for the subagents, and verifies the integration before continuing. That is why it is better to delegate groups of independent tasks rather than automatically create one subagent for every `[P]`. Independence is measured by files: Spec Kit's rule is that tasks that modify the same files are executed in sequence. The three searches in the example will probably end up in the same code module, and in that case they will run one after another even though they are marked `[P]`.

If you need full control over how many agents work and on what, the alternative is to open one session per group of tasks, each in its own copy of the repository, and integrate them afterward. That approach is developed in module 11.

This means `tasks.md` is not just a work list, but also a structure that helps coordinate agentic execution.

## Automatic and manual verification

Having every task marked as completed does not mean development is finished. Once automatic implementation is done, we must **run the application and validate it**.

In ProductChat, we should check manually, among other things:

```text
Can I look up a product by name?

When I ask about notebooks, do the ones entered as Laptops show up too?

When I ask about notebooks with 32 GB, do the correct four come up?

Is NB-1003, which is only expandable to 32 GB, left out?

When I ask for the price of the ThinkBook 14, does it warn me about the conflict in NB-1001?

When I ask for the price of the Monitor LG UltraWide, does it say the price is not available?

When I ask about the stock of NB-1007, whose value is Yes, does it answer that it is available?

Does every answer state the SKU the data came from?

What happens when the product does not exist?

Does the chatbot avoid making up products or features?
```

Each of these questions has a correct answer that we can check by opening `product_catalog.xlsx`. That is the difference between a manual validation and a general impression that the application works.

This validation lets us detect problems that the automatic verifications may have missed, especially those related to interaction, presentation, and user experience.

## Implementation does not close the cycle

**Implement is the last stage of this first round, but not necessarily the end of development.**

When you run and use the application, errors, behaviors that could be improved, or new needs may appear.

For example, while validating ProductChat we might discover that:

```text
The presentation of multiple products is not clear enough.

We want to add a home page.

We need to improve the graphical interface.

A certain query is not interpreted correctly.

A new inconsistency appears in the spreadsheet that no rule covers.

We want to add a new feature.
```

The fifth case is the most likely in a project like this one, and also the one that most easily sends us back to Vibe Coding. If next week someone enters a product with the RAM in gigabytes written with a decimal comma, the temptation is to patch the normalization code. The right thing to do is to add the rule to the Specification and let the change flow down through the cycle: it is a new business rule, not an implementation bug.

Here it is important to distinguish between **fixing an incorrect implementation** and **changing what we want to build**.

If the application does not meet something that was already specified, we fix the corresponding implementation.

If we discover a new need, we must bring it into the SDD cycle:

```text
New need
      ↓
Update Specification
      ↓
Review Plan
      ↓
Update Tasks
      ↓
Analyze
      ↓
Implementation
      ↓
Validation
```

This way we avoid falling back into Vibe Coding by making changes directly through prompts on the code without updating the artifacts that are the project's source of truth.

## SDD as a cycle

The complete process can finally be understood as:

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
     ↓
Validation
     ↓
New requirements / fixes
     ↺
```

The first implementation does not have to be final. **Spec-Driven Development is iterative**: each cycle lets the product evolve while keeping the requirements, the technical decisions, the tasks, and the code aligned.

The fundamental difference is that iterating does not simply mean asking the AI to modify the code. When the product changes, **we first update its specification and propagate the change in a controlled way down to the implementation**.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
