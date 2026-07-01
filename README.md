# cloud-itonami-isco-4229

Open Occupation Blueprint for **ISCO-08 4229**: Client Information Workers Not Elsewhere Classified.

This repository designs a forkable OSS business for an independent client-information services provider: an information-desk robot performs visitor check-in and directory lookup under a governor-gated actor, so the practice keeps its own request and disclosure records instead of renting a closed information-services SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here an information-desk robot performs visitor check-in, directory lookup and physical document handoff under an actor that proposes
actions and an independent **Client Information Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
disclosing sensitive client information, or emergency-request routing) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
client request + information policy + disclosure scope
        |
        v
Client Information Advisor -> Client Information Governor -> respond/log, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `4229`). Required capabilities:

- :robotics
- :forms
- :identity
- :audit-ledger
- :bpmn

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
