# The Plan in SDD

## What is the Plan?

The **Plan** is the third stage of the **Spec-Driven Development (SDD)** cycle. Once the Constitution has been defined, the Specification created and its ambiguities resolved through Clarify, the Plan turns the **what** into the **how**.

At this stage the AI takes on a much more active role. Starting from the rules of the Constitution and the requirements of the Specification, it analyzes the alternatives and proposes the technical decisions needed to implement the system.

The Plan mainly answers the question: **how are we going to build what we have specified?**

## What should the Plan contain?

Unlike the Specification, the Plan is where technical decisions do belong. Depending on the project, it can define aspects such as:

- Solution architecture.
- Languages and frameworks.
- Main components.
- Organization of the frontend and backend.
- Data model and storage.
- APIs and interfaces between components.
- Main dependencies.
- Testing strategy.
- Project structure.
- Deployment strategy.

These decisions must derive from real needs established in the Specification and must always respect the principles defined in the Constitution.

> **If you come from Scrum.** The Plan resembles *Sprint Planning*, but only in part. The Scrum Guide divides that meeting into three topics: why the Sprint is valuable, which backlog items go into it, and how the work will be done. The Plan corresponds to the third one, the "how". In that topic the *Developers* also break the items down into units of work of one day or less, which in SDD is the next stage, Tasks; the most exact equivalence is Plan plus Tasks. There are two differences. Sprint Planning is a time-boxed event in which the team chooses what goes in based on its capacity; in SDD that scope has already been set by the Specification. And Scrum does not require documenting technical decisions: the design emerges during the Sprint. The Plan writes them down before implementation, which is closer to the *Architecture Decision Records* proposed by Michael Nygard than to Scrum. The reason is the same one we saw in the Specification: the agent that implements was not part of the team's conversation.
>
> Nygard, M. (2011). [*Documenting Architecture Decisions*](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions). · Schwaber, K. and Sutherland, J. (2020). [*The Scrum Guide*](https://scrumguides.org/scrum-guide.html).

## The role of AI

In SDD, the Plan can be produced mainly by the **AI agent**. The human responsibility changes compared with the previous stage.

In the Specification, the person actively defines what the system must do. In the Plan, the AI proposes how to build it and the person **reviews, questions and validates the decisions**.

In short:

`Person + AI: Constitution`

`Person + AI: Specification`

`AI + human review: Plan`

This lets you take advantage of the agent's technical knowledge without delegating to it the decisions about what the business really needs.

## Additional information for generating the Plan

Although the Plan is generated from the Constitution and the Specification, you can provide additional instructions before generating it. These instructions should be brief and used only when there is a relevant criterion worth emphasizing.

For example:

```text
Prioritize simplicity as established in the Constitution.

This is a version 1 that must be easy to deploy.

Do not add infrastructure that the specification does not need.

Explain the important technical decisions also in terms a non-technical
person can understand.
```

These instructions should not be used to introduce new functional requirements. If a new requirement appears about **what the product must do**, it must first be added to the Specification.

## Generating the Plan with GitHub Spec Kit

In **GitHub Spec Kit**, the Plan is generated with:

```text
/speckit-plan
```

The agent uses the existing Constitution and Specification as context and starts designing the technical solution.

For **ProductChat** we used:

```text
/speckit-plan

ROLE
Act as a software architect. Propose the how and justify it; the final
decision is mine.

CONTEXT
ProductChat is a first version that must be easy to deploy and maintain.
The data source is product_catalog.xlsx, a spreadsheet with duplicate SKUs,
inconsistent categories, attributes without a stable unit and prices stored as
text. The spreadsheet is neither corrected nor replaced.

INPUT
spec.md, .specify/memory/constitution.md and the product_catalog.xlsx file
itself, which you can inspect to learn the real shape of the data.

EXPECTED OUTPUT
plan.md with the architecture, the stack and the project structure;
data-model.md explicitly distinguishing the raw shape of the spreadsheet from
the normalized internal model, and describing the transformation between them;
research.md justifying the decisions with alternatives.

DECISIONS YOU MUST RESOLVE EXPLICITLY
Where normalization is applied, at load time or on each query. When the Excel
file is read. How a missing value is represented versus a zero and versus a
text such as "ask". How a SKU with two conflicting values is represented
without collapsing them. And which part of the work the language model does and
which part is deterministic code.

CONSTRAINTS
Prioritize simplicity as the Constitution requires. Do not add infrastructure
that the specification does not need. Do not propose generating a clean copy of
the spreadsheet or deduplicating silently: both contradict the Constitution.
Explain every relevant decision also in terms a non-technical person can
understand.

ACCEPTANCE CRITERION
The data model can hold two different values for the same SKU. The
interpretation of units is deterministic and does not depend on the language
model. Every component that adds complexity is justified by a specific
requirement in spec.md.
```

> This prompt is in `prompts/04.1-plan.txt`, ready to copy. If you change it, change it there: that file is the reference version.

From this point on, the agent can ask questions when an important technical decision cannot reasonably be determined from the available information.

## Technical decisions during the Plan

At this stage, questions may come up about architecture, storage, integration, deployment or technologies. In ProductChat, many of those questions come not from the architecture but from the state of the catalog, and it is worth resolving them explicitly in the Plan:

- **Where normalization lives.** Rules BR2, BR3 and BR4 can be applied when the file is loaded, building a clean internal model, or on each query. The first option is simpler to reason about and to test; the second reflects sooner any changes the catalog manager makes to the spreadsheet. This is not an indifferent decision: it determines OQ5 of the specification.
- **When the Excel file is read.** Once at startup, on each query, or with an explicit reload.
- **How a missing value is represented.** An empty price, an `ask` and a `0` are not the same thing, and the internal model must be able to tell them apart so that FR5 and AC4 can be verified.
- **How a conflict is represented.** BR5 requires the internal model to be able to hold two different values for the same SKU without collapsing them. A data model that uses the SKU as a unique key makes it impossible to meet AC3, and it is the easiest mistake to make at this stage.
- **What the language model does and what it does not.** Interpreting the user's question and writing the answer is one thing; deciding whether `32768 MB` is 32 GB is another. If normalization is left in the hands of the model, the result is no longer reproducible and AC1 can no longer be verified.

If the user has technical knowledge, they can select an alternative directly. If not, they can ask the agent to recommend an option and explain the decision in business terms.

For example:

```text
I am deciding how ProductChat should represent a SKU with two different
prices, according to BR5 in spec.md. Recommend the simplest alternative
for this first version and explain its advantages and disadvantages without
assuming technical knowledge. Do not implement it yet.
```

The goal is not to avoid technical decisions, but to make them **explicit, justified and consistent with the Specification and the Constitution**.

## Generated artifacts

Depending on the version and configuration of Spec Kit, the planning stage can generate different documents inside the specification directory. They may include:

```text
plan.md
research.md
data-model.md
quickstart.md
contracts/
```

`plan.md` contains the main technical plan, including architecture, technologies, structure and implementation decisions.

`research.md` documents and justifies relevant technical decisions, especially when there are several possible alternatives.

`data-model.md` describes the entities, fields, relationships and data structures needed to implement the specification. In a case like ProductChat it must clearly distinguish **the raw shape of the source** (the columns of the spreadsheet, exactly as they are) from the **normalized internal model** on which the queries operate, and make explicit which transformation leads from one to the other.

`contracts/` can contain contracts for the APIs or interfaces needed between the components of the system.

`quickstart.md` can include scenarios that let you check manually that the resulting solution satisfies the main requirements.

Not every project necessarily needs all of these artifacts. The content will depend on the nature of the system and on the decisions made during planning.

## Checking against the Constitution

An important part of planning is checking that the technical decisions respect the Constitution.

For example, if the Constitution establishes **simplicity first**, a distributed architecture with many services should be clearly justified by the requirements. If it establishes that the catalog is the source of truth, the Plan must preserve that principle in the proposed architecture.

In ProductChat there are two principles that are especially easy to violate without noticing. The principle **inconsistency is shown, not hidden** is broken by any design that silently deduplicates the catalog when loading it. The principle **the chatbot reads, it does not correct** is broken by any plan that proposes generating a clean version of the spreadsheet and working on it. Both are reasonable technical decisions in the abstract, and both go against the Constitution of this project.

The Plan cannot be used to ignore or silently modify previously established principles.

## Reviewing and iterating on the Plan

The Plan generated by the AI must be reviewed before turning it into tasks. You do not need to understand every technical detail, but you do need to check that the decisions are consistent with the needs of the project.

You can use natural language to ask for explanations:

```text
Explain the decision in plan.md about when product_catalog.xlsx is read
as if I were the business owner and not a developer: what does the
salesperson notice if the catalog manager edits the spreadsheet at noon?
```

You can also ask for alternatives:

```text
Give me two alternatives for the decision about where normalization of the
catalog data is applied. For each one, tell me which requirement in spec.md
it favors, which one it makes harder, and which one you would choose if the
dominant criterion were simplicity.
```

Or ask for specific changes:

```text
Simplify this architecture while keeping all the requirements and acceptance
criteria in spec.md. Tell me explicitly which components you remove
and which requirement each one guaranteed before the change.
```

If the problem you detect belongs to the **how**, you change the Plan. If during the review you discover that the problem is really in the **what**, you must go back to the Specification and correct it before continuing.

## Outcome of the stage

At the end of this stage we should have a technical design precise enough for the agent to turn it into concrete units of work.

The separation between the stages looks like this:

`Constitution = what rules we must respect`

`Specification = what the system must do`

`Plan = how we are going to build it`

`Tasks = what concrete actions we must carry out`

Once the Plan has been reviewed and accepted, the next step is to turn it into **small, ordered and verifiable tasks** with `/speckit-tasks`.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
