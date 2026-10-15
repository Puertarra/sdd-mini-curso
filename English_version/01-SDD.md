# Spec-Driven Development (SDD)

## What is Spec-Driven Development?

**Spec-Driven Development (SDD)** is an approach to software development in which the **specification becomes the main source of truth for the project**.

Instead of asking an AI directly to generate an application from a prompt, you first define precisely what must be built, which constraints it must respect and which conditions it must meet. From that information, AI agents can generate the technical plan, break it into tasks and implement the software.

The core idea can be summarized as:

**Specify → Plan → Break into tasks → Implement**

The developer or project lead focuses mainly on defining and reviewing the **what**, while the AI agent can take care of much of the **how**.

---

## Where does SDD come from?

The idea of building software from specifications is not new. For decades, software engineering has used requirements, specifications, design and planning before implementing a system.

**Modern Spec-Driven Development** emerges with the arrival of AI agents able to work directly on software repositories. The specification stops being only documentation for humans and becomes a structured input that guides the agent during planning and implementation.

Tools such as **GitHub Spec Kit**, **OpenSpec** and **Kiro** formalize different variants of this approach and let you use agents such as **Claude Code** or **Codex** to carry out the development.

The key difference from traditional development is the speed of the cycle. AI can turn a specification into a plan, tasks and code in minutes, which makes it possible to iterate quickly on the specification.

---

## SDD versus Vibe Coding

**Vibe Coding** consists, put simply, of describing through prompts what we want to build and letting the AI generate the code directly.

For example:

> "Build a chatbot to query my company's product catalog."

The AI has to fill in on its own many details that were never specified. It can quickly produce a working application, but it can also make wrong or inconsistent decisions.

This approach is especially useful for **prototypes, proofs of concept and quick experimentation**, where the main goal is to get something working in little time.

The problem appears when the system grows in complexity or has to become stable, maintainable software. Implicit requirements, decisions made by the AI and changes made through successive prompts can lead to inconsistencies and make traceability hard.

SDD tries to solve this problem by changing the flow:

**Vibe Coding:**

`Idea → Prompt → Code`

**Spec-Driven Development:**

`Idea → Constitution → Specification → Clarify → Plan → Tasks → Analyze → Code`

The main difference is that in SDD **the AI should not have to guess important requirements**. They are set before implementation begins.

---

---

# The course case

The whole course works on **a single case**, which we will develop stage by
stage until we reach the code.

**ProductChat** is an internal chatbot for **Distribuidora Andes**, a company
that sells computer equipment. Its catalog of products and services lives in
an Excel spreadsheet maintained by hand by the sales department: `product_catalog.xlsx`.

That spreadsheet is the problem. It has duplicate SKUs with prices that do not match,
the same category written thirteen different ways, attributes without a consistent
unit (`32 GB`, `32768 MB`, `32`), prices that are sometimes numbers and sometimes
text, and a notes column where the data that matters sometimes lives.
Nobody is going to clean it up before the chatbot is built.

That mess is not a background detail we can forget later: it is what
makes the case interesting. A specification that assumes a clean catalog
will produce a chatbot that confidently answers with wrong data.

The starting point is two files in this folder:

```text
product_catalog.xlsx          ← the data, as it is
napkin_spec_productchat.md    ← the spec written by hand, before using Spec Kit
```

Read both before continuing.

# The four stages of SDD

Before going into each one, it helps to pin down the vocabulary, because the cycle has three different kinds of step and they are easy to confuse.

There are **four stages**, and each one produces an artifact: the Constitution, the Specification, the Plan and the Tasks. These are the ones we will study in detail.

There are also **two quality gates**, Clarify and Analyze, which do not produce new artifacts: they review the existing ones and propose corrections. The first reviews the Specification internally; the second reviews the consistency across all the artifacts. Formally they are optional, but in this course we always use them, because that is where problems are caught while they are still cheap to fix.

And there is one **execution**, the Implementation, which turns everything before it into code.

The complete flow looks like this:

`Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation`

## 1. Constitution

The **Constitution** sets out the principles, constraints and non-negotiable rules that will govern the whole project.

They are global rules that must be respected regardless of the functionality being implemented.

For example:

- The product must be available in English.
- Development must be kept as simple as possible.
- No functionality that has not been specified may be implemented.
- Certain security or architecture rules must be respected.

The Constitution is usually defined once and governs the later stages.

**Question it answers:**  
`What rules must we always respect?`

---

## 2. Specification

The **Specification (Spec)** defines **what must be built and why**.

It describes the system requirements from a functional point of view, without going unnecessarily into technical implementation decisions.

It can include:

- Product goal.
- Users.
- Features.
- Requirements.
- User stories.
- Functional constraints.
- Acceptance criteria.
- Conditions that determine when a feature is done.

A good specification must make it possible to check objectively whether the result meets what was asked for.

The specification should focus mainly on the **what**, not on the **how**.

**Question it answers:**  
`What do we want to build and what must it meet?`

---

## 3. Plan

The **Plan** turns the specification into a technical solution.

This is where we decide **how the system will be built**.

It can set out:

- Architecture.
- Technologies.
- Database.
- Backend.
- Frontend.
- APIs.
- Components.
- Project structure.
- Dependencies.
- Technical decisions.

In an agentic development flow, much of this plan can be generated by the AI from the specification and then reviewed by a person.

This makes it possible to clearly separate two responsibilities:

`Specification = what`

`Plan = how`

**Question it answers:**  
`How are we going to build it?`

---

## 4. Tasks

The **Tasks** stage turns the technical plan into small units of work that can be executed and verified individually.

For example:

- Read the catalog from the spreadsheet and locate the header row.
- Normalize equivalent categories.
- Normalize memory attributes to a single unit.
- Implement product search by features.
- Detect duplicate SKUs with conflicting values.
- Create the corresponding tests.

Tasks must be **concrete, small and verifiable**, so that it is possible to tell clearly when each one is done.

The AI agent can execute these tasks one after another until the implementation is complete.

**Question it answers:**  
`What exactly must we do to implement the plan?`

---

# Complete flow

The four stages can be summarized as follows:

| Stage | Main question | Result |
|---|---|---|
| **Constitution** | What rules must we always respect? | Principles and constraints |
| **Specification** | What do we want to build? | Requirements and acceptance criteria |
| **Plan** | How are we going to build it? | Design and technical decisions |
| **Tasks** | What exactly must we do? | Executable work list |

To those four stages we add the two quality gates and the execution:

| Step | Type | Function |
|---|---|---|
| **Clarify** | Quality gate | Detects ambiguities within the Specification |
| **Analyze** | Quality gate | Detects inconsistencies across Constitution, Specification, Plan and Tasks |
| **Implementation** | Execution | Turns the Tasks into verified code |

The resulting flow is:

`Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation`

An important feature of SDD applied to AI agents is that this process does not have to be strictly linear. If during implementation you find that the result does not match what was expected, you can go back to the specification, correct it and regenerate the later stages.

So the **specification works as the central reference for development**, while the plan, the tasks and the code evolve from it.

---

# Course vocabulary

One term per concept, with the same meaning in every module.

| Term | Meaning |
|---|---|
| **Stage** | A step in the cycle that produces an artifact. There are four: Constitution, Specification, Plan and Tasks. |
| **Quality gate** | A step that reviews existing artifacts without producing a new one. There are two: Clarify and Analyze. |
| **Constitution** | The project's permanent rules. File: `constitution.md`. Command: `/speckit-constitution`. |
| **Specification** | What the product must do. File: `spec.md`. Command: `/speckit-specify`. |
| **Clarify** | Gate that detects ambiguities within the Specification. Command: `/speckit-clarify`. |
| **Plan** | How it will be built. File: `plan.md`. Command: `/speckit-plan`. |
| **Tasks** | Verifiable units of work. File: `tasks.md`. Command: `/speckit-tasks`. |
| **Analyze** | Gate that detects inconsistencies across all the artifacts. Command: `/speckit-analyze`. |
| **Implementation** | Execution of the Tasks until verified code is obtained. Command: `/speckit-implement`. |
| **Napkin spec** | The specification written by hand by a person, before using Spec Kit. It is the input to `/speckit-specify`, not its output. |
| **User story (US)** | A user's need in the form *As a <user>, I want <need>, so that <goal>*, followed by a scenario that makes it verifiable. Identified as `US1`, `US2`… |
| **Refined story** | A verifiable subdivision of a user story. Identified as `US1.1`, `US1.2`… |
| **The catalog** | The case's data source: `product_catalog.xlsx`, sheet `Catalog`. The only name used to refer to it. |
| **Method layer** | Spec Kit: organizes the process. |
| **Agent layer** | Claude Code, Codex or another: carries out the process. |

Two clarifications worth keeping in mind. The **Specification** is the formal artifact that Spec Kit produces; the **napkin spec** is what we hand it so that it can produce it. And the **Implementation** is not a fifth stage: the four stages are already over when it begins.

---

# How we write prompts in this course

An SDD prompt is not a conversational request: it is the input to a stage and the source of an artifact that other stages will consume. If it is not reproducible, traceability breaks at the first link.

That is why every prompt in this course declares five things:

| Block | What it answers |
|---|---|
| **Role** | Who the agent is in this step, and which decisions are **not** its call |
| **Context** | What the project is and which particularity of the case shapes the answer |
| **Input** | Which artifacts it consumes, named by their path |
| **Expected output** | What it produces, with what structure and in which file |
| **Acceptance criterion** | How we know, without reading code, that the output is fit for use |

The prompts are in the `prompts/` folder, ready to copy. Their number indicates the module where they are used and their order within it: `03.2-clarify.txt` is the second prompt of module 03. The modules reproduce them, but the reference version is always the file.

## Chaining

Each prompt consumes what the previous one produced. That is the property that makes SDD a method and not a series of disconnected requests:

| # | Prompt | Input | Output |
|---|---|---|---|
| 1 | `prompts/02.1-constitution.txt` | human decisions | `.specify/memory/constitution.md` |
| 2 | `prompts/03.1-specify.txt` | `napkin_spec_productchat.md`, `constitution.md`, `product_catalog.xlsx` | `spec.md` |
| 3 | `prompts/03.2-clarify.txt` | `spec.md`, `constitution.md` | updated `spec.md` |
| 4 | `prompts/04.1-plan.txt` | `spec.md`, `constitution.md`, `product_catalog.xlsx` | `plan.md`, `data-model.md`, `research.md` |
| 5 | `prompts/05.1-tasks.txt` | `plan.md`, `data-model.md`, `spec.md` | `tasks.md` |
| 6 | `prompts/06.1-analyze.txt` | all of the above | findings report |
| 7 | `prompts/07.1-implement.txt` | `tasks.md` and the other artifacts | verified code |

If, when you reach a step, the file its prompt declares as input does not exist, something was skipped. That check, done before pasting the prompt, saves you most problems.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
