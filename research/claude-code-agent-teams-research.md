# Claude Code Agent Teams: Configuration, Best Practices, and Effective Setups

**Research Date:** 2026-03-28
**Status:** Comprehensive research covering official docs, community guides, and expert analysis

---

## Table of Contents

1. [Overview and Architecture](#1-overview-and-architecture)
2. [Setup and Configuration](#2-setup-and-configuration)
3. [Team Role Definitions and Structures](#3-team-role-definitions-and-structures)
4. [CLAUDE.md Configurations for Agent Teams](#4-claudemd-configurations-for-agent-teams)
5. [Orchestration Patterns](#5-orchestration-patterns)
6. [Non-Coding Use Cases](#6-non-coding-use-cases)
7. [Known Limitations and Workarounds](#7-known-limitations-and-workarounds)
8. [Quality Optimization Strategies](#8-quality-optimization-strategies)
9. [Prompt Templates](#9-prompt-templates)
10. [Key Findings and Recommendations](#10-key-findings-and-recommendations)
11. [Sources](#11-sources)

---

## 1. Overview and Architecture

Agent Teams is an experimental feature (released February 5, 2026, alongside Opus 4.6) that coordinates multiple Claude Code instances as a team.

### Core Architecture

| Component     | Role                                                                   |
|---------------|------------------------------------------------------------------------|
| **Team Lead** | Main session: creates team, spawns teammates, coordinates, synthesizes |
| **Teammates** | Independent Claude Code instances with own 1M token context windows    |
| **Task List** | Shared work items with dependency tracking and file locking            |
| **Mailbox**   | Peer-to-peer messaging system between all agents                       |

### Key Architectural Insight

The fundamental advantage over subagents: **teammates communicate directly with each other**, not just back to the lead. Subagents report results only to the parent; teammates share findings, challenge each other, and self-coordinate through the shared task list.

> "The core insight behind swarms is that LLMs perform worse as context expands -- the more information in the context window, the harder it is for the model to focus on what matters right now." -- Addy Osmani

### Storage Locations

- Team config: `~/.claude/teams/{team-name}/config.json`
- Task list: `~/.claude/tasks/{team-name}/`
- Config contains `members` array with each teammate's name, agent ID, and agent type

---

## 2. Setup and Configuration

### Basic Enablement

**Requirements:**
- Claude Code v2.1.32 or later
- Opus 4.6 model minimum

**Enable in settings.json:**
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Or per-session: `claude --teammate-mode in-process`

### Display Modes

Set in `~/.claude.json`:

```json
{
  "teammateMode": "in-process"
}
```

| Mode | Description | Requirements |
|------|-------------|--------------|
| `"auto"` (default) | Uses split panes if in tmux, otherwise in-process | None |
| `"in-process"` | All teammates in main terminal; Shift+Down to cycle | Any terminal |
| `"tmux"` | Each teammate gets own pane with own color | tmux or iTerm2 with `it2` CLI |

**Split panes NOT supported in:** VS Code integrated terminal, Windows Terminal, Ghostty.

### Permission Configuration

Pre-approve common operations before spawning to reduce permission prompt flooding:

```json
// ~/.claude/settings.json
{
  "permissions": {
    "allow": [
      "Read(**)",
      "Glob(**)",
      "Grep(**)",
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(npm run build)"
    ]
  }
}
```

All teammates inherit the lead's permission settings at spawn time. Individual modes can be changed after spawning but not at spawn time (current limitation).

---

## 3. Team Role Definitions and Structures

### Current State: Natural Language Roles

Currently, all teammates spawn as generic `general-purpose` agents. Specialization is achieved through natural language spawn prompts. There is a highly-requested feature (GitHub issue #24316, 26+ upvotes) to allow teammates to inherit from `.claude/agents/` definitions with tool restrictions, model selection, hooks, and skills.

### Effective Role Patterns

**Scope-Focused Approach (recommended for implementation):**
- Security Agent: vulnerability scanning, auth verification
- API Agent: endpoint design, validation, error handling
- Frontend Agent: UI patterns, accessibility, performance
- Test Agent: writes tests, validates coverage

**Builder-Validator Pattern (recommended for quality):**
- **Builder Agent**: Creates and modifies code within specified files
- **Validator Agent**: Reviews output in read-only mode, NEVER modifies source files
- Enforced via `disallowedTools` preventing validators from using Edit/Write tools
- Pairs create natural quality gates through dependency chains

**Research Team:**
- Context Gatherer: explores codebase, reads docs, collects relevant information
- Analyst: processes gathered context, identifies patterns and issues
- Devil's Advocate: challenges findings, looks for counter-evidence

**Content Production Team:**
- Researcher: identifies content gaps and gathers source material
- Writer: drafts based on research findings
- Quality Reviewer: checks clarity, accuracy, SEO

### Model Assignment Per Teammate

You can specify different models per teammate for cost optimization:

```
Create a team where:
- The debugger runs on Opus
- The UI performance agent runs on Sonnet
- The UX quality agent runs on Haiku
```

Additional cost tip: Run the **lead in fast mode** with standard speed on teammates -- this optimizes coordination costs while maintaining quality on implementation.

---

## 4. CLAUDE.md Configurations for Agent Teams

CLAUDE.md is the single most important configuration file for Agent Teams because **teammates load it automatically but do NOT inherit the lead's conversation history**. A well-structured CLAUDE.md drastically reduces per-teammate exploration cost.

### Essential CLAUDE.md Sections for Teams

#### Module Boundaries (Critical for preventing file conflicts)

```markdown
## Module Boundaries

| Module   | Directory     | Owner Agent Type | Notes                    |
|----------|---------------|------------------|--------------------------|
| API      | src/api/      | Backend Agent    | Each file is independent |
| Frontend | src/client/   | Frontend Agent   | Component library        |
| Tests    | tests/        | Test Agent       | Jest framework           |
| Docs     | docs/         | Docs Agent       | Static content           |
```

#### Operational Context

```markdown
## Project Context

- **Stack**: Node.js + Express + TypeORM + React
- **Entry point**: src/index.ts
- **Test command**: npm test
- **Lint command**: npm run lint
- **Build command**: npm run build
- **Database**: PostgreSQL 15
```

> "No teammate should need to ask the lead what the project is about."

#### Verification Criteria

```markdown
## Verification

All changes must pass:
- `npm test`
- `npm run lint`
- `npm run build`

Report results in format:
"[Agent Name] - [Task] - [Pass/Fail] - [Details]"
```

#### Agent Team Coordination Rules

```markdown
## Agent Team Rules

- Each teammate owns specific directories (see Module Boundaries)
- Do NOT modify files outside your assigned directory
- If you need a change in another module, message the owning teammate
- Mark shared files as "coordinate before editing"
- Report completion with specific metrics when possible
```

#### Team Plan Generation Meta-Prompt

```markdown
## Team Plan Generation

When I say "team plan: [feature]", generate:
1. Builder task per component with specific files and criteria
2. Validator task in read-only mode
3. TaskUpdate chaining validator behind builder

For integration, add one validation task blocked by ALL builders.

Format each with:
- **Files**: exact paths
- **Criteria**: measurable acceptance conditions
- **Constraints**: what agents must NOT do
```

### Self-Reporting Pattern

When CLAUDE.md includes specific verification criteria, teammates produce structured self-reports:

> "Removed 27 console.log across 3 files. Kept all 12 console.error and 2 console.warn in component-page.js. Verified zero console.log remaining in assigned files."

---

## 5. Orchestration Patterns

### Pattern 1: Parallel Specialists

Multiple agents simultaneously work on different domains of the same problem.

**Best for:** Code review, research, multi-module features

```
Create an agent team to review PR #142:
- One focused on security implications
- One checking performance impact
- One validating test coverage
Have them each review and report findings.
```

### Pattern 2: Competing Hypotheses (Adversarial Debate)

Multiple investigators test different theories in parallel, actively trying to disprove each other.

**Best for:** Debugging, root cause analysis, architecture decisions

```
Users report the app exits after one message. Spawn 5 agent teammates
to investigate different hypotheses. Have them talk to each other to
disprove each other's theories, like a scientific debate. Update findings
with whatever consensus emerges.
```

> "Sequential investigation suffers from anchoring: once one theory is explored, subsequent investigation is biased toward it."

### Pattern 3: Pipeline Dependencies

Sequential task stages where completion auto-unblocks downstream work.

```
Phase 1: Parallel builders execute simultaneously
Phase 2: Validators block on their builders (addBlockedBy)
Phase 3: Integration validation blocked by ALL builders
```

### Pattern 4: Self-Organizing Swarms

Workers continuously claim available tasks from an independent pool, naturally load-balancing.

**Best for:** Large batch processing, data-parallel work (e.g., classifying 500 items)

### Pattern 5: Builder-Validator Chains

```
Builder (implements) -> Validator (reviews, read-only) -> Fix Builder (if issues) -> Fresh Validator
```

Each cycle narrows scope, converging toward correctness without manual debugging.

### Pattern 6: Research-then-Implement

Synchronous research phase returns results that guide subsequent implementation tasks.

```
Spawn researchers to investigate three different approaches to [problem].
Have them debate findings. Once consensus emerges, spawn implementation
teammates based on the winning approach.
```

### Lead Coordination: Delegate Mode

**Critical:** Enable delegate mode (`Shift+Tab`) immediately after starting a team. Without it, the lead often grabs tasks that teammates should handle.

- Restricts lead to coordination-only tools (spawning, messaging, task management)
- Prevents lead from writing code
- Most effective for teams of 4+ members
- Forces proper orchestration behavior

### Plan Approval Workflow

For complex or risky tasks:

```
Spawn an architect teammate to refactor the authentication module.
Require plan approval before they make any changes.
```

Plan mode is re-evaluated on every turn. Once spawned in a mode, teammates cannot transition. For plan-then-implement workflows, spawn new default-mode teammates for execution after planning approval.

---

## 6. Non-Coding Use Cases

### Research and Analysis

Agent teams are explicitly recommended as the **starting point** for new users, even before coding tasks:

> "If you're new to agent teams, start with tasks that have clear boundaries and don't require writing code: reviewing a PR, researching a library, or investigating a bug."

**Research Sprint Example:**
```
Create an agent team to research [topic]:
- Teammate 1: Investigate approach A, gather evidence
- Teammate 2: Investigate approach B, gather evidence
- Teammate 3: Play devil's advocate, challenge both approaches
Have them debate findings and produce a synthesis document.
```

### Writing and Content Production

**Content Pipeline:**
- Researcher identifies content gaps
- Writer drafts based on research
- Quality Reviewer checks clarity/proof/SEO
- Tasks chained so researcher finishes before writer starts

**Campaign Research Sprint:**
- Three researchers: competitor positioning, voice-of-customer insights, stress-test messaging
- Each teammate's output feeds directly into others' analysis

### Architecture Decision Records

```
Three teammates each advocate for different technical approaches
(e.g., PostgreSQL vs. ClickHouse vs. MongoDB). They challenge each
other's arguments. The lead synthesizes a decision document from
the strongest points.
```

### Ad Creative / Copy Exploration

```
Four teammates develop competing hook angles with headline variations.
They debate which direction is strongest, producing battle-tested
creative through competitive pressure.
```

### Marketing Landing Page with Adversarial Review

```
- Copywriter develops messaging
- CRO specialist designs conversion flow
- Skeptical buyer challenges weak claims and friction points
Require plan approval before implementation.
```

### Data Classification

Split data-parallel work by segments -- each teammate processes a portion independently using consistent taxonomy, flagging edge cases for human review.

---

## 7. Known Limitations and Workarounds

### Critical Limitations

| Limitation | Impact | Workaround |
|------------|--------|------------|
| **No session resumption** | `/resume` and `/rewind` do not restore in-process teammates | Tell lead to spawn new teammates after resuming |
| **Task status lag** | Teammates sometimes fail to mark tasks complete, blocking dependents | Manually check and update, or tell lead to nudge |
| **One team per session** | Cannot run multiple teams simultaneously | Clean up current team before starting new one |
| **No nested teams** | Teammates cannot spawn their own teams | Only lead manages hierarchy |
| **Fixed leadership** | Cannot promote teammate to lead | Session that creates team is lead for lifetime |
| **Permissions locked at spawn** | All teammates start with lead's permission mode | Change individual modes after spawning |
| **Slow shutdown** | Teammates finish current request before shutting down | Plan for graceful exit time |
| **No per-teammate customization** | All spawn as generic `general-purpose` agents | Use natural language prompts; feature request #24316 pending |

### Workarounds and Tips

**Lead doing work instead of delegating:**
```
Wait for your teammates to complete their tasks before proceeding.
Delegate this to your teammates.
```

**Excessive permission prompts:**
Pre-approve common operations in permission settings BEFORE spawning teammates.

**Teammates stopping on errors:**
Give direct instructions via Shift+Up/Down, or spawn replacement teammates.

**File conflicts between teammates:**
Define explicit directory boundaries in spawn prompts AND in CLAUDE.md. Never let two agents edit the same file.

**Orphaned tmux sessions:**
```bash
tmux ls
tmux kill-session -t <session-name>
```

**Context loss (teammates don't get lead's history):**
Embed important details into task descriptions and spawn prompts. The more specific the prompt, the less back-and-forth needed.

**Cost management:**
- Assign Haiku to read-only/review agents, Opus to implementation agents
- Run lead in fast mode
- Start with 3 teammates, not 5+
- Read-heavy tasks are more cost-effective than write-heavy ones

---

## 8. Quality Optimization Strategies

### Quality Gates with Hooks

Three hook events enforce automated quality:

**TeammateIdle Hook:**
Runs when a teammate finishes before others. Exit code 2 assigns follow-up tasks.

**TaskCreated Hook:**
Runs when tasks are being created. Exit code 2 prevents creation with feedback.

**TaskCompleted Hook:**
Runs when marking tasks complete. Exit code 2 blocks completion and sends corrective feedback.

> "No task closes with broken tests, regardless of which teammate worked on it."

### Seven Quality Principles

1. **Upfront clarity** -- Specific prompts reduce token waste and iteration
2. **Progressive complexity** -- Master research/review before implementation
3. **Boundary definition** -- Explicit file ownership eliminates silent failures
4. **Regular checkpoints** -- Interrupt long-running teammates; verify alignment
5. **CLAUDE.md leverage** -- Codify rules to enable autonomous self-reporting
6. **Sequential phases** -- Multiple small teams outperform one large team
7. **Supervised operation** -- Maintain active management role throughout execution

### The 80/20 Rule

> "80% planning and review, 20% execution." -- Compound Engineering Plugin approach

Better specifications produce better parallel execution. Codifying learnings helps subsequent agents avoid redundant work.

### Progressive Learning Path

- **Week 1:** Parallel code review (low risk, high learning)
- **Week 2:** Debugging with debate (teaches inter-agent communication)
- **Week 3:** Feature implementation with file boundaries (builds confidence)

---

## 9. Prompt Templates

### Reusable Spawn Templates

Create standardized prompt templates for common team configurations:

**Review Team Template:**
```
Create an agent team to review [target]:
- Security reviewer: Focus on [auth/input validation/data handling]
- Performance reviewer: Focus on [queries/rendering/memory]
- Test coverage reviewer: Focus on [edge cases/error paths/integration]
Each reviewer reports findings with severity ratings. Lead synthesizes.
```

**Implementation Team Template:**
```
Create a team with [N] teammates to implement [feature]:
- Backend: owns src/api/ and src/models/ -- [specific requirements]
- Frontend: owns src/components/ and src/pages/ -- [specific requirements]
- Tests: owns tests/ -- [specific requirements]
Use Opus for backend, Sonnet for frontend and tests.
Require plan approval before implementation.
```

**Research Team Template:**
```
Create an agent team to research [topic]:
- Researcher A: Investigate [angle 1], gather evidence from [sources]
- Researcher B: Investigate [angle 2], gather evidence from [sources]
- Devil's Advocate: Challenge all findings, look for counter-evidence
Have them debate findings and produce a synthesis document.
```

**Debugging Team Template:**
```
[Bug description]. Spawn [N] agent teammates to investigate different
hypotheses. Have them talk to each other to disprove each other's
theories. Update findings with whatever consensus emerges.
```

### Key Prompt Elements

- **Specific roles** beat vague instructions ("one on security" not just "reviewers")
- **File boundaries** prevent conflicts in implementation tasks
- **Success criteria** provide clear finish lines
- **Delegate mode** prevents lead from duplicating teammate work
- **Plan approval** catches bad directions before token waste
- **Debate structures** outperform consensus-seeking

---

## 10. Key Findings and Recommendations

### What Works Best

1. **Research and review tasks** are the strongest use case and best starting point
2. **3-5 teammates** is the optimal team size for most workflows
3. **5-6 tasks per teammate** keeps everyone productive with natural check-in points
4. **Delegate mode** should be enabled immediately for teams of 4+
5. **CLAUDE.md** is the single most important lever for team quality -- it's shared context
6. **Builder-Validator patterns** with dependency chains produce highest quality output
7. **Adversarial debate** structures outperform consensus-seeking for research
8. **File boundary definition** is the single most important rule for implementation teams
9. **Read-heavy tasks** (research, review, analysis) are more cost-effective than write-heavy
10. **Progressive complexity** (research -> review -> implementation) is the recommended learning path

### What Doesn't Work

1. Sequential tasks with heavy dependencies -- use single session or subagents
2. Same-file edits by multiple teammates -- leads to overwrites
3. "Set it and forget it" -- teams need active monitoring and steering
4. Vague prompts like "build me an app" -- burns tokens while agents flail
5. Large teams (5+) without genuine parallelization need
6. Tasks without clear boundaries or deliverables

### Anthropic's Measured Results

- 67% more PRs merged per engineer daily
- 0-20% fully delegated tasks (collaboration remains essential)
- 27% new work (tasks that wouldn't exist without AI assistance)

### The Compound Engineering Approach

> "'Build me an app' burns tokens while agents flail. 'Implement these five clearly-defined API endpoints according to this specification' produces something good."

The philosophy: precise specifications, explicit file boundaries, and codified learnings produce the best multi-agent output. Activity doesn't always translate to value.

---

## 11. Sources

### Official Documentation
- [Orchestrate teams of Claude Code sessions - Anthropic Docs](https://code.claude.com/docs/en/agent-teams)

### Expert Analysis
- [AddyOsmani.com - Claude Code Swarms](https://addyosmani.com/blog/claude-code-agent-teams/)
- [AddyOsmani.com - The Code Agent Orchestra](https://addyosmani.com/blog/code-agent-orchestra/)
- [AddyOsmani.com - The Future of Agentic Coding](https://addyosmani.com/blog/future-agentic-coding/)

### Comprehensive Guides
- [30 Tips for Claude Code Agent Teams - John Kim (Substack)](https://getpushtoprod.substack.com/p/30-tips-for-claude-code-agent-teams)
- [Claude Code Agent Teams: The Complete Guide 2026 - ClaudeFast](https://claudefa.st/blog/guide/agents/agent-teams)
- [Agent Teams Controls: Delegate Mode, Hooks & More - ClaudeFast](https://claudefa.st/blog/guide/agents/agent-teams-controls)
- [Team Orchestration: Builder-Validator Patterns - ClaudeFast](https://claudefa.st/blog/guide/agents/team-orchestration)
- [Agent Teams Best Practices & Troubleshooting - ClaudeFast](https://claudefa.st/blog/guide/agents/agent-teams-best-practices)
- [Agent Teams Use Cases and Prompt Templates - ClaudeFast](https://claudefa.st/blog/guide/agents/agent-teams-use-cases)

### Community Guides
- [How to Set Up and Use Claude Code Agent Teams - Dara Sobaloju (Medium)](https://darasoba.medium.com/how-to-set-up-and-use-claude-code-agent-teams-and-actually-get-great-results-9a34f8648f6d)
- [Claude Code Swarm Orchestration Skill - Kieran Klaassen (GitHub Gist)](https://gist.github.com/kieranklaassen/4f2aba89594a4aea4ad64d753984b2ea)
- [claude-code-ultimate-guide - Florian Bruniaux (GitHub)](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/workflows/agent-teams.md)
- [From Tasks to Swarms - alexop.dev](https://alexop.dev/posts/from-tasks-to-swarms-agent-teams-in-claude-code/)

### Feature Requests and Development
- [Custom .claude/agents/ definitions for teammates - GitHub Issue #24316](https://github.com/anthropics/claude-code/issues/24316)

### Additional References
- [Shipyard - Multi-agent orchestration for Claude Code](https://shipyard.build/blog/claude-code-multi-agent/)
- [SitePoint - Claude Code Agent Teams Setup Guide](https://www.sitepoint.com/anthropic-claude-code-agent-teams/)
- [Building Agent Teams in OpenCode - DEV Community](https://dev.to/uenyioha/porting-claude-codes-agent-teams-to-opencode-4hol)
- [Sean Kim - Claude Code Team Mode March 2026 Update](https://blog.imseankim.com/claude-code-team-mode-multi-agent-orchestration-march-2026/)
