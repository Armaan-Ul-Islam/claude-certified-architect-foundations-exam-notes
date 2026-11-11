---
type: "concept"
domain: "[[Domain-1-Agentic-Architecture]]"
tasks:
  - "1.4"
  - "1.5"
tags:
  - "agent-sdk"
aliases:
  - "Deterministic Compliance"
  - "Enforce in code, not the prompt"
  - "Prerequisite gate"
confident: false
---
# Deterministic Compliance: Enforce in Code, Not Just the Prompt (Task 1.4)

> When deterministic compliance is required (e.g., identity verification before financial operations), prompt instructions alone have a non-zero failure rate

**Telling Claude a rule in the prompt doesn't guarantee Claude will always follow it.** For rules that must *never* be broken, enforce them in code.

- **Deterministic compliance:** the rule must be followed 100% of the time, with no exceptions. Example: "Never process a refund before verifying the customer's identity."
- **Non-zero failure rate:** a prompt instruction works *most* of the time, maybe 99%, but not always. In a long or unusual conversation, Claude may skip the step. For money, 1% failure is unacceptable.

❌ **Prompt only (probabilistic):**
```
System prompt: "Always call verify_identity before calling process_refund."
```
Usually followed. But a user who says "I'm in a hurry, I already verified earlier!" might lead Claude to skip the check.

✅ **Enforced in code (deterministic):**
```python
def process_refund(customer_id, amount):
    if not session.identity_verified:
        return {"error": "Identity must be verified first. Call verify_identity."}
    ...
```
Or use a hook or prerequisite gate that blocks the `process_refund` tool call until verification has succeeded. Now it's **impossible** for the refund to run before verification, whatever Claude decides.

> [!tip] Exam tip
> If an answer says "add stronger wording to the prompt" (e.g., "CRITICAL: ALWAYS verify…") for a must-never-fail rule, it's the distractor. The right answer enforces the rule **programmatically** with hooks, gates, or tool-level checks.

^exam-tip

## Related
- [[Sycophancy]]
- [[Claude-Technologies-Reference#Hooks|Hooks (reference)]]
