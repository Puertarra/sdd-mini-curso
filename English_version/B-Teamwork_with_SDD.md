# Teamwork with SDD

This annex gathers the rules and practices for applying SDD when the project is developed by several people, each with their own agent. It is worth reading when the group is formed, before writing the first Spec, and keeping it at hand throughout the project. It assumes you already know the cycle from the first part and the branches per Spec from module 09.

## The central idea

When you work alone, the intent of the product lives in your head. In a team, that no longer works: every person and every agent needs to read the same thing in order to build the same thing. The Spec is that written agreement, and everything that follows serves to protect it. It comes down to three rules:

1. **Every shared rule has a single owner.** The owner can be a person or a Spec, but never two at once.
2. **The Spec is reviewed before the code.** An error in the Spec costs minutes; the same error in the code costs days.
3. **Nothing goes into `main` without backing in a Spec.** It is the gate that protects the rest of the team.

## The team agreement

Before starting, the group makes a few decisions once. If they are not made explicitly, each person makes them in their own way and the repository fills up with different styles.

| Decision | Recommendation | Where it is recorded |
|---|---|---|
| Way of working with the Specs | *Flow-forward*: each change in a new Spec that declares what it replaces (module 10) | Constitution |
| Branches | Git branches, published on GitHub: one per Spec, named after its folder, for example `003-catalog-filters` | Constitution |
| Commit messages | In English, with the Spec prefix: `SPEC003: plan and tasks` | Constitution |
| Who approves a Spec | The product lead and a person other than the author (preferably the owner of a shared rule the Spec uses, or whoever will implement it) | Constitution |
| Who merges into `main` | The integrator on duty, never the author of the change | Constitution |
| Agents | Each person uses the one they prefer, Claude Code or Codex, with the same prompts (see "Teams with different agents") | `.specify/` and `AGENTS.md` |
| Where the Specs are registered | In the shared repository on GitHub; the index `specs/README.md` on `main` reserves the number of each one (see "Where the Specs live") | Repository |

The decisions the team does not negotiate go in the Constitution, which already plays that role for the agent. It is enough to add a section like this one to `.specify/memory/constitution.md`:

```text
## Team working rules

1. Every change in behavior starts in a Spec. No change in behavior is
   merged into main without a Spec that backs it.
2. Specs are reviewed and approved before planning. They are approved by the
   product lead and a person other than the author.
3. Every shared rule (prices, stock, categories, duplicates) has an owning
   Spec. The other Specs cite it; they do not redefine it.
4. One branch per Spec, named after its folder in specs/. One commit per
   stage, with the prefix SPECnnn.
5. The integrator on duty merges, never the author. Before merging: tests
   green and a drift audit with no pending findings.
```

Since the agent reads the Constitution before each stage, these rules bind it too: if you ask it to implement something without a Spec, it should point that out.

## Where the Specs live

When you work alone, the Specs are on your disk and nobody else touches them. In a team, each person works on their own computer, with their own copy of the repository. The version that counts for everyone is the one on GitHub: Specs travel from one computer to another through branches and *pull requests*, just like code. Whatever is only on your disk does not exist for the rest of the team.

That arrangement produces three concrete problems:

| Problem | Why it happens | How to avoid it |
|---|---|---|
| Two people create Spec 007 | `/speckit-specify` assigns the next free number it finds in your copy of `specs/`, and does not know what someone else created on their computer (module 11) | Reserve the number on `main` before specifying |
| Someone specifies on top of requirements that have already changed | Their copy of `main` is out of date | Update `main` before starting each Spec |
| A new Spec goes for days without anyone seeing it | It is only on its author's computer | Publish the branch as soon as `spec.md` exists |

The reservation is made in the index `specs/README.md`. Whoever wants to start a Spec updates `main`, adds their line to the index and publishes it. If GitHub rejects the change on publishing because someone else reserved first, they update again and take the next number:

```bash
git switch main
git pull
# add this line to the index: | SPEC007 | Sort results by price | Reserved | — |
git add specs/README.md
git commit -m "SPEC007: reserve number"
git push

git switch -c 007-sort-results
```

Then, when running `/speckit-specify`, the directory is set in the prompt, as in module 11: "The spec.md of SPEC007, in the directory `specs/007-sort-results` (use it as SPECIFY_FEATURE_DIRECTORY)". That way the number is decided by the shared index and not by each person's local copy.

One last rule for working together: you do not edit another person's Spec on their branch. Observations are made as comments on their *pull request*, and the author decides how to incorporate them.

## Teams with different agents

Having some members use Claude Code and others Codex is not a problem: Specs, plans and tasks are text files, and the course prompts work for both agents. Three things need to be prepared:

1. **Install both integrations in the same project.** Spec Kit 1.0.8 allows it without touching the Constitution or the Specs:

   ```bash
   specify init . --integration claude     # when creating the project
   specify integration install codex       # adds Codex to the same project
   specify integration status              # should list claude and codex
   ```

   The repository ends up with `.claude/skills/` and `.agents/skills/`, and each member invokes the operations with their agent's syntax: `/speckit-specify` in Claude Code, `$speckit-specify` in Codex. Do not use `specify init --force` on a project that already has a Constitution: it may overwrite it.
2. **Keep a single context file.** Codex reads `AGENTS.md` and Claude Code reads `CLAUDE.md`. To avoid maintaining two files that end up saying different things, write the context in `AGENTS.md` and leave in `CLAUDE.md` a single line that imports it: `@AGENTS.md`.
3. **Review everyone the same way.** Two different models write differently, and one may produce more detailed Specs than the other. The Spec review and the integration gate are the same for everyone, and they are what keep quality even.

## Roles

In a group of three or four people, roles are not fixed positions. It is worth rotating them with each delivery, so that everyone goes through all of them.

| Role | What it decides | What it does not decide |
|---|---|---|
| Product lead | What goes into each Spec, how open questions are resolved, which rule prevails when two clash | How it is implemented |
| Technical lead | The Plan: stack, data model, code structure | What the product does |
| Integrator | Whether a branch is ready to go into `main` | The content of the Spec or the code |
| Everyone | Specify, implement and review others' work | A shared rule that has another owner |

> **If you come from Scrum.** The product lead is the equivalent of the *Product Owner*. The difference is that here their decisions are written down in the Spec, because the agent that implements was not part of the conversation.

## The cycle of a change in a team

This is how a change travels through the repository, from the moment someone proposes it until it goes into `main`:

| Step | Who | What is done | Result |
|---|---|---|---|
| 1. Propose | Anyone | Describes the need in one line, with a concrete example from the catalog | An *issue* or an entry in the index |
| 2. Specify | The author | Reserves the number in the index, creates the branch and the Spec with `/speckit-specify` and `/speckit-clarify`, and publishes the branch | `spec.md` on GitHub |
| 3. Review the Spec | Product lead and a second person | Review only `specs/`, before any code exists | Approved Spec |
| 4. Plan | The author, with the technical lead | Plan, tasks and Analyze | `plan.md` and `tasks.md` |
| 5. Implement | The author and their agent | Implementation and tests per acceptance criterion | Code on the branch |
| 6. Request integration | The author | Opens the *pull request* with this annex's template | PR ready for review |
| 7. Integrate | The integrator | Passes the integration gate and does the merge | Change on `main` |
| 8. Update what is current | The integrator | Regenerates the current requirements view with `10.5` | `specs/current-requirements.md` |

Step 3 is the one most often skipped and the one that saves the most. You can open a *pull request* that contains only the Spec's folder: that way the review happens on GitHub, it is recorded, and nobody starts programming on a Spec the rest have not seen.

## Reviewing a Spec: when it is ready

Before planning, a Spec has to be **ready**: another person, or their agent, must be able to implement it without asking anything. Reviewing it is not correcting the wording, but checking that. This list works for any project:

1. **Purpose.** Is it clear what need it addresses and for whom, without reading the code?
2. **Verifiable criteria.** Does every acceptance criterion have a concrete input and an expected result that can be checked? In ProductChat, a SKU and what the chatbot must answer.
3. **Shared rules.** Are the rules that belong to another Spec cited, and not redefined?
4. **Declared changes.** If it modifies something that already existed, does it say which criteria of other Specs it replaces?
5. **Edge cases.** Does it say what happens with missing, empty or contradictory data?
6. **Open questions.** Are they all resolved? If any remains, the Spec is not ready.
7. **Scope.** Is it clear what is out of scope?

If any answer is no, the Spec goes back to its author. Returning a Spec costs minutes; returning a plan or code built on it costs much more.

> **If you come from Scrum.** This list plays the role of a *Definition of Ready*: the agreement some teams use to decide when a backlog item can go into a sprint. The integration gate in the next section, in turn, resembles the *Definition of Done*.

## The integration gate

Before each merge into `main`, the integrator checks four things:

1. **The tests pass,** including those of earlier Specs. A new Spec breaking a current criterion is the most common form of drift.
2. **Every change in `src/` is backed in `specs/`.** The following command compares the branch with `main` and warns if `src/` changed without any change in `specs/`:

   ```bash
   changed=$(git diff --name-only main...HEAD)
   if echo "$changed" | grep -q '^src/' && ! echo "$changed" | grep -q '^specs/'; then
     echo "Warning: src/ changed without any change in specs/"
   fi
   ```

   Not every code modification needs a Spec: an internal rename or an optimization does not change behavior. But the warning forces you to ask.
3. **The drift audit has no pending findings.** The prompt `prompts/10.2-drift-audit.txt` is run on the branch.
4. **The branch is up to date with `main`.** If it is not, it is updated first and the tests are run again.

If the team wants to automate steps 1 and 2, it can move them into an agent *hook* or into GitHub's continuous integration. It is worth doing only when the team already applies the gate by hand without forgetting it: automating a practice that does not exist yet does not create it.

## Pull request template

If the repository includes the file `.github/pull_request_template.md`, GitHub uses it as the starting text of every *pull request*:

```markdown
## Spec

- Spec: SPEC00X, `specs/00X-name/`
- Replaces criteria of other Specs: no / yes (which ones)

## What changes

One or two sentences, from the user's point of view.

## How it is verified

| Criterion | SKU | Expected result |
|---|---|---|
| AC1 | NB-1005 | Appears among the 32 GB notebooks |

## Integration gate

- [ ] The Spec was approved before implementing
- [ ] All tests pass, including those of earlier Specs
- [ ] Every change in src/ is backed in specs/
- [ ] The drift audit (10.2) has no pending findings
```

## When not everyone uses SDD

It is common for someone in the group not to want to, or not to manage to, use Spec Kit. You can still work, as long as the rule is applied to the result and not to the tool:

- **The Spec is required, not Spec Kit.** Whoever does not use the tool can write or edit `spec.md` by hand with the structure from module 03. What matters is that the change is backed.
- **The gate is the same for everyone.** A change without a Spec is not merged, no matter who it comes from.
- **The agent can also help whoever does not use SDD.** Ask it to draft the Spec from the change already made; it is a *reverse spec* in miniature, as in module 12.

When a change made directly in the code arrives, there are three ways out:

| Situation | What to do |
|---|---|
| The change is correct and you want to keep it | Write the Spec that backs it before merging |
| The change contradicts a current Spec | Do not merge. Decide with the rule's owner which one prevails and, if it changes, write a new Spec |
| There was an emergency and it is already on `main` | Accept it as a patch and write its Spec in the next work session, as in module 10 |

The goal is not to punish anyone. It is for the Spec to keep telling the truth, because the rest of the team, and their agents, work on top of it.

## Common situations

| Situation | What to do | Where it is covered |
|---|---|---|
| Two people created a Spec with the same number | The one reserved later in the index changes its number and folder before being merged | This annex, module 11 |
| Two people need to change the same rule | The rule's owner decides it, in a single Spec; the other person cites it | Module 11 |
| A teammate changed the code without a Spec | It is not merged until the Spec exists, or it is reverted | This annex, module 10 |
| An ambiguity in the Spec appears while implementing | Go back to the Spec and make it precise before continuing | Module 09, prompt `09.6` |
| Git reports a conflict | The correct version is decided by the Spec, not by whoever got there first | Annex A |
| Two current Specs contradict each other | It shows up in the conflicts section of the current view; it is resolved with a new Spec | Module 10, prompt `10.5` |
| A new person joins the team | Reads `specs/README.md`, `specs/current-requirements.md` and `CLAUDE.md`, in that order | Modules 09 and 10 |
| Each person uses a different agent | Both integrations in the same project and a single context file | This annex, modules 00 and 09 |
| Someone works with *vibe coding* and delivers something that works | Their Spec is reconstructed before merging | Module 12 |

## Warning signs

If any of these signs appears, the team is no longer working with Specs, even if it still believes it is:

- There are commits in `src/` without any commit in `specs/` for several days.
- Specs are approved after implementing, or nobody reviews them.
- The same rule (for example, what to do with a SKU with two prices) appears written in more than one Spec.
- The current requirements view has not been regenerated for several merges.
- Someone asks "so how did the stock thing end up?" and the answer is not in any Spec.

## Work rhythm

If the team works in *sprints* or in weekly deliveries, the practice is organized like this:

| Moment | What is done |
|---|---|
| Start of the sprint | Review and approve the Specs that will be worked on. It is the equivalent of the "what" of *Sprint Planning* (note in module 04) |
| During the sprint | One branch per Spec, commits per stage, Spec reviewed before implementing |
| Before each merge | Integration gate: tests, backing in `specs/` and audit `10.2` |
| After each merge | Regenerate `specs/current-requirements.md` with `10.5` |
| End of the sprint | Review the drift findings of the period: what caused them and which rule of the agreement needs adjusting? |

## Short activity

For groups of three or four people, before starting the project:

1. Write the team agreement and add it to the Constitution.
2. Assign the roles for the first delivery.
3. Take a small change through the eight steps of the cycle, for example, sorting the results of a search by price.

| What is delivered | How it is checked |
|---|---|
| Agreement | The Constitution has the team working rules section |
| Spec review | There is a *pull request* with only `specs/`, approved by two people before any code exists |
| Integration | The merge was done by a person other than the author, and the PR has the integration gate completed |
| Current view | `specs/current-requirements.md` includes the new change |

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
