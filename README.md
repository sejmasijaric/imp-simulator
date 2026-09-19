# imp-simulator

A planned Java simulator for testing IoT systems without access to real hardware.
It will model device behavior, communication, delays, and failures.

The same engine should run different systems. Each use case supplies its own
components, connections, behavior, and test scenarios. The engine handles time,
events, and state updates; adapters connect it to external software.

For example, a factory model could simulate machines and conveyors, while a lab
model could simulate sensors sending readings to a controller. Both should use
the same engine without changes to its code.

**Status:** design only. There is no implementation or runnable demo yet.

The planned simulator needs to:

- Define components, their initial state, and their connections.
- Process scheduled events and update component state.
- Run scenarios with delays, failures, and recovery.
- Accept external commands and publish telemetry through adapters.
- Produce repeatable runs with fixed inputs.
- Support two distinct use cases to check that the engine is reusable.

This phase covers the architecture. High-fidelity physics, 3D visualization,
distributed simulation, cloud deployment, and production-scale performance are
outside its scope.

See [the design](docs/design.md) for the execution model, decisions, and open questions.
