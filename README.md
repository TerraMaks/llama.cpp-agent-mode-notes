# llama.cpp Agent Mode Notes

Practical observations from running agent-oriented workflows with llama-server through an OpenAI-compatible API.

---

# Overview

During long-running agent workflows, a significant stability difference was observed when model-side reasoning was enabled inside Qwen3.6 while an external agent was already responsible for planning and orchestration.

The issue appeared in the following stack:

```text
Visual Studio Code
        +
GitHub Copilot Agent
        +
OpenAI-Compatible API
        +
llama-server
        +
Qwen3.6-35B-A3B
```

After investigation, the single most impactful configuration change was:

```bash
--reasoning off
--reasoning-budget 0
```

This repository documents the observation and provides reproducible configuration details.

---

# Environment

## Server

* llama.cpp
* llama-server
* OpenAI-compatible API

## Client

* Visual Studio Code
* GitHub Copilot Agent Mode

## Models

### Qwen3.6-35B-A3B

* GGUF
* MXFP4_MOE
* Vision enabled through mmproj-F32

### GPT-OSS 120B

* GGUF
* MXFP4

---

# Official llama.cpp Behavior

Recent llama.cpp releases provide native controls for reasoning behavior:

```bash
--reasoning on|off|auto
--reasoning-budget N
```

The configuration used in this repository relies exclusively on official llama.cpp functionality:

```bash
--reasoning off
--reasoning-budget 0
```

No custom patches, template modifications, or source-code changes were required.

---

# Observed Problem

With Qwen3.6 reasoning enabled, agent workflows exhibited several undesirable behaviors:

* reduced Agent Mode stability
* degraded tool-calling reliability
* visible reasoning artifacts
* duplicated planning behavior
* increased response latency
* less predictable execution during long sessions

The effect became increasingly visible as workflows became more tool-driven and multi-step.

---

# Architectural Hypothesis

The external agent already performs planning and orchestration.

When Qwen reasoning is enabled, two independent planning systems may operate simultaneously:

```text
GitHub Copilot Agent
        +
Qwen Reasoning
```

This may create competing planning layers and negatively affect agent execution.

A more stable architecture was observed when the model primarily acted as an execution engine:

```text
GitHub Copilot Agent
        +
Qwen Execution
```

This is an observed hypothesis based on production testing.

The repository does not claim a proven causal relationship.

---

# Working Configuration

The following configuration produced consistently better results with Qwen3.6:

```bash
--reasoning off
--reasoning-budget 0
```

Observed improvements:

* cleaner outputs
* more predictable execution
* improved tool-calling behavior
* reduced latency
* improved long-session stability
* elimination of visible reasoning artifacts

The improvement was reproducible across multiple long-running Agent Mode sessions.

---

# Tool Calling Observations

The largest practical improvement was observed in tool-calling workflows.

With reasoning enabled:

* additional thinking phases appeared before tool usage
* tool invocation became less predictable
* response latency increased
* planning responsibilities became less clearly separated

With reasoning disabled:

* tool invocation became more direct
* responses became shorter and cleaner
* agent execution became more predictable

These observations are consistent with reports from other users in the llama.cpp and local-LLM communities.

---

# Evidence

Example llama-server log:

```text
reasoning-budget: activated, budget=0 tokens
reasoning-budget: budget=0, forcing immediately
reasoning-budget: forced sequence complete, done
```

This confirms that llama.cpp terminated the reasoning phase immediately before normal generation began.

The log confirms correct operation of the reasoning controls.

It does not by itself prove the architectural hypothesis described above.

---

# Results

## Qwen3.6-35B-A3B

Configuration:

```bash
--reasoning off
--reasoning-budget 0
```

Observed:

* stable agent execution
* improved tool-calling reliability
* cleaner responses
* no visible reasoning artifacts
* better behavior during long-running sessions

This configuration became the production configuration for agent-oriented workflows.

---

## GPT-OSS 120B

GPT-OSS 120B was used differently.

Configuration:

```bash
--reasoning auto
--reasoning-format auto
--reasoning-budget 96
```

Observed role:

* expert analysis
* architectural review
* final validation
* deep reasoning tasks

Unlike Qwen3.6, GPT-OSS 120B was intentionally operated with reasoning enabled and was not used as the primary agent-execution model.

Therefore, the findings documented here should not be interpreted as evidence that reasoning should be disabled for GPT-OSS 120B.

---

# Recommendation

For Qwen3.6 running inside agent-oriented workflows, test the following configuration before changing prompts, context size, sampling parameters, or model selection:

```bash
--reasoning off
--reasoning-budget 0
```

Especially when using:

* GitHub Copilot Agent
* VS Code Agent Mode
* OpenAI-compatible APIs
* Tool Calling
* Long-running autonomous workflows

This recommendation should be treated as a configuration to test first, not as a universal best practice.

---

# Scope

This observation does not imply that reasoning should always be disabled.

The finding is limited to environments where:

* an external agent already performs planning
* the model primarily executes tasks
* tool calling is heavily used
* Qwen3.6 is used as the primary working model

Reasoning may still provide significant benefits in:

* standalone chat scenarios
* expert analysis
* architectural review
* deep research workflows

GPT-OSS 120B is one example where reasoning remained enabled in production use.

---

# Reproducibility

Hardware:

* NVIDIA GeForce RTX 5070

Software:

* llama.cpp
* llama-server
* OpenAI-compatible API
* Visual Studio Code
* GitHub Copilot Agent Mode

Models tested:

* Qwen3.6-35B-A3B MXFP4_MOE
* GPT-OSS 120B MXFP4

Additional validation on other hardware, models, and agent frameworks is encouraged.

---

# Key Finding

For Qwen3.6 operating inside VS Code Agent Mode, the most impactful configuration change tested was:

```bash
--reasoning off
--reasoning-budget 0
```

The change improved:

* stability
* predictability
* tool-calling behavior
* response quality

while reducing:

* latency
* reasoning artifacts
* duplicated planning behavior

The finding applies specifically to agent-oriented execution workflows where an external orchestration layer already performs planning.

The observation should not be generalized to:

* all models
* all llama.cpp deployments
* all reasoning workloads
* standalone chat usage
