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

## Reference implementation (`:maturity :implemented`)

Full itonami Actor pattern (per ADR-2607011000 / CLAUDE.md's Actors
section, alongside `cloud-itonami-isco-6130`, `-8160`, `-2166`, `-2641`,
`-2651`, `-2652`, `-2654`, `-1219`, `-1223`, `-1330`, `-1341`, `-1349`,
`-1412`, `-1439`, `-2144`, `-2320`, `-2411`, `-2422`, `-2431`, `-2621`,
`-2634`, `-3122`, `-3123`, `-3141`, `-3255`, `-3339`, `-3512`, `-4120`,
`-4131`, `-4132`, `-4211` and `-4224`): a real
[`kotoba-lang/langgraph`](https://github.com/kotoba-lang/langgraph)
`StateGraph`, with the Advisor and Governor as distinct graph nodes and
human-in-the-loop interrupt/resume via checkpointing.

```text
:intake -> :advise -> :govern -> :decide -+-> :commit            (:ok? true)
                                           +-> :request-approval   (:escalate? true, interrupt-before)
                                           +-> :hold               (:hard? true)
```

- `src/client_information/store.cljk` — `Store` protocol +
  `MemStore`: registered clients, committed records, an append-only
  audit ledger.
- `src/client_information/advisor.cljk` — `Advisor` protocol;
  `mock-advisor` (deterministic, default) proposes an information
  operation from a request; `llm-advisor` wraps a
  `langchain.model/ChatModel` — either way the advisor only ever
  produces a `:propose`-effect proposal, never a committed record, and
  LLM parse failures always yield `confidence 0.0` (forces escalation,
  never fabricated confidence).
- `src/client_information/governor.cljk` —
  `ClientInformationGovernor/check`: a pure function, wired as its own
  `:govern` node. Hard invariants (unregistered client, a proposal
  whose `:effect` isn't `:propose`) always route to `:hold`. Escalation
  invariants (`:disclose-sensitive-information`,
  `:route-emergency-request`, or low advisor confidence) always route
  to `:request-approval` — an `interrupt-before` node that the graph
  checkpoints and only resumes on explicit human approval
  (`actor/approve!`), matching the README's robotics-premise statement
  that disclosing sensitive client information and emergency-request
  routing always require human sign-off.
- `src/client_information/actor.cljk` — `build-graph`, `run-request!`,
  `approve!`: the `langgraph.graph/state-graph` wiring itself.

```bash
clojure -M:test
```

This is what backs this repo's `:maturity :implemented` entry in
[`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation).

## License

AGPL-3.0-or-later.
