# The Constitution in SDD

## What is the Constitution?

The **Constitution** is the first stage of the **Spec-Driven Development (SDD)** cycle. It defines the fundamental principles and rules that must be respected throughout the development of the project. Its goal is to establish a small set of **global, non-negotiable** rules that later guide the specification, the plan, the tasks and the implementation.

The Constitution mainly answers the question: **what rules must our project always respect?**

It does not describe specific features, and it does not determine how the system will be implemented technically. For example, it should not state which programming language, framework, AI model or database to use. Those decisions belong mainly to the **Plan**.

## What should it contain?

The principles of a Constitution should be **high-level, clear and actionable**. They usually represent decisions about the business, the scope and the quality of the product. For example: keeping the solution simple, not implementing features that are not specified, setting the language of the product, requiring verifiable results and defining general principles for how data is handled.

The Constitution should stay **short**, because its principles will be added again and again to the context used by the AI agents. It should contain only rules that truly need to govern the whole project.

## Constitution, Specification and Plan

It is important to keep these stages properly separated. The **Constitution** establishes permanent rules, for example: *"The chatbot must never invent information about the products"*. The **Specification** defines what the product must do, for example: *"The user must be able to check the price and availability of a product"*. The **Plan** determines how to implement that functionality.

In short:

`Constitution = permanent rules`

`Specification = what the product must do`

`Plan = how it will be implemented`

## Example: the ProductChat Constitution

Let's go back to the course case. **ProductChat** is an internal chatbot for Distribuidora Andes that lets users query the company's catalog of products and services in natural language. That catalog lives in a hand-maintained Excel spreadsheet, with duplicate SKUs, categories written in several ways, attributes without a consistent unit and free-text columns. The spreadsheet will not be cleaned before the chatbot is built.

That last point forces a constitutional decision that would not appear in a project with clean data: what the product must do when the source contradicts itself. It is neither a technical decision nor a feature. It is a permanent rule about how the system behaves when faced with data that cannot be fully trusted.

The Constitution should not yet specify how the Excel file will be read, which language model will be used or how the backend will be built. Those decisions belong to the Plan.

### Principles

**1. Simplicity first.** Given two solutions that satisfy the same requirements, the simpler one will be used. This is a first version of the product, and no complexity will be added to address future needs that have not yet been specified.

**2. The catalog is the source of truth, with all its defects.** Every answer about products or services must be traceable to a specific row of the file. The chatbot does not invent products, prices, attributes or availability.

**3. Inconsistency is shown, not hidden.** When the catalog contradicts itself, the chatbot reports the conflict instead of silently choosing a value. A piece of data delivered with false confidence is worse than a missing one.

**4. The chatbot reads, it does not correct.** The source file is never modified. Normalization of categories, units and formats happens when the data is interpreted, not when it is written.

**5. Zero phantom scope.** No functionality that is not defined in the specification will be implemented. If a potentially useful new feature comes up during development, it must be proposed and added to the specification before it is implemented.

**6. Verifiable answers.** The information provided must be checkable by opening the spreadsheet. Success criteria must be checkable by using the application, without having to review the code.

**7. Explicit uncertainty.** When the available data does not allow a query to be answered, the chatbot must say so clearly instead of filling in the answer with assumptions.

**8. Data protection.** Only the information needed to provide the service will be requested. Absolute paths, credentials and secrets must never be written directly into the code or exposed to the user.

## Workflow and governance

Every new feature must first appear in the specification. If a requirement is ambiguous and its interpretation could significantly change the behavior of the product, a clarification must be requested before implementing it. The specification, the plan and the tasks must be consistent with the principles established in the Constitution.

The Constitution prevails over individual implementation decisions that conflict with its principles. Any change to it must be documented and versioned, and you must then check that the existing specifications, plans and tasks are still consistent with it.

## Example with GitHub Spec Kit

Using **GitHub Spec Kit**, we request the creation of the Constitution with this prompt. Notice its structure: role, context, input, expected output and acceptance criterion. It is the contract that every prompt in the course follows, described in the previous module.

```text
/speckit-constitution

ROLE
Act as the person responsible for defining the permanent rules of a software
project. You are not the architect or the programmer: you do not choose
technologies.

CONTEXT
ProductChat is an internal chatbot for Distribuidora Andes that lets users
query the company's catalog of products and services in natural language. The
catalog lives in product_catalog.xlsx, a hand-maintained spreadsheet with
duplicate SKUs, categories written in several ways, attributes without a
consistent unit, prices stored as text and free-text columns. The spreadsheet
will not be cleaned before the chatbot is built.

INPUT
The eight principles listed below. There is no previous artifact: this is the
first stage of the cycle.

EXPECTED OUTPUT
The project's Constitution file, usually
.specify/memory/constitution.md, short, actionable and in plain language.
No decisions about languages, frameworks, AI models, architecture or specific
technologies.

PRINCIPLES
1. Simplicity first: given two solutions, always the simpler one. This is a
   version 1; no anticipated complexity.
2. The catalog is the source of truth, with all its defects: answers must be
   traceable to a specific row of the file. Do not invent products, prices,
   attributes or availability.
3. Inconsistency is shown, not hidden: when the catalog contradicts itself,
   the chatbot reports it instead of silently choosing a value.
4. The chatbot reads, it does not correct: it never modifies the source file.
   Normalization happens when the data is interpreted, not when it is written.
5. Zero phantom scope: do not implement any functionality that is not written
   in the Specification. If a new idea comes up, it is proposed, not built.
6. Verifiable by a non-technical person: every success criterion must be
   checkable by using the application and opening the spreadsheet, without
   reading code.
7. Explicit uncertainty: when the data does not allow an answer, say so
   clearly instead of guessing.
8. Respect for data: ask only for what is strictly necessary. Do not put
   absolute paths, credentials or secrets in the code.

ACCEPTANCE CRITERION
The resulting Constitution fits on one page, contains the eight principles
without adding others, and none of them mentions a specific technology.
```

> This prompt is in `prompts/02.1-constitution.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Spec Kit will use these instructions to generate the project's Constitution file, usually:

```text
.specify/memory/constitution.md
```

This document establishes the rules that will govern the following stages of development:

`Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation`

The Constitution is initially defined only once, and it should only be changed when the fundamental principles that govern the project change.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
