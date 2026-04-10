# Superpowers

Superpowers is a complete software development workflow for your coding agents, built on top of a set of composable "skills" and some initial instructions that make sure your agent uses them.

## How it works

It starts from the moment you fire up your coding agent. As soon as it sees that you're building something, it *doesn't* just jump into trying to write code. Instead, it steps back and asks you what you're really trying to do. 

Once it's teased a spec out of the conversation, it shows it to you in chunks short enough to actually read and digest. 

After you've signed off on the design, your agent puts together an implementation plan that's clear enough for an enthusiastic junior engineer with poor taste, no judgement, no project context, and an aversion to testing to follow. It emphasizes true red/green TDD, YAGNI (You Aren't Gonna Need It), and DRY. 

Next up, once you say "go", it launches a *subagent-driven-development* process, having agents work through each engineering task, inspecting and reviewing their work, and continuing forward. It's not uncommon for Claude to be able to work autonomously for a couple hours at a time without deviating from the plan you put together.

There's a bunch more to it, but that's the core of the system. And because the skills trigger automatically, you don't need to do anything special. Your coding agent just has Superpowers.


## Sponsorship

If Superpowers has helped you do stuff that makes money and you are so inclined, I'd greatly appreciate it if you'd consider [sponsoring my opensource work](https://github.com/sponsors/obra).

Thanks! 

- Jesse


## Installation

**Note:** Installation differs by platform. There are two versions of superpowers:

- **Upstream (obra/superpowers)** -- the original, marketplace-installable version
- **O-type-Labs fork** -- extended with gstack integration, research-context, ship-gate, and release-report (this repo)

### O-type-Labs Fork (recommended -- includes gstack integration)

This fork adds 3 new skills and 15 gstack integration points to the superpowers pipeline. When you brainstorm, it runs gstack plan reviews. When you debug, it uses gstack's headless browser. When you ship, it delegates to gstack's release automation. Everything is conditional -- if gstack isn't installed, superpowers works normally.

**Extra skills in this fork:**
- **research-context** -- 10-dimension context gathering with visual gap dashboard, gates on 80% coverage before brainstorming
- **ship-gate** -- full pre-merge verification pipeline: spec compliance, smoke tests, CI watch, confidence scoring, fix loop
- **release-report** -- 3-level visual summary (executive/architecture/file-detail) exported to ~/Downloads after merge

#### Step 1: Clone and point Claude Code at the fork

```bash
git clone https://github.com/O-type-Labs/superpowers.git ~/Desktop/ai/superpowers
```

Then tell Claude Code to use it. Open `~/.claude/plugins/installed_plugins.json` and set (or add) the superpowers entry:

```json
{
  "version": 2,
  "plugins": {
    "superpowers@claude-plugins-official": [
      {
        "scope": "user",
        "installPath": "/Users/<your-username>/Desktop/ai/superpowers",
        "version": "5.0.7",
        "installedAt": "2026-01-01T00:00:00.000Z",
        "lastUpdated": "2026-01-01T00:00:00.000Z",
        "gitCommitSha": ""
      }
    ]
  }
}
```

Replace `<your-username>` with your macOS username. The path must be absolute.

If you already have superpowers installed from the marketplace, just change the `installPath` -- all other fields can stay.

**Restart Claude Code** (quit and reopen). The next session loads from the local clone.

#### Step 2: Install gstack (auto or manual)

gstack auto-installs on your first session after Step 1 (the session-start hook clones it). But if you want it immediately, or the auto-install was slow, install manually:

**Prerequisites:**
- [Bun](https://bun.sh/) v1.0+ (required for gstack's headless browser daemon)
- [Git](https://git-scm.com/)
- macOS, Linux, or Windows with Git Bash

**Install Bun first (if you don't have it):**

```bash
curl -fsSL https://bun.sh/install | bash
```

After install, **restart your terminal** (or run `source ~/.zshrc`) so `bun` is on your PATH. Verify:

```bash
bun --version    # should print 1.x.x
```

**macOS troubleshooting:** If `bun` installs but commands hang or the gstack setup takes forever:

1. Make sure you're on a recent macOS (13+). Older versions have issues with Bun's binary.
2. If `bun install` hangs, try:
   ```bash
   # Clear Bun's cache
   rm -rf ~/.bun/install/cache
   
   # Retry
   cd ~/.claude/skills/gstack && bun install
   ```
3. If gstack's `./setup` script hangs during the Bun compile step (`bun build --compile`), it's building the browse daemon binary (~58MB). This is a one-time operation. On M1/M2/M3 Macs it takes 30-60 seconds. On Intel Macs or slow connections it can take 2-3 minutes. Let it finish.
4. If you see `error: could not resolve "..."` during setup, your Bun cache may be stale:
   ```bash
   bun pm cache rm
   cd ~/.claude/skills/gstack && ./setup
   ```

**Install gstack:**

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

The `./setup` script:
1. Installs npm dependencies via Bun
2. Compiles the headless browser daemon (`bun build --compile` -- the slow part, ~30-60s)
3. Sets up configuration at `~/.gstack/`
4. Verifies everything works

**Verify gstack installed:**

```bash
ls ~/.claude/skills/gstack/bin/gstack-*    # should list several binaries
```

#### Step 3: Install Codex CLI (optional, for adversarial code review)

The `requesting-code-review` skill can invoke Codex for an independent second opinion from GPT. This is optional -- skip if you don't have a ChatGPT Plus/Pro subscription.

```bash
npm install -g @openai/codex
codex login    # sign in with your ChatGPT account
```

#### Step 4: Verify the full stack

Start a new Claude Code session. You should see all three frameworks in the skill list:

```
superpowers:brainstorming        -- from your local clone
superpowers:research-context     -- new in this fork
superpowers:ship-gate            -- new in this fork
superpowers:release-report       -- new in this fork
gstack-health                    -- auto-installed or manually installed
gstack-browse                    -- from gstack
gstack-ship                      -- from gstack
... (37 gstack skills total)
```

Type `/brainstorming` on any task. It should:
1. Start with research-context (step 0) -- gap dashboard appears
2. Run `gstack-health` during autonomous scan -- composite score shown
3. After design approval, offer gstack plan reviews (CEO/eng/design/DX)

If all three fire, the integration is working.

#### Updating

To update this fork:
```bash
cd ~/Desktop/ai/superpowers && git pull
```

To update gstack:
```bash
cd ~/.claude/skills/gstack && git pull && ./setup
```

Superpowers and gstack update independently. Neither overwrites the other.

---

### Upstream Installation (obra/superpowers -- original, no gstack)

If you don't want the gstack integration and prefer the vanilla superpowers:

#### Claude Code Official Marketplace

Superpowers is available via the [official Claude plugin marketplace](https://claude.com/plugins/superpowers)

Install the plugin from Claude marketplace:

```bash
/plugin install superpowers@claude-plugins-official
```

### Claude Code (via Plugin Marketplace)

In Claude Code, register the marketplace first:

```bash
/plugin marketplace add obra/superpowers-marketplace
```

Then install the plugin from this marketplace:

```bash
/plugin install superpowers@superpowers-marketplace
```

### Cursor (via Plugin Marketplace)

In Cursor Agent chat, install from marketplace:

```text
/add-plugin superpowers
```

or search for "superpowers" in the plugin marketplace.

### Codex

Tell Codex:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.codex/INSTALL.md
```

**Detailed docs:** [docs/README.codex.md](docs/README.codex.md)

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md
```

**Detailed docs:** [docs/README.opencode.md](docs/README.opencode.md)

### GitHub Copilot CLI

```bash
copilot plugin marketplace add obra/superpowers-marketplace
copilot plugin install superpowers@superpowers-marketplace
```

### Gemini CLI

```bash
gemini extensions install https://github.com/obra/superpowers
```

To update:

```bash
gemini extensions update superpowers
```

### Verify Installation

Start a new session in your chosen platform and ask for something that should trigger a skill (for example, "help me plan this feature" or "let's debug this issue"). The agent should automatically invoke the relevant superpowers skill.

## The Basic Workflow

1. **brainstorming** - Activates before writing code. Refines rough ideas through questions, explores alternatives, presents design in sections for validation. Saves design document.

2. **using-git-worktrees** - Activates after design approval. Creates isolated workspace on new branch, runs project setup, verifies clean test baseline.

3. **writing-plans** - Activates with approved design. Breaks work into bite-sized tasks (2-5 minutes each). Every task has exact file paths, complete code, verification steps.

4. **subagent-driven-development** or **executing-plans** - Activates with plan. Dispatches fresh subagent per task with two-stage review (spec compliance, then code quality), or executes in batches with human checkpoints.

5. **test-driven-development** - Activates during implementation. Enforces RED-GREEN-REFACTOR: write failing test, watch it fail, write minimal code, watch it pass, commit. Deletes code written before tests.

6. **requesting-code-review** - Activates between tasks. Reviews against plan, reports issues by severity. Critical issues block progress.

7. **finishing-a-development-branch** - Activates when tasks complete. Verifies tests, presents options (merge/PR/keep/discard), cleans up worktree.

**The agent checks for relevant skills before any task.** Mandatory workflows, not suggestions.

## What's Inside

### Skills Library

**Testing**
- **test-driven-development** - RED-GREEN-REFACTOR cycle (includes testing anti-patterns reference)

**Debugging**
- **systematic-debugging** - 4-phase root cause process (includes root-cause-tracing, defense-in-depth, condition-based-waiting techniques)
- **verification-before-completion** - Ensure it's actually fixed

**Collaboration** 
- **brainstorming** - Socratic design refinement
- **writing-plans** - Detailed implementation plans
- **executing-plans** - Batch execution with checkpoints
- **dispatching-parallel-agents** - Concurrent subagent workflows
- **requesting-code-review** - Pre-review checklist
- **receiving-code-review** - Responding to feedback
- **using-git-worktrees** - Parallel development branches
- **finishing-a-development-branch** - Merge/PR decision workflow
- **subagent-driven-development** - Fast iteration with two-stage review (spec compliance, then code quality)

**Meta**
- **writing-skills** - Create new skills following best practices (includes testing methodology)
- **using-superpowers** - Introduction to the skills system

## Philosophy

- **Test-Driven Development** - Write tests first, always
- **Systematic over ad-hoc** - Process over guessing
- **Complexity reduction** - Simplicity as primary goal
- **Evidence over claims** - Verify before declaring success

Read more: [Superpowers for Claude Code](https://blog.fsck.com/2025/10/09/superpowers/)

## Contributing

Skills live directly in this repository. To contribute:

1. Fork the repository
2. Create a branch for your skill
3. Follow the `writing-skills` skill for creating and testing new skills
4. Submit a PR

See `skills/writing-skills/SKILL.md` for the complete guide.

## Updating

Skills update automatically when you update the plugin:

```bash
/plugin update superpowers
```

## License

MIT License - see LICENSE file for details

## Community

Superpowers is built by [Jesse Vincent](https://blog.fsck.com) and the rest of the folks at [Prime Radiant](https://primeradiant.com).

- **Discord**: [Join us](https://discord.gg/35wsABTejz) for community support, questions, and sharing what you're building with Superpowers
- **Issues**: https://github.com/obra/superpowers/issues
- **Release announcements**: [Sign up](https://primeradiant.com/superpowers/) to get notified about new versions
