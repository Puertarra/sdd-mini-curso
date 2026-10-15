# The Work Environment in SDD

## The two layers of the environment

To work with **Spec-Driven Development (SDD)** it helps to distinguish two layers that play different roles:

```text
┌──────────────────────────────────────┐
│             METHOD LAYER             │
│                                      │
│ GitHub Spec Kit                      │
│ OpenSpec                             │
│ Kiro                                 │
│                                      │
│ Organizes the SDD process            │
│ Specification → Plan → Tasks → ...   │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│              AGENT LAYER             │
│                                      │
│ Claude Code                          │
│ Codex                                │
│ Other agents / local models          │
│                                      │
│ Reasons, edits files,                │
│ codes, runs and verifies             │
└──────────────────────────────────────┘
```

The **method layer** defines how we organize development. It manages specifications, plans, tasks, quality checks and the overall SDD flow.

The **agent layer** is the engine that runs that method. The agent reads the artifacts, reasons about them, writes code, runs commands and verifies the results.

The separation matters because **method and agent are interchangeable**. We can use GitHub Spec Kit as the method and work with Claude Code, Codex or another agent without changing the core logic of the process.

In this course we will use:

```text
Method layer  → GitHub Spec Kit v1.0.8
Agent layer   → Claude Code or Codex
```

> **About the version.** Spec Kit evolves quickly, and the way its operations work can change from one version to the next. All the material in this course has been tested against **version 1.0.8**. If you install a different one, check the section "Check the integration" before going on.

## 1. Prepare the agent layer

Before installing Spec Kit we need a coding agent: Claude Code, Codex or another compatible agent.

Installation and authentication depend on the agent you choose. Once it is installed, check that you can start it from the terminal inside a project directory. For example, with Claude Code:

```bash
claude
```

or with Codex:

```bash
codex
```

Always start the agent from the directory of the project you want to work on.

## 2. Install uv

Spec Kit is distributed as a Python tool and is installed with **uv**, a Python package and environment manager. On macOS:

```bash
brew install uv
```

On other systems, or if you don't use Homebrew, follow the official uv installer. Check that it is available:

```bash
uv --version
```

## 3. Install GitHub Spec Kit

> ⚠️ **Before running this command, check the version.** This material is pinned to
> v1.0.8 and tested against it. Spec Kit is developed quickly, and **the way you invoke its
> operations is not stable**: in 1.0.8 it depends on the agent, and it does not even match the
> notation used in its own documentation. Before installing, check the repository's releases
> (`github.com/github/spec-kit`) and make a conscious decision: either you install v1.0.8 and the
> material works as is, or you install a newer one and accept that you will have to check the
> operation names and adapt the prompts. What you should not do is install `@main` without
> looking: it is a development branch and can change from one day to the next.

We install the version the course uses, pinning it explicitly:

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v1.0.8
```

This makes the `specify` command available. If the terminal says it cannot find it, the uv tools directory is not on your `PATH`:

```bash
uv tool update-shell
```

Then open a new terminal and check the installation:

```bash
specify --version
```

It should reply:

```text
specify 1.0.8
```

Spec Kit does not replace the agent. Its job is to provide the structure, templates, scripts and instructions that let the agent work following SDD.

## 4. Create a project

First we create a working directory:

```bash
mkdir sdd-course
cd sdd-course
```

Next we initialize a project, stating which agent we will use.

### With Claude Code

```bash
specify init project1 --integration claude
```

### With Codex

```bash
specify init project1 --integration codex
```

Spec Kit automatically prepares the project to work with the selected agent. If you want to see which integrations your installation supports:

```bash
specify check
```

## 5. Bring the case files into the project

We go into the project:

```bash
cd project1
```

and copy the two case files into it:

```text
project1/
├── product_catalog.xlsx
└── napkin_spec_productchat.md
```

This step matters more than it seems. Every prompt in the course declares its input files by name; if they are not in the project, the agent cannot read them and we end up pasting content by hand, which is exactly what we want to avoid.

Now we can start the agent inside the project:

```bash
claude
```

From this point on, the two layers work together:

```text
GitHub Spec Kit
      │
      │ provides the method,
      │ templates, scripts and instructions
      ▼
AI agent
      │
      │ interprets and executes
      ▼
Project
```

## 6. What does Spec Kit add to the project?

When you initialize it, two groups of files appear, all of them hidden.

The first depends on the agent. With Claude Code, Spec Kit installs its instructions as **skills** in `.claude/skills/`. With Codex, in `.agents/skills/`. These are the instructions the agent will follow at each SDD stage.

The second is Spec Kit's own resources, in `.specify/`:

```text
.specify/
├── memory/         ← constitution.md will live here
├── templates/      ← structures of the artifacts
├── scripts/        ← helper operations
├── integrations/   ← configuration of the chosen agent
└── workflows/
```

The `templates` provide the structure of each artifact:

```text
constitution-template.md
spec-template.md
plan-template.md
tasks-template.md
checklist-template.md
```

You do not need to edit these files by hand. The agent will use them when we run the different stages.

## 7. Check the integration

Once the agent is running inside the project, we check that the Spec Kit operations are available. How you write them depends on the agent:

| Agent | How to invoke | Where Spec Kit installs them |
|---|---|---|
| Claude Code | `/speckit-specify` | `.claude/skills/` |
| Codex | `$speckit-specify` | `.agents/skills/` |

The course writes operations in the Claude Code form. If you use Codex, replace the slash with the dollar sign, `$speckit-plan` instead of `/speckit-plan`; the rest of each prompt stays the same.

You will come across a third form, with a dot: `/speckit.specify`. It is the reference notation in the official Spec Kit documentation and in many tutorials, but it is not what you type in your agent with version 1.0.8.

In Claude Code, the course operations are these:

```text
/speckit-constitution
/speckit-specify
/speckit-clarify
/speckit-plan
/speckit-tasks
/speckit-analyze
/speckit-implement
```

It is worth understanding clearly what each thing is, because they are easy to confuse. What Spec Kit installs in the project is a **skill**: an instructions file the agent will read. What we write is the invocation of that skill: in Claude Code, a **slash command**. We invoke it explicitly so that the action is explicit, both for the agent and for whoever later reads what was done. Typing `/speckit-plan` leaves a record that the planning stage was run; asking for it in plain language does not.

So when we write:

```text
/speckit-specify
```

we are not switching agents. We are still using Claude Code, Codex or whichever one we chose, but now that agent receives the instructions provided by the **Spec Kit method layer**.

This separation can be summarized as:

```text
Spec Kit says WHAT PROCESS to follow.
The agent carries out that process.
```

Spec Kit 1.0.8 also installs `/speckit-checklist`, which we will use in module 08; `/speckit-converge`, which evaluates the existing code and adds the pending work as tasks, and which appears in module 10; and `/speckit-taskstoissues`, which is out of scope for the course.

> **If you installed a different version.** Operation names and the way to invoke them can change between Spec Kit versions. You can see the exact names in your installation by listing the corresponding directory, `ls .claude/skills/` or `ls .agents/skills/`, and substitute them in the course prompts.

## Environment ready

Once both layers are set up, our environment looks like this:

```text
project1
│
├── GitHub Spec Kit           ← Method layer
│   ├── Skills
│   ├── Templates
│   └── Scripts
│
├── Claude Code / Codex       ← Agent layer
│   ├── Reasons
│   ├── Edits files
│   ├── Generates code
│   ├── Runs
│   └── Verifies
│
└── Case files
    ├── product_catalog.xlsx
    └── napkin_spec_productchat.md
```

With this environment ready we can start the Spec-Driven Development cycle:

`Constitution → Specification → Clarify → Plan → Tasks → Analyze → Implementation`

---

© 2026 Andrés Leiva-Araos · [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) · [github.com/Puertarra/sdd-mini-curso](https://github.com/Puertarra/sdd-mini-curso)
