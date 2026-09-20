# Scope options: pros and cons

Notes for the kickoff discussion, written by Nikola. The purpose is to lay out
the trade-offs between the design currently in this repository and the scope
proposal, so we can pick a direction together and ask the supervisors the right
questions. Neither option is wrong. They optimise for different things.

## What the official project description asks for

As we read it, the description names three things:

1. Migration of the existing FischerTechnik simulator, preserving its functionality.
2. A second factory layout with two robotic manipulators and a conveyor belt.
3. A modular architecture, evaluated for maintainability, extensibility and reuse.

Worth confirming at kickoff how strictly points 1 and 2 are meant, because the
two options below differ mainly in how closely they follow them.

## Decision 1: starting point

### Option A: generic simulator engine, built fresh

| | |
| --- | --- |
| Pros | Clean abstractions, not shaped by the legacy code. Not limited to factories. Stronger architectural story for the report. |
| Cons | No baseline to compare against, so "does it still work" cannot be answered. The existing simulator and its tests are not reused. Larger amount of new code. Further from the project description. |

### Option B: refactor the existing simulator

| | |
| --- | --- |
| Pros | The old simulator is a measurable baseline, so regression tests give real evidence. Existing HTTP and MQTT integration code is reused. Matches the description. Incremental, so there is always something running. |
| Cons | Legacy structure can constrain the abstractions. Migration work is not very visible in the final result. Risk of inheriting design decisions we would not make again. |

## Decision 2: time model

### Option A: discrete event simulation with logical time

The engine keeps its own clock, never waits, and jumps from scheduled event to
scheduled event.

| | |
| --- | --- |
| Pros | Runs are fast, so long scenarios finish in milliseconds. Fully deterministic with a fixed seed, so ordered logs can be compared exactly. Standard method in simulation literature. Enables later throughput experiments. |
| Cons | Live commands from the application arrive in real time and have to be mapped into simulation time, which needs extra rules and synchronisation. More implementation work. Larger change to the existing simulator. |

### Option B: real elapsed time with configurable durations

A three second action completes after approximately three real seconds.

| | |
| --- | --- |
| Pros | Matches how the existing application interacts over HTTP and MQTT, so no time mapping is needed. Less work, smaller change during migration. Behaviour is easy to explain and demonstrate. |
| Cons | No accelerated runs. Timing varies slightly between runs, so tests need tolerances rather than exact log comparison. Weaker fit with classical simulation expectations. |

### Possible middle ground

Keep all waiting behind one scheduling component. Tests replace it with a helper
that completes pending actions immediately, so test runs stay fast and
deterministic even with real time in production. Logical time can then be added
later by replacing that one component instead of the whole system. Determinism
would be an explicit design goal from the start rather than an afterthought.

## Decision 3: second demonstrator

### Option A: IoT lab with sensors and a controller

| | |
| --- | --- |
| Pros | A genuinely different domain, so reuse of the core is much better evidence for modularity. Periodic measurement behaviour exercises different parts of the engine than process flow. |
| Cons | Second behaviour model, second message mapping and a second set of tests, so roughly double the work. Not the second example named in the project description. |

### Option B: second factory layout with two manipulators and a conveyor

| | |
| --- | --- |
| Pros | Named in the project description. Reuses machine and transport behaviour, so it is achievable in the available time. Allows a like for like comparison of reuse between two layouts. |
| Cons | Similar to the first layout, so it proves less about genericity. The claim of modularity rests more on documented reuse than on a contrasting example. |

### Mitigation if we pick Option B

Measure reuse explicitly rather than asserting it: which components both layouts
share, which core changes the second layout required, and what effort it takes to
change a duration, add another instance of an existing machine type or alter a
connection. Keep the IoT lab in the plan as a stretch goal after the two factory
layouts, so the stronger evidence is still possible if time allows.

## Where the two designs already agree

The disagreement is narrower than it looks. Both assume:

- A shared simulation core that owns time, events and state.
- Use case or factory modules that define components, behaviour and message meaning.
- Protocol adapters that keep HTTP and MQTT details out of the core.
- Simple values in configuration and behaviour in Java code.
- Explicit fault scenarios with delays, failures and recovery.
- Two demonstrations as the test of reuse.

So decisions 1 to 3 can be settled independently. Nothing in the architecture
needs to be rewritten either way.

## Suggested combination

1. Start from the existing simulator and preserve its behaviour as the baseline.
2. Use real elapsed time behind a single scheduling component, with determinism
   as a stated design goal and logical time as a documented extension path.
3. Build the second factory layout as the reuse demonstration and keep the IoT
   lab as a stretch goal rather than dropping it.

This follows the project description, keeps a measurable baseline, and leaves
the more ambitious version reachable instead of required.

## Questions for the supervisors

- Is the goal a test double that replaces the hardware for software testing, or a
  simulation intended for experiments over longer time spans? The time model
  follows from that answer.
- Which existing behaviours, messages and failure cases define compatibility for
  the migration?
- Are the two factory layouts sufficient as the initial demonstrations, with the
  IoT lab deferred?
- What evidence is required for the architectural evaluation, and is a small test
  client enough for the second layout?
- Does the second layout have a physical counterpart we can compare against, or is
  it a representative model only?
