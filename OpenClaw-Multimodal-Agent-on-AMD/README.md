# OpenClaw Multimodal Agent on AMD

A personal deployment of [OpenClaw](https://github.com/openclaw) running entirely on my own AMD GPU cloud instance — a 35B-parameter multimodal model served locally with `llama.cpp`, an OpenClaw agent onboarded against it, and a multi-agent review-and-fix pipeline built on top: two specialist reviewers and a fixer that coordinate through file-based rendezvous instead of a framework like LangGraph.

Everything here runs on-GPU, on hardware I control — no third-party API keys, no rate limits, no external inference dependency.

## What's inside

| File | Description |
|---|---|
| `openclaw_AAI_radeon_llamacpp.ipynb` | End-to-end build notebook: model serving, agent onboarding, skill authoring, and the multi-agent review/fix pipeline |

## Environment

- AMD Cloud GPU instance with ROCm drivers installed
- `llama.cpp` built with ROCm/HIP support (`llama-server`)
- [OpenClaw](https://github.com/openclaw) installed
- Local model in GGUF format with vision support — `unsloth-Qwen3.6-35B-A3B-GGUF` (`Qwen3.6-35B-A3B-UD-Q6_K.gguf` + `mmproj-BF16.gguf`)
- Jupyter, for driving the build (notebook cells + terminal side by side)

## What it does

**Model serving.** `llama-server` loads Qwen3.6-35B-A3B onto the AMD GPU (`-ngl 999`, full offload) and exposes an OpenAI-compatible API on port 8080.

**Agent onboarding.** OpenClaw is configured non-interactively against that local endpoint, then switched into **lean mode** (`agents.defaults.experimental.localModelLean`) so the tool surface stays small enough for the local model to drive reliably. The gateway runs in the background; a TUI is how I actually talk to the agent.

**Agent memory.** OpenClaw persists agent state as plain markdown in a workspace folder — `SOUL.md`, `AGENTS.md`, `IDENTITY.md`, `USER.md`, `TOOLS.md`, `HEARTBEAT.md`, plus daily session logs under `memory/`. All of it is readable, editable, and version-controllable.

**Skill authoring.** Debugging workflows the agent runs once get packaged as a `SKILL.md` under `skills/<name>/` — YAML frontmatter plus step-by-step instructions, auto-injected into the system prompt. Once written, the same skill applies to any unseen Python project without re-explaining the steps.

**Multi-agent review pipeline.** A coordinator agent (`main`) spawns two specialist sub-agents — `performance` and `readability` — each reviewing the same file through its own narrow lens, in parallel. A third agent, the **Fixer**, takes their findings and applies them, closing the loop with a `verify.py` gate.

## Architecture

```
                 ┌─────────┐
   me ─────────▶ │  main   │  (coordinator)
                 └────┬────┘
           spawns     │     spawns
      ┌────────────┬──┴───┬────────────┐
      ▼            ▼      ▼            ▼
 ┌──────────┐ ┌───────────┐      ┌──────────┐
 │performance│ │readability│      │  fixer   │
 └────┬─────┘ └─────┬─────┘      └────┬─────┘
      │             │                 │
      ▼             ▼                 ▼
 /tmp/panel/   /tmp/panel/       applies fixes,
 performance.md readability.md   runs verify.py
```

Each sub-agent is a full OpenClaw agent with its own isolated workspace (`openclaw agents add <name> --workspace ...`). The coordinator only spawns agents and reads the rendezvous files — it never reviews code itself. The **file rendezvous** pattern (each reviewer writes findings to a fixed path, the coordinator polls for both files to exist) is what keeps this reliable on a local model: the coordinator never has to hold review text across turns, so it can't lose track and stall.

## OpenClaw mechanics used here

- **Local-model onboarding** — pointing OpenClaw at any OpenAI-compatible endpoint via `openclaw onboard --non-interactive`
- **Lean mode** — trimmed tool surface for reliable tool use on a local, non-frontier model
- **Sub-agents** — `agents.defaults.subagents.allowAgents` (spawn allow-list) plus the `sessions_spawn` / `sessions_yield` / `subagents` tools, no external orchestration framework
- **File rendezvous** — coordination primitive for parallel sub-agents that avoids relying on context to track state

## Running it

1. Open `openclaw_AAI_radeon_llamacpp.ipynb` in Jupyter on the GPU instance.
2. Start `llama-server` in a terminal, then run the setup cells to onboard OpenClaw against it.
3. Keep `llama-server` and the OpenClaw gateway running in the background for the whole session.
4. Add the reviewer and fixer agents, wire up the spawn allow-list and Review Panel Protocol, and restart the gateway to pick up config changes.
5. Drive everything from the OpenClaw TUI (`openclaw tui`).

## Stack

- **Hardware:** AMD GPU (ROCm), cloud-hosted
- **Inference:** `llama.cpp` (HIP/ROCm backend), OpenAI-compatible API
- **Model:** Qwen3.6-35B-A3B (Q6_K quantization, multimodal/vision-capable)
- **Agent runtime:** [OpenClaw](https://github.com/openclaw) (gateway + TUI + multi-agent spawning)
