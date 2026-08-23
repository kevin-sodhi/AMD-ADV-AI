# OpenClaw Agent Systems on AMD GPUs

Hands-on exploration of **local LLM inference and multi-agent systems on AMD GPUs** using OpenClaw, llama.cpp, ROCm, and Qwen3.6-35B.

The project progresses from serving a large open-weight model locally to building an autonomous multi-agent workflow capable of reviewing, fixing, and verifying code.

## Overview

The goal of this project was to explore how an agentic system can be built entirely around a locally served LLM rather than relying on external model APIs.

I served **Qwen3.6-35B** locally using llama.cpp on an AMD ROCm environment and connected it to OpenClaw through a local model endpoint.

From there, I explored increasingly complex agent workflows:

```text
Local Qwen3.6-35B
        ↓
    llama.cpp
        ↓
     OpenClaw
        ↓
Single Debugging Agent
        ↓
Reusable Debugging Skill
        ↓
Multi-Agent Review Panel
        ↓
Autonomous Fixer
```

The final workflow uses specialized agents to independently review unfamiliar code, aggregate their findings, apply fixes, and verify the resulting changes.

## Tech Stack

- **AMD ROCm**
- **AMD GPU**
- **Qwen3.6-35B-A3B**
- **llama.cpp**
- **OpenClaw**
- **Python**
- **pytest**
- **Jupyter**
- **Git / GitHub**

## 1. Local LLM Inference

The first step was serving an open-weight **Qwen3.6-35B-A3B** model locally using `llama.cpp`.

The model was loaded from a quantized GGUF checkpoint:

```text
Qwen3.6-35B-A3B-UD-Q6_K.gguf
```

with a multimodal projection model for vision support.

This created a local inference endpoint that OpenClaw could use without relying on an external model API.

The setup provided hands-on experience with:

- local LLM serving
- quantized model deployment
- AMD ROCm inference
- llama.cpp configuration
- model endpoint integration
- multimodal model configuration

## 2. Persistent OpenClaw Agent

I connected OpenClaw to the locally hosted model and explored its file-based agent architecture.

The agent maintains persistent information through workspace files that define its:

- identity
- behavior
- memory
- operating instructions
- reusable skills

Because these artifacts are stored as files, they can be inspected, edited, and version-controlled rather than existing only inside an opaque agent runtime.

## 3. Autonomous Repository Debugging

I then gave the agent an unfamiliar Python repository and had it:

1. Clone and inspect the repository
2. Understand the test suite
3. Run the application/tests
4. Identify a bug
5. Trace the failure to the relevant source
6. Apply a minimal fix
7. Re-run the tests
8. Verify the change

This demonstrated how a locally hosted model can interact with a development environment and execute a complete debugging workflow through tools.

## 4. Reusable Debugging Skill

Instead of keeping the debugging process as a one-off interaction, I packaged the workflow into a reusable OpenClaw skill.

The debugging protocol follows a structured process:

```text
Inspect Tests
      ↓
Run pytest
      ↓
Analyze Failure
      ↓
Inspect Source
      ↓
Apply Minimal Fix
      ↓
Re-run Tests
      ↓
Report Result
```

The skill can then be applied to another Python repository without redefining the debugging procedure from scratch.

This experiment helped me understand how persistent instructions and reusable tool workflows can make agent behavior more repeatable.

## 5. Multi-Agent Code Review

The next stage expanded the system from one general-purpose agent into multiple specialized agents.

The architecture uses a coordinator and two independent reviewers:

```text
                  ┌──────────────────┐
                  │   Coordinator    │
                  │      main        │
                  └────────┬─────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
       ┌─────────────────┐   ┌─────────────────┐
       │   Performance   │   │   Readability   │
       │    Reviewer     │   │    Reviewer     │
       └────────┬────────┘   └────────┬────────┘
                │                     │
                └──────────┬──────────┘
                           ▼
                    Merged Review
```

### Performance Reviewer

Focuses specifically on issues such as:

- repeated work inside loops
- unnecessary allocations
- redundant computation
- inefficient algorithmic patterns

### Readability Reviewer

Focuses on:

- unclear naming
- dead code
- overly long functions
- maintainability
- missing structure

Keeping each agent focused on one responsibility makes the workflow easier to reason about and reduces overlap between reviewers.

## 6. Parallel Agent Orchestration

The coordinator spawns the two reviewer agents in separate sessions.

Each reviewer analyzes the same source file independently and writes its findings to:

```text
/tmp/panel/performance.md
/tmp/panel/readability.md
```

The coordinator waits for both agents to finish and then merges their findings into a single report.

The files act as a simple shared communication mechanism between otherwise isolated agent sessions.

This architecture allowed me to explore:

- parallel agent execution
- specialized agent roles
- isolated agent contexts
- coordinator/sub-agent orchestration
- persistent intermediate outputs
- deterministic handoffs between agents

## 7. Closing the Loop with a Fixer Agent

The final stage extends the review system with a third specialized agent: the **Fixer**.

```text
              Coordinator
                   │
          ┌────────┴────────┐
          ▼                 ▼
    Performance        Readability
      Reviewer           Reviewer
          │                 │
          └────────┬────────┘
                   ▼
              Merged Review
                   │
                   ▼
                Fixer
                   │
                   ▼
              Run Tests
                   │
              ┌────┴────┐
              │         │
            FAIL       PASS
              │         │
              ▼         ▼
          Fix Again   Complete
```

The Fixer's responsibility is intentionally narrow:

- consume reviewer findings
- modify only the relevant code
- keep changes minimal
- run the tests
- verify that the identified issues have been resolved

This closes the loop from **analysis → action → verification**.

## What I Learned

The most interesting part of this project was seeing how a capable local model becomes only one component of a larger agent system.

The overall behavior depends on several layers:

```text
Local Model
     ↓
Inference Runtime
     ↓
Agent Runtime
     ↓
Tools + Persistent Memory
     ↓
Agent Specialization
     ↓
Orchestration
     ↓
Verification
```

The project also reinforced the value of giving agents narrow responsibilities and explicit handoff mechanisms rather than asking one agent to perform every task.

Building the reviewer/Fixer workflow gave me hands-on experience with the systems problems behind agentic applications: orchestration, context isolation, persistence, parallel execution, tool use, and verification.

## Repository Contents

`openclaw_AAI_radeon_llamacpp.ipynb`

The notebook contains the complete hands-on workflow covering:

- Qwen3.6-35B local inference
- llama.cpp serving on AMD ROCm
- OpenClaw configuration
- persistent agent memory
- autonomous repository debugging
- reusable OpenClaw skills
- parallel specialist reviewers
- coordinator/sub-agent orchestration
- autonomous code fixing and verification

## Background

This work was completed as part of hands-on experimentation with AMD's AI/ROCm ecosystem and OpenClaw.

The notebook preserves the implementation and exercises I used to understand how local LLM inference can be combined with persistent tools, specialized agents, and verification loops to build increasingly autonomous software-engineering workflows.

## Areas I'm Exploring Next

I'm interested in extending this work toward:

- evaluating multi-agent systems systematically
- measuring single-agent vs. multi-agent performance
- failure detection and recovery
- agent reliability and verification
- context and memory efficiency
- local LLM inference performance
- model selection and routing
- parallel agent execution
- evaluation of autonomous coding agents
