# Reverse Specs in SDD

The whole course started from the requirements: first the Constitution and the Specification, then the code. An engineer's first job is almost never like that. The usual situation is arriving at a system that already works, that someone uses every day and that nobody documented.

**Reverse specs** follow the opposite process: you rebuild the specification from the code, from how it runs and from the people who use it. The goal is not to criticize the system or to improve it, but to understand and protect what it does today before changing it.

It is worth clarifying something from the start. To adopt SDD in an existing project, you do not need to specify all of it: the Spec Kit guide for existing projects recommends capturing the rules that matter and applying the cycle to the next bounded change. Here we do it in full because that is the exercise, and because it is the best way to close the course: it forces you to use everything that came before in the opposite direction.

## Reversing the direction

```text
Existing code → Observation and evidence → Rebuilt rules
                                                  ↓
                                   Confirmation with the business
                                                  ↓
                             Characterization tests and Spec
```

Throughout the process there are three questions you should not mix: **what the system does**, **why it is used that way** and **what should change**. Execution answers the first. People answer the second. The third is left for later, as a separate proposal.

An observed behavior is not automatically a requirement. And a desired requirement does not necessarily describe what the system does.

## The system to study

The system is a classmate's ProductChat. Each student receives someone else's version, finished at the end of module 11, without `specs/` and without the napkin spec. That way nobody has the requirements of the system they are studying at hand.

The copy is handed over **without history**: if it arrived with its Git history, the deleted Specs would still be visible in earlier commits. Whoever prepares it, the instructor or the classmate, exports it like this from the original repository:

```bash
git archive --format=tar --prefix=productchat-received/ HEAD | tar -x -C ..
cd ../productchat-received
rm -rf specs napkin_spec_productchat.md
```

And whoever receives it records that version as the starting point, noting the source commit:

```bash
git init -b main
git add .
git commit -m "Received snapshot of ProductChat (source commit abc1234)"
```

There is a trap in this exercise, and it is worth naming. You know the Specs your classmate had to meet: they are the same ones you implemented. The temptation will be to write what the system *should* do. But a reverse spec describes what the code **does**, and it is precisely in the differences that the value of the exercise appears.

Outside the classroom, the system rarely comes from a classmate who took the same course, and the central question becomes what serves as the reference. There are two typical situations.

The first is your own project built with *vibe coding*: software built by asking an agent for changes, without writing a specification (Andrej Karpathy coined the term in 2025). There you have access to part of the history: your memory, the conversation with the agent and, if there were any, the commits. But none of those sources is reliable on its own. Memory mixes what you wanted with what ended up in the code. Prompts record what you asked for, not what the agent did. And the intent probably changed halfway through without being written down. The risk is writing the Spec of the system you imagined. The rule is the one from this module: the characterization tests say what the system does; your memories and your prompts are evidence of intent, and they are recorded separately, like an interview.

The second is someone else's project. There, intent only reaches you through people or through traces such as the history and the tickets. If there is no one to ask, the observed behavior is the only reference, and every rule stays as observed and not confirmed. The risk is the opposite one: taking as a requirement something that is actually an error, like `NB-1009` shown at $0, which you will find later in this module.

In neither case will you have, at the end, original Specs to compare your reconstruction against. That comparison is a luxury of the classroom exercise. In practice, the reverse spec is validated another way: it is reviewed by whoever uses or commissioned the system, and the characterization tests pass on the received version.

The discovery documents go in a `discovery/` folder at the root, not inside `specs/`. Spec Kit numbers its Specs based on what it finds in `specs/`, and it is best not to mix material that is not a Spec into that folder.

## Discovery: from the map to the flow

Exploration goes from the general to the specific: first which modules exist and what each one does, then the full path of one concrete query. That way you know where to look before reading code line by line.

```text
ROLE
Act as an engineer who receives a system with no documentation. You observe
and describe; you do not modify code or data.

CONTEXT
You received a copy of ProductChat built by someone else, without specs/ and
without the napkin spec. You know it is a chatbot that queries
product_catalog.xlsx, but you do not know which rules it implements. The
first commit in the repository is the received version.

INPUT
The code, the existing tests, CLAUDE.md if it exists, and
product_catalog.xlsx.

EXPECTED OUTPUT
Two files in discovery/: map.md, with the modules, their responsibility in
business language, the entry point and the verified commands to run and
test; and query-flow.md, with the path of the query "Peripherals with a
maximum price of 34.99" from the question to the answer, showing in which
function the data is read, normalized and filtered, and the answer produced.

CONSTRAINTS
Every claim cites its source: path and lines, or a run with its input and
its output. Distinguish what you ran from what you only read. Do not fix
anything that looks like an error: note it.

ACCEPTANCE CRITERION
Someone else can follow query-flow.md by opening the paths it cites and
reach the same answer by running the query.
```

> This prompt is in `prompts/12.1-explore.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Every claim in the discovery cites its source: a path with lines, or a run with its input and its output. Code comments help, but they may be out of date: they are not evidence.

## Extracting rules with evidence

A rule is written in terms of behavior, not code. "A product priced exactly at the maximum is returned" is a rule. "The `<=` operator is used" is a description of the implementation.

The most effective way to extract rules is to probe the system with the hard cases in the catalog. In ProductChat we already know which they are:

| Query | What to observe |
|---|---|
| Peripherals, maximum 34.99 | Does `PER-4001`, which costs exactly 34.99, appear? |
| Notebooks with 32 GB of RAM | Which ones does it return? Does `NB-1003`, which is only expandable, count? And `NB-1009`, which has no stock? |
| Price of `NB-1001` | One price or two? Does it warn about the conflict? |
| Price of `NB-1009` | Empty, 0 or "not available"? |
| Stock of `SRV-5001` | "Not applicable", "0" or "out of stock"? |
| RAM of `NB-1009` | "32 GB", "32" or something else? |
| Monitors, maximum 150 | `MON-3001` and `MON-3004` separately or merged? |

```text
ROLE
Act as an analyst who rebuilds business rules from behavior. You describe
what the system does, not what it should do.

CONTEXT
The ProductChat map and the flow of one query already exist. Now you need to
extract the rules the system applies to a catalog full of ambiguous cases:
empty prices or prices as text, duplicate SKUs, categories with several
spellings, different memory units and stock as text.

INPUT
discovery/map.md, discovery/query-flow.md, the code and
product_catalog.xlsx.

EXPECTED OUTPUT
discovery/rules.md with rules numbered BR-01, BR-02… Each one with a
statement in business language, an example SKU, the observed output, the
evidence and a status: observed (it was run) or inferred (only read in the
code).

WHAT TO PROBE
Run at least these queries and record the exact answer: Peripherals with a
maximum price of 34.99; notebooks with 32 GB of RAM; price of NB-1001;
price of NB-1009; stock of SRV-5001; RAM of NB-1009; Monitors with a maximum
price of 150.

CONSTRAINTS
Do not infer intent from the code. If a result looks like an error, record
it anyway as an observed rule and note it for the anomaly log.

ACCEPTANCE CRITERION
Every query on the list has its output recorded, and every rule cites at
least one SKU that reproduces it.
```

> This prompt is in `prompts/12.2-rules.txt`, ready to copy. If you change it, change it there: that file is the reference version.

Each rule is marked as **observed**, if it was run, or **inferred**, if it was only read in the code. The difference matters: an inferred rule can be wrong without anyone noticing.

## Anomalies: what is odd is not fixed, it is recorded

Discovery always finds odd behaviors. The natural reaction is to fix them. That is a mistake: first you need to know whether someone depends on them.

Suppose that in the received version the empty price of `NB-1009` shows up as \$0. It looks like a defect, and it probably is. But if the salespeople learned to ignore products at \$0, "fixing" it changes what they see every day without anyone having asked for it. The anomaly is recorded, the question is asked, and only then is a decision made.

| Classification | What is done in this phase |
|---|---|
| Intentional | It is specified as a rule |
| Accidental | It is described as observed behavior, not as a requirement |
| Defect | It is recorded with its evidence and its dependencies, and a future fix is proposed |
| Pending | The question is left open |

A frequent source of anomalies is **parallel paths**: the same rule implemented twice. In ProductChat this can easily happen with the RAM of `NB-1009`, whose value in the spreadsheet is just `32`. The feature search may interpret it as 32 GB, while the answer to "how much RAM does the MSI Modern 15 have?" shows "32" with no unit. Two paths, two interpretations of the same data.

```text
ROLE
Act as a behavior auditor. You classify what is odd; you do not fix it.

CONTEXT
The ProductChat rules are already recorded. Some look like errors, and some
rules are applied differently depending on the path: the same feature can
be interpreted one way when searching and another way when answering about
a product.

INPUT
discovery/rules.md, discovery/query-flow.md and the code.

EXPECTED OUTPUT
discovery/anomalies.md with entries A-01, A-02… Each one with the behavior,
the SKU that reproduces it, evidence, impact, a provisional classification
(intentional, accidental, defect or pending) and the question that would
resolve it. Include the cases where two paths treat the same data
differently, for example the RAM of NB-1009 in a search and in an attribute
query.

CONSTRAINTS
Do not close a classification without evidence; when in doubt, leave it
pending. Do not decide which path is correct.

ACCEPTANCE CRITERION
Every anomaly is reproduced with its SKU, and every classification other
than "pending" cites its evidence.
```

> This prompt is in `prompts/12.3-anomalies.txt`, ready to copy. If you change it, change it there: that file is the reference version.

## Asking the business

The code says what the system does, but not why. Intent, manual processes and dependencies are known only to whoever uses or maintains it. Each question separates the present from the future:

> In this version, `NB-1009` appears with a price of $0 because its cell is empty. Does anyone use that $0 for anything today, or do they ignore it? What do you want to happen from now on?

In class, the interview can be simulated: a classmate plays the salesperson or the catalog manager. That is valid, as long as every answer is marked as a simulation. An answer generated by the agent does not count as an answer from the business.

```text
ROLE
Act as an interviewer. You prepare questions and record answers; you do not
answer for the business.

CONTEXT
The ProductChat anomaly log has questions the code cannot answer: whether a
behavior is intentional, whether someone depends on it and what should be
kept. They are answered by whoever uses or maintains the system.

INPUT
discovery/anomalies.md and discovery/rules.md.

EXPECTED OUTPUT
discovery/interview.md with the questions ordered by impact, each one
addressed to a role (salesperson, catalog manager or maintainer), with the
answers received, the date and the classification change it causes in
anomalies.md.

CONSTRAINTS
Ask one question at a time and wait for the answer. Each question separates
what happens today from what should change. If the interview is simulated
in class, mark each answer as SIMULATION. Do not invent answers, people or
dates.

ACCEPTANCE CRITERION
Every question has a recorded answer or is marked as pending, and
anomalies.md reflects the classification changes.
```

> This prompt is in `prompts/12.4-interview.txt`, ready to copy. If you change it, change it there: that file is the reference version.

## Characterization tests

A **characterization test** records what the system does today for a known input, so that any later change is detected. It does not ask whether the result is correct, but whether it is still the same. That is why it can, and sometimes must, protect a behavior that looks like an error: until someone decides to change it, it is the system's behavior.

In Python, with `pytest`, they look like this:

```python
"""Characterization tests for the received ProductChat snapshot.

They pin down what this version does today, not what it should do.
Expected values were observed on the baseline commit; change them only
through a recorded decision.
"""
from productchat.catalog import load_catalog, search, get_product  # adapt to the snapshot

CATALOG = load_catalog("product_catalog.xlsx")


def test_br01_max_price_is_inclusive():
    """BR-01: a product priced exactly at the maximum is returned."""
    result = search(CATALOG, category="peripherals", max_price=34.99)
    assert {item.sku for item in result} == {"PER-4001", "PER-4002"}


def test_a01_missing_price_shown_as_zero():
    """A-01: NB-1009 has no price and this version shows it as 0. Kept on purpose."""
    assert get_product(CATALOG, "NB-1009").price == 0
```

The module and function names are examples: each received version has its own. What does not change are three rules.

**Expected values come from observation.** The received version was run, the result was noted, and that result is written as is in the test. It is never computed inside the test, neither by calling the same function nor by reproducing its algorithm.

**You test the deterministic layer, not the conversation.** The tests target the search and the normalization, not the text written by the language model, which can change from one run to the next. That separation was decided in the Plan in module 04, and here it pays off.

**A failing test is not fixed by changing the expected value.** First you find out what changed: the environment, the data or the behavior.

If an output is large and fully deterministic, there is a variant called a *golden master*: you save the full reviewed output in a file and the tests compare against it. ProductChat does not need it.

```text
ROLE
Act as a test developer. You pin down the current behavior; you do not fix
it or judge it.

CONTEXT
Before changing ProductChat, you need to protect what it does today,
including its odd behaviors. A characterization test records the output the
received version produces for a known input.

INPUT
discovery/rules.md, discovery/anomalies.md, the code and
product_catalog.xlsx.

EXPECTED OUTPUT
tests/test_characterization.py with one test for each BR rule and each
executable A anomaly, using the system's real entry point and the real
catalog. Each test states in its docstring the rule or anomaly it protects.

CONSTRAINTS
Expected values come from running the received version: do not compute them
inside the test or reproduce the algorithm. If the chatbot's answer
includes generated text, compare only the stable data (SKUs, prices,
warnings), not the full text. Use the existing runner and do not change
production code.

ACCEPTANCE CRITERION
All tests pass on the received version, and none would pass if the behavior
it protects changed.
```

> This prompt is in `prompts/12.5-characterization.txt`, ready to copy. If you change it, change it there: that file is the reference version.

## The reverse spec

With the discovery, the interview and the tests, you can now write the Spec. It differs from the ones in the rest of the course in one key point: it separates four things that do not coexist in a normal Spec.

| Part | What it contains |
|---|---|
| Current requirements | What the business confirmed the system must keep doing |
| Observed behaviors | What the system does and nobody has confirmed yet |
| Known anomalies | What looks like an error, with its reference to the log |
| Proposed changes | What should change from now on, separate from the current state |

Spec Kit helps you write it, but the cycle stops there: there is no Plan, no Tasks and no Implementation, because the system already exists.

```text
/speckit-specify

ROLE
Act as an analyst who documents an existing system. You describe what it
does today; you do not specify a new feature.

CONTEXT
ProductChat is in production without a specification. Discovery has already
rebuilt its rules, its anomalies and the decisions from the interview, and
the characterization tests protect its current behavior.

INPUT
The files in discovery/, tests/test_characterization.py and
product_catalog.xlsx.

EXPECTED OUTPUT
A spec.md for the catalog query with four separate parts: current
requirements confirmed in the interview, observed behaviors that no one has
confirmed, known anomalies with their A reference, and proposed changes for
the future. The acceptance criteria cite characterization tests.

CONSTRAINTS
Do not turn an accidental anomaly into a requirement. Do not declare
confirmed what only a simulation answered. Do not generate a plan, tasks or
code.

ACCEPTANCE CRITERION
Every current requirement points to a passing test, and every anomaly in
anomalies.md appears in the Spec.
```

> This prompt is in `prompts/12.6-reverse-spec.txt`, ready to copy. If you change it, change it there: that file is the reference version.

After generating it, check with `git diff` that only documents changed. A discovery phase that modifies the code is no longer discovery.

## The Spec Pack

The **Spec Pack** is the final deliverable: everything another person needs to understand the system and change it safely, gathered and linked from a single document.

```text
SPEC-PACK.md                    ← entry point
discovery/
├── map.md
├── query-flow.md
├── rules.md
├── anomalies.md
└── interview.md
specs/
└── 001-…/spec.md               ← the reverse spec
tests/
└── test_characterization.py
```

```text
ROLE
Act as the person responsible for the handover. You assemble what exists;
you do not add new evidence.

CONTEXT
The discovery of ProductChat is finished. You need to hand someone else
everything they need to understand the system and change it safely.

INPUT
The files in discovery/, the reverse spec in specs/ and
tests/test_characterization.py.

EXPECTED OUTPUT
SPEC-PACK.md at the root, with: the version studied and the scope; a
one-page summary for a non-technical reader; an index with links to each
document, the Spec and the tests; a BR → A → test traceability table; the
commands to run the tests; and what was left unverified.

CONSTRAINTS
Link only files that exist. Do not declare the system fully documented if
the scope does not support it.

ACCEPTANCE CRITERION
All links open, the test commands run, and SPEC-PACK.md explicitly states
what was left out of scope or unverified.
```

> This prompt is in `prompts/12.7-spec-pack.txt`, ready to copy. If you change it, change it there: that file is the reference version.

A good Spec Pack is honest about its limits. Having tested the search is not the same as having tested the whole chatbot, and the document must say so.

## Final course activity

Do the full discovery of your classmate's ProductChat: map and flow, rules, anomalies, interview, characterization tests, reverse spec and Spec Pack. Throughout the activity, the production code does not change.

When you finish, and only then, compare your reconstruction with your classmate's original Specs. The differences are of two kinds, and both teach something. If your reconstruction missed a rule that was in the Spec, the code does not make it clearly visible. If the code does something the Spec did not say, you found a drift, like the ones in module 10, from the other side.

| What is delivered | How it is checked |
|---|---|
| Preservation | The first commit is the received version, and the production code did not change afterward |
| Reconstruction | Each rule cites a SKU and its evidence, observed or inferred |
| Classification | What is observed, what is confirmed and what is proposed appear separately |
| Characterization | The tests pass on the received version and each one references its rule or anomaly |
| Delivery | The Spec Pack links everything and states what was left unverified |
| Comparison | A final paragraph with the differences between your reconstruction and the original Specs |

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
