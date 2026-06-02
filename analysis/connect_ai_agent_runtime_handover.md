# connect-ai Agent Runtime Analysis — Handover

## Important

Do NOT spend time on security findings already documented.

The security review is complete.

Focus only on:

- Agent Runtime
- Agent Memory
- Agent State
- Scheduling
- Handoff
- Agent-to-Agent Communication

The goal is to determine whether connect-ai is:

1. True Multi-Agent Runtime
2. Persona Layer
3. Hybrid

## Purpose

This handover is for a new local code-review agent working on the `connect-ai` VS Code extension source.

The previous report focused mainly on security and architecture risk.  
That report was useful for threat modeling, but it did **not** answer the user's primary question:

> Is connect-ai a real multi-agent system, or mostly one LLM/chat runtime wearing different agent personas?

The next analysis must focus on the **agent system itself**.

---

## Current Working Hypothesis

The current hypothesis is:

> connect-ai appears closer to a **Persona Layer / Agent Theater** than a true multi-agent runtime.

Meaning:

```text
One central chat/runtime
    +
multiple agent cards/personas
    +
shared history/context
```

rather than:

```text
independent agents
    +
separate state
    +
separate memory
    +
message passing
    +
scheduler/queue
    +
handoff protocol
```

This is a hypothesis, not yet final.  
Your task is to verify or falsify it from code evidence.

---

## Prior Confirmed Findings

From the previous security/architecture review:

### Confirmed

- `extension.ts` is about 20,391 lines.
- `SidebarChatProvider` appears to be the main chat/runtime class.
- `agents.ts` defines 10 agents with fields such as:
  - `id`
  - `name`
  - `role`
  - `emoji`
  - `color`
  - `specialty`
  - `tagline`
  - `persona`
- LLM output can be parsed for `<run_command>` blocks.
- Parsed commands can reach `runCommandCaptured(...)`.
- There is a `runTool` webview message path.
- There are HTTP endpoints such as `/api/brain-inject` and `/api/skill-inject`.

### Important Caution

Do not re-open the previous security audit unless needed.

The next task is **not** to find more vulnerabilities.

The next task is to determine the real agent architecture.

---

## Core Question

Answer this:

```text
Does connect-ai implement true multi-agent collaboration,
or does it implement one shared LLM runtime with multiple personas?
```

Use code evidence only.

---

## What Counts as a True Agent?

A real agent should have at least some of the following:

### 1. Independent State

Look for:

```text
agentState
agentStates
perAgentState
agentRuntime
agentSession
```

Evidence of per-agent state separate from global chat state.

---

### 2. Independent Memory

Look for:

```text
agentMemory
agentMemories
memoryByAgent
agentHistory
perAgentHistory
```

Check whether each agent has its own durable memory, or whether all agents share `_chatHistory`.

---

### 3. Task Queue / Scheduler

Look for:

```text
queue
taskQueue
jobQueue
scheduler
dispatch
runNext
agentLoop
```

Determine whether agents are scheduled like independent workers, or whether a single synchronous prompt path is reused.

---

### 4. Message Passing

Look for:

```text
sendToAgent
routeToAgent
broadcastToAgents
handoff
delegate
agentMessage
interAgent
```

A true multi-agent system should have agent-to-agent communication beyond simple prompt concatenation.

---

### 5. Capability Boundaries

Look for:

```text
allowedTools
agentTools
toolPermissions
capabilities
permission
```

Determine whether agents have different tool permissions, or whether all agents can access the same runtime capabilities.

---

### 6. Handoff / Continuity

Look for:

```text
handoff
summary
contextTransfer
handover
agentBrief
continuity
```

Determine whether the system preserves work state when one agent hands off to another.

---

## Key Files to Inspect

Start with:

```text
src/agents.ts
src/extension.ts
assets/prompts/
assets/webview/
```

Within `extension.ts`, prioritize:

```text
SidebarChatProvider
_handleCorporatePrompt
runTool handler
auto-cycle logic
agent dispatch logic
_chatHistory
_displayMessages
_brainEnabled
_recentFileActions
```

---

## Suggested Search Terms

Use local search / grep / ripgrep:

```bash
rg "agentState|agentStates|agentMemory|agentHistory|perAgent|memoryByAgent" src assets
rg "dispatchAgent|routeToAgent|sendToAgent|broadcastToAgents|handoff|delegate|interAgent" src assets
rg "taskQueue|jobQueue|scheduler|agentLoop|runNext|queue" src assets
rg "_chatHistory|_displayMessages|_handleCorporatePrompt|SidebarChatProvider" src/extension.ts
rg "allowedTools|toolPermissions|capabilities|agentTools|listAgentTools" src assets
```

---

## Required Output Format

Produce a concise evidence-based report with this structure:

```markdown
# connect-ai Agent Runtime Review

## 1. Executive Finding

One of:

- True multi-agent runtime
- Persona-layer over one shared runtime
- Hybrid
- Inconclusive

## 2. Evidence Table

| Area | Evidence | Interpretation |
|---|---|---|
| Agent definition | ... | ... |
| State | ... | ... |
| Memory | ... | ... |
| Scheduling | ... | ... |
| Agent-to-agent communication | ... | ... |
| Tool permissions | ... | ... |
| Handoff | ... | ... |

## 3. Architecture Diagram

Use text diagrams.

## 4. Final Judgment

Answer directly:

> Are there really multiple agents, or is this one runtime changing labels?

## 5. What Would Be Required for a True Multi-Agent Runtime?

List missing components.
```

---

## Important Rules

1. Do not modify code.
2. Do not search for new security issues.
3. Do not speculate beyond code evidence.
4. Separate:
   - Confirmed facts
   - Inferences
   - Unknowns
5. If evidence is missing, say `Not found in inspected code`.
6. Keep the focus on **agent architecture**, not general code quality.

---

## Peter's Working Lens

Peter's core question:

> Are there really several agents, or is one actor changing name tags and clothes?

The reviewer should inspect whether there is a lower runtime layer beneath the visual/persona layer.

In Peter's terms:

```text
Surface layer:
    agent names, icons, sprites, personas

Runtime layer:
    scheduler, state, memory, queue, handoff, capability control
```

The task is to determine whether the runtime layer exists.
