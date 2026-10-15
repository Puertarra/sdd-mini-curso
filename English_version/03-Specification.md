# The Specification in SDD

## What is the Specification?

The **Specification (Spec)** is the second stage of the **Spec-Driven Development (SDD)** cycle and probably the most important one. It defines precisely **what the product must do, for whom, and under what conditions it is considered correct**.

While the Constitution establishes the permanent rules of the project, the Specification describes the expected behavior of the product. It must focus on the **what** and avoid defining the **how**, since technical decisions belong later to the Plan.

The Specification mainly answers the question: **what do we want to build and what must it satisfy?**

## What makes a specification good?

A good specification must contain at least four fundamental elements.

**1. Clear intent.** It must explain what problem is being solved and who the user is. The goal must be understandable without technical knowledge.

**2. Business context.** It must explain why the problem matters and how the product will actually be used. This ties the requirements to real situations, so they are not just an abstract list of features.

**3. Constraints and business rules.** It must state the conditions that the product must necessarily respect. These rules must be concrete and leave no room for interpretation.

**4. Verifiable acceptance criteria.** It must state conditions that make it possible to determine objectively whether a feature is correctly implemented. Ideally, each criterion should be answerable with a **yes or a no** by using the application.

## What makes a specification bad?

A bad specification usually has four problems. The first is **ambiguity**, through expressions such as *"the chatbot must answer well"* or *"the interface must look professional"*, which force the AI to interpret what exactly "well" or "professional" means.

The second is **technical overspecification**, prematurely including languages, frameworks, models, databases or architecture decisions. The third, closely related, is **mixing the what with the how**.

The fourth is quieter, and it is the one that will occupy us most in this course: **assuming the data is clean**. It is common to describe the mess in the source in the introduction and then write requirements and acceptance criteria that only work if that mess did not exist. A criterion such as *"the price shown matches the one recorded in the source"* looks verifiable, but it stops being verifiable as soon as the source records two different prices for the same product. If the source is imperfect, the specification has to say what to do with each imperfection.

For example:

> "The user must be able to query products using natural language."

is a specification of the **what**.

On the other hand:

> "Implement the queries using Python, FastAPI and a specific model."

describes the **how** and belongs to the Plan.

## Checklist

Before approving a specification, it is worth checking:

1. Can the goal be understood without technical knowledge?
2. Is it clear who the user is?
3. Are the business rules concrete?
4. Are there acceptance criteria that can be verified with a yes or a no?
5. Is the what separated from the how?
6. Is there a section that explicitly states what is out of scope?
7. Have edge cases been considered, such as empty, nonexistent or incorrect data?
8. Is there any important decision the AI would have to guess?
9. Are there ambiguous expressions that could be interpreted in different ways?
10. Could another person read the specification and build essentially the same product?
11. Does it say what must happen when the data source contradicts itself?
12. Are the acceptance criteria still verifiable with the real data, as it stands today?

If any relevant decision remains open, it should be resolved in the specification instead of letting the AI agent decide it arbitrarily.

## Structure of a specification

A simple, reusable structure can contain:

```text
1. Goal and business context
2. Users
3. User stories
4. Functional requirements
5. Business rules
6. Acceptance criteria
7. Edge cases
8. Out of scope
```

> **If you come from Scrum.** Scrum does not define how requirements are written; user stories come from XP. A good source for writing them is a story map (*User Story Mapping*, Jeff Patton) made before specifying: it lays out what the user does from start to finish and makes the missing stories visible. In an agile backlog, what are functional requirements here usually end up as acceptance criteria of each item, or as smaller items that appear when a story is split during *refinement*. Business rules apply to several stories at once; in the example that follows, the memory unit conversion affects three. In a backlog those rules get copied into each item and, over time, they contradict each other. The agile practice closest to this structure is *Example Mapping*, proposed by Matt Wynne, which organizes each story into rules, examples and open questions. The underlying difference lies in the conversation. Ron Jeffries described the user story with three Cs: the *Card*, the *Conversation* and the *Confirmation*. The card is brief because the details are agreed on with the team. In SDD, whoever implements is an agent that was not part of that conversation, and the requirements and rules are that conversation put in writing.
>
> Jeffries, R. (2001). [*Essential XP: Card, Conversation, Confirmation*](https://ronjeffries.com/xprog/articles/expcardconversationconfirmation/). · Patton, J. (2014). *User Story Mapping: Discover the Whole Story, Build the Right Product*. O'Reilly. · Wynne, M. (2015). [*Introducing Example Mapping*](https://cucumber.io/blog/bdd/example-mapping-introduction/).

# Example: the ProductChat specification

Let's go back to **ProductChat**, the internal chatbot of Distribuidora Andes. Its catalog of products and services lives in `product_catalog.xlsx`, a hand-maintained spreadsheet with duplicate SKUs, inconsistent categories, attributes without a consistent unit and free-text columns.

What follows is the formal specification derived from `napkin_spec_productchat.md`. It is worth reading both and comparing them: the napkin spec is what a person writes; this is what that person wants to get back.

## 1. Goal and business context

ProductChat lets users query the Distribuidora Andes catalog in natural language, without opening the spreadsheet and without knowing its structure.

The problem is not only that the spreadsheet is awkward to query. It is that **querying it by hand gives wrong answers**. A salesperson looking for notebooks with 32 GB of RAM by filtering on category misses the machines entered as `Laptops`; searching for the text `32 GB` misses the ones entered as `32768 MB` and as `32`; searching for just `32` brings in a Dell that can only be expanded to 32 GB on request. The catalog will not be cleaned before the chatbot is built, so **correctly interpreting messy data is the central requirement of the product**, not an implementation detail.

## 2. Users

The main user is the **salesperson**: they serve customers, are not technical, and need fast, reliable answers. They prefer an "I'm not sure" to made-up data.

The **catalog manager** does not use the chatbot, but wants inconsistencies to be made visible instead of being hidden, because that tells them what to fix first in the spreadsheet.

The **end customer** does not use the chatbot either: they get the answer from the salesperson.

## 3. User stories

Each story is written in the standard form *As a <user>, I want <need>, so that <goal>*. That form says what someone needs and why, but it does not let you check whether it was satisfied. That is why each story is followed by a **scenario**: a concrete query about the catalog with its expected result. It is the same structure as the Spec Kit specification template, where each *User Story* comes with its *Acceptance Scenarios*.

### US1 — Search for products

As a salesperson, I want to find products by name, category or features written in natural language, so that I do not depend on how the data is written in the spreadsheet.

*Scenario:* the salesperson asks *"which notebooks have 32 GB of RAM?"*. The chatbot returns the four machines that actually qualify, including those that state the memory in another unit, and excluding those that only support it as an upgrade.

### US2 — Ask about the attributes of a product

As a salesperson, I want to ask for the price, the stock or a specific feature of a product, so that I can answer the customer on the spot.

*Scenario:* the salesperson asks *"how much RAM does the Acer Swift Go have?"*. The chatbot answers 32 GB, citing the source SKU, even though the spreadsheet records it as `32768 MB`.

### US3 — Handle missing or insufficient information

As a salesperson, I want the chatbot to tell me clearly when it does not know, so that I do not repeat data that does not exist to the customer.

*Scenario:* the salesperson asks about a product that is not in the catalog, or about the price of the LG UltraWide Monitor, whose cell says `ask`. The chatbot reports that the information is missing instead of filling it in.

### US4 — Deal with an inconsistent catalog

As a salesperson, I want the chatbot to understand that `32768 MB` and `32 GB` are the same and that `Laptops` and `Notebooks` are the same family, and to warn me when the catalog contradicts itself, so that I do not have to choose blindly between two prices.

*Scenario:* the salesperson asks *"what is the price of the ThinkBook 14?"*. SKU `NB-1001` appears twice with different prices. The chatbot reports that two prices are recorded and that the catalog is inconsistent on that point, **without choosing one**.

## 4. Functional requirements

**FR1.** The system must read the catalog from `product_catalog.xlsx`, locating the header row even if it is not the first row of the file.

**FR2.** The system must discard empty rows and trim extra whitespace from text values.

**FR3.** The system must normalize the category, so that variants in capitalization, spelling and declared synonyms are treated as the same family.

**FR4.** The system must normalize numeric attributes with units, so that `32 GB`, `32768 MB`, `RAM 32 GB` and `32` are interpreted as the same value.

**FR5.** The system must normalize the price to a numeric value, and mark empty or textual cases as **price not available**.

**FR6.** The system must normalize stock to three states: available, out of stock and not reported.

**FR7.** The system must detect duplicate SKUs and, when the duplicate records do not match on the queried attribute, report the conflict.

**FR8.** The user must be able to query products by name, category or features using natural language.

**FR9.** When several products satisfy a query, the system must be able to present multiple results.

**FR10.** The system must distinguish a declared attribute from a conditional attribute mentioned in free text, and must not treat the second as if it were the first.

**FR11.** When there is not enough information to answer, the system must say so explicitly.

**FR12.** Every piece of data delivered must be accompanied by the SKU it comes from.

## 5. Business rules

**BR1.** The `Catalog` sheet is the source of truth. The `old_catalog` sheet is historical and is not used in this version.

**BR2.** Equivalent families: `Notebooks` ≡ `Laptops` ≡ `Laptop`, with any combination of capitalization and spelling variants.

**BR3.** Memory unit conversion: 1024 MB = 1 GB. A number without a unit in a memory field is interpreted as GB.

**BR4.** An attribute mentioned as a future condition ("expandable to", "on request", "made to order") does not count as an attribute of the product.

**BR5.** When a duplicate SKU has conflicting values, the chatbot does not choose: it reports both values and points out the inconsistency.

**BR6.** The chatbot does not invent products, prices or attributes. Every piece of data delivered must be traceable to a row of the file.

## 6. Acceptance criteria

**AC1.** Given "which notebooks have 32 GB of RAM?", the chatbot returns `NB-1005`, `NB-1007`, `NB-1009` and `NB-1012`, and does not include `NB-1003`.

**AC2.** Given "which notebooks are there?", the result includes the machines entered as `Laptops` and `Laptop`.

**AC3.** Given "what is the price of the ThinkBook 14?", the chatbot reports that there are two conflicting prices for `NB-1001` and does not give a single one as if it were the correct one.

**AC4.** Given a query for the price of `MON-3002`, the chatbot answers that the price is not available.

**AC5.** Given a query for the stock of `NB-1007`, whose value is `Yes`, the chatbot answers "available" and not a number.

**AC6.** Given a query for a nonexistent product, the chatbot states that it found no information.

**AC7.** Every piece of data delivered is accompanied by its source SKU.

**AC8.** A non-technical person can run the queries above and verify each answer by opening the spreadsheet.

## 7. Edge cases

**EC1.** Completely empty row in the middle of the data: it is ignored.

**EC2.** Product without features (`MON-3003`): it is found by name and category; when a feature is requested, the answer is that it is not recorded.

**EC3.** Product without a price (`NB-1009`): it appears in the results, with the price marked as not available.

**EC4.** Very similar names (`NB-1001` and `NB-1015`): both are returned and are distinguished by SKU.

**EC5.** Same product under two SKUs (`NB-1002` and `NB-2002`): both are returned; the chatbot does not merge them on its own.

**EC6.** Services (`SRV-*`): they have no stock; asking for their stock must return "not applicable", not "out of stock".

**EC7.** The file cannot be read: the chatbot reports it and does not answer with old data.

## 8. Out of scope

For this first version, the following are out of scope:

- Modifying, creating or deleting products from the chatbot.
- **Correcting the Excel file.** The chatbot reads; it does not write.
- Buying, issuing formal quotes or managing orders.
- Recommending products based on criteria outside the catalog.
- Answering questions that are not about the catalog.
- Using the `old_catalog` sheet.
- Connecting to an SQL database. In this version the source is the Excel file.
- User accounts, permissions and multi-user support.
- Adding features that are not explicitly described in this specification.

## Validating the Specification with Clarify

Once the specification has been created, it is a good idea to review it before moving on to the Plan. In **GitHub Spec Kit**, this review is done with:

```text
/speckit-clarify

ROLE
Act as a requirements reviewer. Your task is to find what still needs to be
decided, not to decide it yourself.

CONTEXT
You are reviewing the specification of ProductChat, a chatbot over an Excel
catalog with inconsistent data.

INPUT
spec.md and .specify/memory/constitution.md.

EXPECTED OUTPUT
A list of concrete questions, each with its possible alternatives and their
consequences, for me to choose from. After I answer, incorporate the
decisions into spec.md.

FOCUS
Pay special attention to the points where the specification assumes clean
data that the catalog does not guarantee: what happens with two conflicting
values for the same SKU, which unit is assumed when an attribute does not
state one, whether a question about availability refers to the catalog or to
stock, and what to do with attributes mentioned only as a future condition in
free text.

ACCEPTANCE CRITERION
When you finish, spec.md contains no pending decision that could change the
observable behavior of the product. Every decision incorporated was made by
me, not by you.
```

> This prompt is in `prompts/03.2-clarify.txt`, ready to copy. If you change it, change it there: that file is the reference version.


**Clarify** analyzes the specification looking for **ambiguities, unresolved decisions, incomplete requirements or situations in which the AI would have to assume information that has not been defined**. When it detects these points, it asks questions so that the user is the one who makes the necessary decisions.

The goal is to reduce assumptions to a minimum before planning begins. The answers obtained during this process are incorporated into the specification, leaving a more precise version that is ready for generating the Plan.

In ProductChat, section 9 of the napkin spec already anticipates much of what Clarify should ask. Two concrete examples. First, BR5 says that, given two conflicting prices, the chatbot reports both, but that is a business decision nobody has confirmed: the company might prefer a tie-breaking rule. Second, when the salesperson asks which notebooks *have* 32 GB, it is not clear whether they are asking about the catalog or about what they can sell today: of the four machines that qualify, only two have a numeric stock greater than zero. Neither of these decisions should be made by the agent on its own.

The flow then becomes:

`Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation`

# The specification as a human responsibility

AI can help structure, review and formalize a specification, but the fundamental decisions should come from whoever understands the problem and the business.

In SDD, the person mainly defines the **intent, the business rules, the scope and the acceptance criteria**. The agent can then turn that information into more formal documents, generate the Plan and break it down into executable tasks. The specification is therefore the stage where there should be the most human participation; we call this **Human in the Loop**, or simply **HITL**:

`Person: defines and validates the what`

`AI: helps formalize the what and develops the how`

# Using it with GitHub Spec Kit

Once the elements above have been defined, they can be used as input for **GitHub Spec Kit** through `/speckit-specify`.

The goal is not simply to ask the AI:

```text
Create a chatbot to query products.
```

but to give it the intent, users, user stories, requirements, business rules, acceptance criteria, edge cases and out-of-scope items that we have just defined. All of that is already written in `napkin_spec_productchat.md`, so the prompt does not repeat its content: it names it as the input file.

```text
/speckit-specify

ROLE
Act as a requirements analyst. Your task is to formalize a handwritten
specification, not to design the solution or choose technologies.

CONTEXT
ProductChat is an internal chatbot for Distribuidora Andes that lets users
query the company's catalog of products and services in natural language. The
catalog lives in product_catalog.xlsx and is messy on purpose: duplicate SKUs
with different prices, categories written in several ways, attributes without
a consistent unit, prices stored as text and data hidden in free-text columns.
The spreadsheet will not be cleaned.

INPUT
The file napkin_spec_productchat.md, in the project root. Read it in full
before you start. You may also consult product_catalog.xlsx to check the
examples the napkin spec mentions, and .specify/memory/constitution.md so as
not to contradict its principles.

EXPECTED OUTPUT
The project's formal specification, spec.md, with these sections: goal and
business context, users, user stories identified as US1, US2…, functional
requirements numbered FR1, FR2…, business rules numbered BR1, BR2…,
acceptance criteria numbered AC1, AC2…, edge cases and out of scope.

CONSTRAINTS
Do not decide languages, frameworks, AI models, architecture or libraries:
that belongs to the Plan. Do not resolve on your own any of the open questions
in section 9 of the napkin spec. Before drafting, ask me any questions you
need to resolve any ambiguity.

ACCEPTANCE CRITERION
Every acceptance criterion in spec.md can be checked by opening
product_catalog.xlsx and using the application, without reading code. No
requirement assumes the catalog is clean. The open questions remain open.
```

> This prompt is in `prompts/03.1-specify.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Two details deserve attention. The prompt explicitly forbids resolving the open questions in the napkin spec, because that is the job of Clarify and of a person. And its acceptance criterion requires that no requirement assume a clean catalog, which is how you check that the unreliability of the data survived the formalization.

From this information, Spec Kit can generate the formal specification that will serve as the source of truth for the following stages:

`Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation`

If an ambiguity about **what the product must do** comes up during later stages, the solution should not be to improvise it in the code. You must go back to the specification, resolve the decision and continue the cycle from a more precise definition.

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
