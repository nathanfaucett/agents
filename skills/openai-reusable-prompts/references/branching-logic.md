# Branching Logic

- If output must follow strict schema:
  Use explicit field-level formatting rules and schema-like examples.

- If behavior differs by customer or locale:
  Keep global rules stable and isolate locale-specific rules in dedicated context blocks.

- If the prompt is long and complex:
  Split sections clearly and remove non-essential context.

- If task is reasoning-heavy and under-specified:
  Start with goal + constraints, then tighten with examples after observing failures.

- If task is deterministic and format-sensitive:
  Use strict format instructions and highly specific examples.
