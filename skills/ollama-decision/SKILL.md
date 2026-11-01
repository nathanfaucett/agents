---
name: ollama-decision
description: This skill facilitates quick, local, structured decision-making using Ollama System One (yes/no, choice, or score). It is used when an agent requires fast structured classification or judgment against text or JSON, avoiding free-form generation. It is triggered by phrases such as ollama decision, systemone, laya, local decision, classify, route, score urgency, or quick yes/no judgment.
---

# Ollama Decision

Use for **quick structured decisions** (yes/no, pick a label, or score) against text or JSON via local Ollama. Prefer this over free-form chat when you need a discrete answer + probability.

Requires Ollama ≥ 0.35 and a decision model. Default model: **laya**.

## CLI (preferred)

```bash
# Yes/no
ollama run laya --ask "Is a refund being requested?" < ticket.txt

# Choice
ollama run laya --ask "Which team?" \
  --choice billing="Payments and refunds" \
  --choice technical="Software problems" < ticket.txt

# Score (levels ordered low → high)
ollama run laya --ask "How urgent?" \
  --level "Routine" --level "Soon" --level "Immediate" < ticket.txt

# Multiple questions at once
ollama run laya \
  --ask "Is a refund being requested?" \
  --ask "Which team?" --choice billing="Payments" --choice technical="Bugs" \
  --ask "How urgent?" --level "Routine" --level "Soon" --level "Immediate" \
  < ticket.txt
```
