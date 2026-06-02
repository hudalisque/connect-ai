# connect-ai Source Inspection Plan

## Goal

Inspect the source to answer one question:

> Is connect-ai a true multi-agent system or a persona-layer UI over one shared LLM runtime?

---

## Inspection Strategy

Do not read the whole 20k-line `extension.ts` sequentially.

Use targeted searches around:

1. Agent definitions
2. Runtime state
3. Memory ownership
4. Scheduling
5. Agent-to-agent communication
6. Tool permissions
7. Handoff / continuity

---

## Step 1 — Agent Definitions

Inspect:

```text
src/agents.ts
assets/prompts/
```

Questions:

- Are agents only metadata/persona?
- Are prompts materially different?
- Is there any reference to per-agent runtime identity?

Expected evidence type:

```text
id, name, role, persona, specialty
```

If this is all that exists, it suggests persona layer.

---

## Step 2 — Central Runtime

Inspect:

```text
SidebarChatProvider
_handleCorporatePrompt
```

Search:

```bash
rg "class SidebarChatProvider|_handleCorporatePrompt|sendPrompt|aiMessage|_chatHistory|_displayMessages" src/extension.ts
```

Questions:

- Is there one central chat provider?
- Is there one shared `_chatHistory`?
- Does every agent call the same prompt path?

---

## Step 3 — Agent State

Search:

```bash
rg "agentState|agentStates|perAgent|agentSession|agentRuntime|agentContext" src assets
```

Questions:

- Is state stored per agent?
- Or is state stored globally in the provider?

---

## Step 4 — Agent Memory

Search:

```bash
rg "agentMemory|agentMemories|memoryByAgent|agentHistory|perAgentHistory|_chatHistory" src assets
```

Questions:

- Do agents have their own durable memory?
- Does switching agent preserve independent context?
- Is Brain shared by all agents?

---

## Step 5 — Scheduling / Execution Model

Search:

```bash
rg "scheduler|taskQueue|jobQueue|agentLoop|runNext|dispatch|autoCycle|startAutoCycle" src assets
```

Questions:

- Is there a scheduler?
- Does it schedule independent agents?
- Or does it just trigger one prompt path periodically?

---

## Step 6 — Agent-to-Agent Communication

Search:

```bash
rg "sendToAgent|routeToAgent|broadcastToAgents|handoff|delegate|interAgent|agentMessage" src assets
```

Questions:

- Can one agent send a structured message to another?
- Are there handoff summaries?
- Or is corporate mode just a single prompt containing multiple persona names?

---

## Step 7 — Tool Permissions

Search:

```bash
rg "allowedTools|toolPermissions|capabilities|agentTools|listAgentTools|runTool" src assets
```

Questions:

- Are tools scoped per agent?
- Is there enforcement?
- Or are tools selected by label and run through the same execution path?

---

## Step 8 — Final Classification

Use this decision table:

| Finding | Classification |
|---|---|
| Per-agent memory + scheduler + handoff + message passing | True multi-agent |
| Persona prompts + shared chat + shared runtime | Persona layer |
| Agent-specific tools but shared memory/runtime | Hybrid |
| Insufficient evidence | Inconclusive |

---

## Important Warning

A visual office UI, sprites, badges, and agent names do **not** prove multi-agent architecture.

They prove a **representation layer**.

The task is to find whether there is an actual **runtime layer** beneath it.
