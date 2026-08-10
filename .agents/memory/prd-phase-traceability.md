---
name: PRD-to-phase traceability
description: Durable rule for keeping JOBEN requirements mapped to delivery phases and release evidence.
---

Every JOBEN capability must be traceable from the canonical PRD to a phase owner,
work package, dependency, acceptance evidence, and release gate. Cross-cutting
requirements are re-verified in every phase that changes the affected data,
provider, UI, capability, or operational behavior; they are not considered
covered by a one-time mention.

**Why:** The product's safety contract depends on truth, tenant isolation,
provider verification, recovery, and proof records, so a feature list alone can
hide missing delivery obligations.

**How to apply:** Before starting a phase or changing a capability, update the
traceability register, module ownership matrix, exit proof, and capability
status rules together. Never promote a capability without its phase proof packet.