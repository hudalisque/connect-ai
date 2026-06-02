### ChatGPT UI-Specific Send/Stop Safety Rule

This rule applies specifically to ChatGPT UI behavior.

Observed issue:
- In saturated or delayed ChatGPT conversations, the prompt box and SEND/STOP button state may lag.
- The visible UI state may not immediately reflect the internal state.
- Pressing Enter or clicking the button while the UI is still in STOP state can interrupt generation or create partial prompt/state drift.

Operational rules:

1. If the button shows STOP, do NOT press Enter.
2. If the button shows STOP, do NOT click the button unless the intention is to stop generation.
3. Wait until the button returns to the SEND arrow state.
4. Only after the SEND arrow is visible, enter or send the next prompt.
5. Send exactly once.
6. After sending, do not press Enter or SEND again while waiting.
7. If the prompt box does not visibly show typed text, stop typing and wait.
8. Important instructions must be composed externally and pasted only when the prompt box is stable.

Principle:

STOP state means the channel is not ready for a new authoritative prompt.
SEND arrow state means the channel is ready.