# AI-DLC Steering Pack — general-purpose AI-DLC workflow

A **tool-agnostic**, general-purpose starter pack that applies the **AI-Driven
Development Lifecycle (AI-DLC)** decision-gated workflow to any software effort —
greenfield builds, modernization, migration, or feature work. It ships the
**workflow itself** (Requirements → Design → Tasks, with an optional brownfield
reverse-engineering pre-phase) and **no use-case-specific skills**, so it stays
neutral and reusable. Add your own skills to specialize it (see
[Extending this pack](#extending-this-pack)).

The pack is authored once as tool-neutral source and works with **Kiro, Claude
Code, GitHub Copilot, and Cursor**. Whichever agent you use follows the same
structured, decision-gated workflow — no manual setup needed.

## Use case

Any project where you want AI to plan before it builds: it asks for your
decisions first (writing a `_decisions-*.md` before each spec document), then
generates requirements, design, and an executable task plan — gate by gate, with
your approval at each step. Works **greenfield** (new system) and **brownfield**
(the agent reverse-engineers the existing codebase first in Phase 0).

Typical kickoffs:
- *"Start the AI-DLC workflow to build \<your system\>."*
- *"We're modernizing this legacy app — reverse-engineer it, then plan the work."*
- *"Add \<feature\> to our existing service using the decision-gated workflow."*

## Getting started

Pick **one** of the two ways to add this pack to your project.

### Option A — copy a pre-built folder (no tooling)

Pre-generated, tool-correct configs live under [`scaffolded-packs/`](scaffolded-packs/). Copy the folder for your tool into your project root:

| Your tool | Copy from | Into your project |
|---|---|---|
| **Kiro** | `scaffolded-packs/kiro/` | `.kiro/` |
| **Claude Code** | `scaffolded-packs/claude-code/` | `CLAUDE.md`, `.claude/`, `.mcp.json` |
| **GitHub Copilot** | `scaffolded-packs/copilot/` | `.github/`, `.vscode/mcp.json` |
| **Cursor** | `scaffolded-packs/cursor/` | `.cursor/` |

### Option B — generate it (installer)

Run the `ramp-pack` installer from the repo root; it reads the neutral source and writes the correct layout into your target project:

```bash
node installer/bin/ramp-pack.js init ai-dlc-steering-pack --tool <kiro|claude-code|copilot|cursor> --target /path/to/your/project
```

Add `--dry-run` to preview, `--force` to overwrite existing files. Option B always works even if `scaffolded-packs/` is missing or out of date — the neutral source is the single source of truth.

### Then

1. **(Brownfield only)** Put the existing app in the workspace (or an `existing-codebase/` subfolder). The workflow detects existing code and runs **Phase 0 Reverse Engineering** first.

2. Open the project in your tool and start a conversation. Try:
   - *"Start the AI-DLC workflow for this project."*
   - On Claude Code / Copilot you can also run the **`/aidlc`** command to kick off the workflow.

The workflow guides the agent to ask for your decisions first (writing a `_decisions-*.md` before each spec document), then generate requirements, design, and tasks — each behind an approval gate.

## What's in this pack

```
ai-dlc-steering-pack/
├── pack.yaml                 # Manifest: instruction roles, MCP servers, /aidlc command
├── instructions/             # Tool-neutral steering (source of truth)
│   ├── aidlc-workflow.md         # Decision-gated Requirements → Design → Tasks (primary)
│   ├── skill-activation.md       # Knowledge/MCP activation + how to extend with skills (companion, always)
│   └── reverse-engineering.md    # Phase 0 playbook (companion, brownfield-only)
├── skills/                   # Intentionally empty — add your own Agent Skills here
└── scaffolded-packs/         # Pre-generated per-tool configs (Option A above)
    ├── kiro/         # .kiro/{steering,settings}
    ├── claude-code/  # CLAUDE.md, .claude/{rules,commands}, .mcp.json
    ├── copilot/      # .github/{copilot-instructions.md,instructions,prompts}, .vscode/mcp.json
    └── cursor/       # .cursor/{rules}, .cursor/mcp.json
```

> `instructions/`, `skills/`, and `pack.yaml` are the **neutral source** you edit. `scaffolded-packs/` is **generated** from them by the installer — regenerate it after editing the source; don't hand-edit the scaffolded output.

### The workflow

The `aidlc-workflow.md` instruction is the heart of the pack. It enforces:

- **Decision gates** — before writing any `requirements.md`, `design.md`, or `tasks.md`, the agent first produces a matching `_decisions-*.md` with options and waits for your input.
- **Phased flow** — optional **Phase 0** reverse engineering (brownfield) → **Phase 1** Requirements → **Phase 2** Design → **Phase 3** Tasks, each behind an explicit approval gate.
- **Zoomable diagrams** — any generated diagram (architecture, sequence, ER, or the tasks parallelism diagram) is saved as a standalone, openable file with a zoom link, and split when too dense, so it never renders as an unreadable thumbnail.
- **Parallel task planning** — `tasks.md` groups tasks into independent waves that can run concurrently, with a **parallelism flow diagram** on top of the list showing which groups run in parallel and which must wait.
- **State + audit** — a state file and append-only audit log make sessions resumable and every decision traceable.

### How each instruction maps per tool

The neutral instructions declare a **role** (`primary` / `companion`) and a **load** rule (`always` / `auto`); the installer renders each into the target tool's native mechanism:

| Neutral role | Kiro | Claude Code | Copilot | Cursor |
|---|---|---|---|---|
| `aidlc-workflow` (primary) | `.kiro/steering/*` `inclusion: always` | `CLAUDE.md` | `.github/copilot-instructions.md` | `.cursor/rules/*.mdc` `alwaysApply: true` |
| `skill-activation` (always) | `inclusion: always` | `.claude/rules/*` | `.github/instructions/*` `applyTo: '**'` | `.mdc` `alwaysApply: false` |
| `reverse-engineering` (auto) | `inclusion: auto` | `.claude/rules/*` | `.github/instructions/*` (conditional) | `.mdc` `alwaysApply: false` |
| `/aidlc` command | — | `.claude/commands/aidlc.md` | `.github/prompts/aidlc.prompt.md` | — |

### Extending this pack

This pack intentionally ships an **empty `skills/` folder** — it's the neutral
workflow only. To give the agent deep, domain-specific expertise (a language, a
framework, a cloud service, a testing approach), drop an Agent Skill (a folder
containing a `SKILL.md`, optionally with a `references/` library, following the
[Agent Skills open standard](https://agentskills.io/)) into `skills/`, set
`skills: all` (or list them) in `pack.yaml`, and regenerate the scaffolded packs.
Skills load automatically when their trigger keywords appear in the conversation.

### MCP servers

Declared once in `pack.yaml`; the installer writes it to each tool's MCP config (`.kiro/settings/mcp.json`, `.mcp.json`, `.vscode/mcp.json`, `.cursor/mcp.json`).

| MCP Server | When the agent uses it |
|---|---|
| **AWS Knowledge** (`aws-knowledge-mcp-server`) | Validating AWS specifics — service capabilities, quotas/limits, regional availability, and pricing shape — before putting them in a decision file or design. |

## Prerequisites

- One of: [Kiro](https://kiro.dev), [Claude Code](https://claude.com/claude-code), GitHub Copilot, or Cursor — installed and signed in.
- **Option B (installer) only:** Node.js 18+ (to run `ramp-pack`).

## License

Sample code, licensed under MIT-0. See the repository [`LICENSE`](../LICENSE). AI-DLC steering is adapted from [awslabs/aidlc-workflows](https://github.com/awslabs/aidlc-workflows) (MIT-0).
