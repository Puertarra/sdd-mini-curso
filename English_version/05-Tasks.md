# Tasks in SDD

## What are Tasks?

**Tasks** are the fourth stage of the **Spec-Driven Development (SDD)** cycle. Once the Plan has defined how the system will be built, this stage breaks it down into **small, concrete and verifiable units of work** that the AI agent can then execute.

Tasks mainly answer the question: **what concrete actions must we carry out to implement the Plan?**

At this stage human involvement is limited. The AI has the Constitution, the Specification and the Plan, so it can automatically generate the required sequence of work.

## What makes a good task?

Each task must represent a unit of work small enough to be implemented and checked independently. It must state clearly what has to be done and make it possible to determine when it is finished.

For example, a task that is too general would be:

```text
Implement the chatbot.
```

An appropriate breakdown would be:

```text
Create the component that accesses the product catalog.

Implement reading products from the data source.

Implement product search by name.

Implement search by category.

Implement handling of queries with no results.

Create the tests that verify product queries.
```

The goal is to eliminate tasks that are too large or ambiguous and that force the agent to make multiple decisions during their execution.

## Organizing Tasks

Tasks are usually organized into phases according to their dependencies and the logical order of implementation. One possible structure is:

```text
Phase 1: Initial setup
Phase 2: Foundational components
Phase 3: User story 1
Phase 4: User story 2
Phase 5: User story 3
Phase 6: Integration and final adjustments
```

This organization lets you first build the elements needed for the system to work in general, and then implement the different user stories defined in the Specification.

## Dependencies between tasks

Not all tasks can be executed in any order. Some need others to have finished first.

For example:

```text
Create the data access mechanism
        ↓
Implement product search
        ↓
Integrate search with the chatbot
        ↓
Validate the answers
```

The Tasks stage must identify these dependencies in order to establish a coherent execution order.

## Parallelizable tasks

When two tasks do not depend on each other, they can potentially be executed in parallel. GitHub Spec Kit can identify these tasks, usually with a marker such as **[P]**.

For example:

```text
[P] Implement search by category
[P] Implement search by name
[P] Create the product display format
```

Identifying parallelizable tasks is especially useful when you work with **several agents or subagents**, because it lets you distribute independent work and reduce implementation time.

Not every task should be parallelized. The dependencies defined in the Plan must be respected.

## Verification

A fundamental characteristic is that tasks must have a verifiable result. It is not enough to state what must be built; it must be possible to determine objectively whether the task was completed correctly.

For example:

```text
Task:
Implement product search by name.

Verification:
When querying "Acer Swift Go", the system returns NB-1007
with 32 GB of RAM, even though the spreadsheet records 32768 MB.
```

This verifiability lets you connect the implementation with the requirements and acceptance criteria defined earlier in the Specification.

You do not need to invent data to verify: the catalog itself is the test set. It was built with its defects in plain sight, and every acceptance criterion has a real row that puts it to the test. It helps to distinguish three levels. **Unit tests** check an isolated rule, such as that `32768 MB` equals 32 GB or that `Laptops` and `Notebooks` are the same family; they use values written inside the test itself, taken from the real spellings in the spreadsheet. **Acceptance tests** run on `product_catalog.xlsx` and name the SKU, just like the acceptance criteria. **Synthetic data** is only needed for cases the catalog does not contain, such as an empty file or a missing sheet; in that case, the test spreadsheet is declared in the Specification or in the Plan, and is not improvised by the agent during implementation.

## Generating with GitHub Spec Kit

In **GitHub Spec Kit**, tasks are generated with:

```text
/speckit-tasks

ROLE
Act as the person responsible for work planning.

CONTEXT
ProductChat, a chatbot over an inconsistent Excel catalog.

INPUT
plan.md, data-model.md, spec.md and .specify/memory/constitution.md.

EXPECTED OUTPUT
tasks.md with the tasks organized by phases, each task small enough to be
implemented and checked independently. Mark with [P] the tasks that can run in
parallel. Label each feature task with the refined story it implements, in the
format [US1.1].

OUTPUT REQUIREMENTS
Every refined story must have at least one implementation task and one
verification task. Verification tasks must be checkable against
product_catalog.xlsx, naming the specific SKU when applicable. Do not introduce
any task that implements something that is not in spec.md.

ACCEPTANCE CRITERION
Every functional requirement and every acceptance criterion in spec.md has at
least one associated task, and no task lacks a requirement that originates it.
```

> This prompt is in `prompts/05.1-tasks.txt`, ready to copy. If you change it, change it there: that file is the reference version.


Usually you do not need to provide a new detailed prompt. The agent uses the artifacts created during the previous stages:

```text
Constitution
      ↓
Specification
      ↓
Plan
      ↓
Tasks
```

From them it usually generates a file:

```text
tasks.md
```

This file contains the tasks organized by phases, their dependencies, possible opportunities for parallelization and the elements needed to verify their execution.

## Example for ProductChat

For ProductChat we use the previous prompt unchanged: all the information specific to the case is already in the artifacts it declares as input.

The result could contain a structure similar to:

```text
Phase 1: Setup
- Create the initial project structure.
- Initialize the required dependencies.
- Create the environment configuration.

Phase 2: Foundational components
- Implement reading the Excel file and locating the header row.
- Implement discarding empty rows and trimming whitespace.
- Create the internal product model.
- Implement normalization of categories, units, price and stock.
- Implement detection of duplicate SKUs.
- Create the common components for processing queries.

Phase 3: Features
- [US1.1] Search products by name.
- [US1.2] Search products by category.
- [US1.3] Search products by features.
- [US1.4] Get multiple products that satisfy a query.

- [US2.1] Query the available attributes of a product.
- [US2.2] Get the available features of a product.
- [US2.3] Report when a requested attribute is not available.

- [US3.1] Detect queries about nonexistent products.
- [US3.2] Inform the user when no products are found.
- [US3.3] Avoid answers with information that does not exist in the source.

- [US4.1] Apply the category equivalences defined in BR2.
- [US4.2] Convert memory units according to BR3.
- [US4.3] Discard conditional attributes according to BR4.
- [US4.4] Report duplicate SKUs with conflicting values according to BR5.
- [US4.5] Indicate the source SKU in every answer.

Phase 4: Validation
- [US1.1] Verify searches by name.
- [US1.2] Verify searches by category.
- [US1.3] Verify searches by features.
- [US1.4] Verify queries with multiple results.
- [US2.1] Verify that the attributes match the data source.
- [US2.3] Verify queries about unavailable attributes.
- [US3.1] Verify queries about nonexistent products.
- [US3.3] Verify that the chatbot does not invent products or attributes.
- [US4.1] Verify that Laptops and Notebooks are treated as one family.
- [US4.2] Verify that 32768 MB is interpreted as 32 GB.
- [US4.3] Verify that NB-1003 does not appear among the 32 GB machines.
- [US4.5] Verify that every answer indicates its source SKU.

Phase 5: Final integration
- Verify the full integration of the refined stories.
- Check the acceptance criteria of the specification.
- Verify the edge cases.
- Check compliance with the Constitution.
```

Conceptually, this lets us present the breakdown as a refinement of User Stories based on applying INVEST. **INVEST** is a mnemonic for evaluating whether a user story is well written: it must be *Independent*, *Negotiable*, *Valuable*, *Estimable*, *Small* and *Testable*. A story that is not *Small* or *Testable* is exactly the one worth refining before turning it into tasks:

```text
US1: Search products
 ├── US1.1: Search by name
 ├── US1.2: Search by category
 ├── US1.3: Search by features
 └── US1.4: Get multiple results

US2: Query the attributes of a product
 ├── US2.1: Query attributes
 ├── US2.2: Get features
 └── US2.3: Handle unavailable attributes

US3: Handle nonexistent information
 ├── US3.1: Detect nonexistent products
 ├── US3.2: Report absence of results
 └── US3.3: Avoid invented information

US4: Resolve an inconsistent catalog
 ├── US4.1: Category equivalences
 ├── US4.2: Unit conversion
 ├── US4.3: Conditional attributes
 ├── US4.4: Conflicts between duplicate SKUs
 └── US4.5: Traceability to the source SKU
```

This example `tasks.md` **is not correct**: it has a duplication and leaves three refined stories unverified. We leave it like this on purpose. In the next module, `/speckit-analyze` will find exactly those problems, which is precisely what it is for.

Obviously, the real list will be more detailed and will depend on the Plan generated earlier.

## Reviewing the Tasks

Although generation can be done automatically, it is a good idea to review `tasks.md` before starting implementation. The review should mainly check that the tasks are small enough, have a verifiable result, respect the dependencies and cover the user stories and acceptance criteria of the Specification.

You should also check that no new features have appeared. If a task implements something that does not exist in the Specification, it must be reviewed before continuing.

## From the Plan to implementation

Tasks are the bridge between the technical design and the execution. The Plan can describe relatively large components, while `tasks.md` turns them into actions that the agent can execute progressively and mark as completed.

The cycle then becomes:

`Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation`

Where:

`Constitution = rules that govern the project`

`Specification = what the system must do`

`Clarify = remove ambiguities`

`Plan = how it will be built`

`Tasks = small, verifiable actions`

Once `tasks.md` has been generated and reviewed, the project is ready for the agent to begin the **progressive implementation of the tasks**.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
