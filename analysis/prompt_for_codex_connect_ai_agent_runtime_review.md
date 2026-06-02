# Prompt for Local Code Agent — connect-ai Agent Runtime Review

You are an expert software architect reviewing the `connect-ai` VS Code extension.

Do not perform a broad review.  
Do not focus on security unless it directly affects agent architecture.  
Do not modify code.

## Mission

Determine whether `connect-ai` implements a true multi-agent runtime or mostly a persona layer over one shared LLM/chat runtime.

The user's core question:

> Are there really multiple agents, or is it one actor changing name tags and clothes?

## Context

Previous review found:

- `extension.ts` is around 20,391 lines.
- `agents.ts` defines 10 agents with persona-like metadata.
- `SidebarChatProvider` appears to be the main chat/runtime class.
- Some security issues were found, but those are not this task.
- The current hypothesis is that the project may be closer to **Persona Theater** than a true multi-agent runtime.

Your job is to verify this from source code.

## What to Inspect

Prioritize:

```text
src/agents.ts
src/extension.ts
assets/prompts/
assets/webview/
```

Especially inspect:

```text
SidebarChatProvider
_handleCorporatePrompt
agent dispatch logic
runTool handler
auto-cycle logic
_chatHistory
_displayMessages
_brainEnabled
_recentFileActions
listAgentTools
```

## Search Commands

Use these first:

```bash
rg "agentState|agentStates|agentMemory|agentHistory|perAgent|memoryByAgent" src assets
rg "dispatchAgent|routeToAgent|sendToAgent|broadcastToAgents|handoff|delegate|interAgent" src assets
rg "taskQueue|jobQueue|scheduler|agentLoop|runNext|queue" src assets
rg "_chatHistory|_displayMessages|_handleCorporatePrompt|SidebarChatProvider" src/extension.ts
rg "allowedTools|toolPermissions|capabilities|agentTools|listAgentTools" src assets
```

## Evaluation Criteria

Classify the system as one of:

1. **True Multi-Agent Runtime**
   - independent state per agent
   - independent memory per agent
   - scheduler or task queue
   - agent-to-agent message passing
   - handoff protocol
   - capability boundaries

2. **Persona Layer over Shared Runtime**
   - one shared chat history
   - one shared LLM path
   - agent identity mainly changes prompt/persona/style
   - no real independent memory or scheduling

3. **Hybrid**
   - some per-agent tool/persona separation
   - but no full runtime separation

4. **Inconclusive**
   - insufficient code evidence

## Required Report

Return markdown:

```markdown
# connect-ai Agent Runtime Review

## Executive Finding

State the conclusion directly.

## Evidence Table

| Area | Evidence | Interpretation |
|---|---|---|
| Agent definition | ... | ... |
| State | ... | ... |
| Memory | ... | ... |
| Scheduling | ... | ... |
| Message passing | ... | ... |
| Tool permissions | ... | ... |
| Handoff | ... | ... |

## Architecture Diagram

Text diagram showing the actual observed architecture.

## Final Judgment

Answer:

> Multiple agents, or one runtime changing labels?

## Missing Components for True Multi-Agent

List what would need to exist.

## Confidence Level

High / Medium / Low, with reason.
```

## Rules

- Use exact code snippets and line numbers.
- Do not modify code.
- Do not broaden scope.
- Do not re-audit all security issues.
- Separate facts, inferences, and unknowns.
