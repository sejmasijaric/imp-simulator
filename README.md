# imp-simulator

A planned modular Java platform for testing factory software without access to
physical machines. The project builds on the existing
[FischerTechnik simulator in edpo-project](https://github.com/sejmasijaric/edpo-project/tree/main/factory-simulator).

**Status:** design and scope proposal for team and supervisor review. This
repository has no implementation or runnable simulator yet.

## Initial scope

- Extract reusable components for machines, workpieces, state, timing, layout,
  configuration, and communication from the existing simulator.
- Migrate the FischerTechnik simulation while preserving its functionality and
  the HTTP/MQTT behaviour required by the existing application.
- Implement a second layout with two robotic manipulators and a conveyor belt
  using the same shared components.
- Support adjustable durations and a small set of explicit machine and
  communication fault scenarios.
- Evaluate compatibility, maintainability, extensibility, and component reuse.

The initial timing approach uses configurable delays in real elapsed time.
A transport configured for three seconds completes after approximately three
real seconds. A shared scheduling component keeps timing separate from machine
behaviour and lets tests control completion without waiting in real time.

Each factory supplies its own machines, connections, behaviour, and message
mapping. Simple settings live in configuration. New behaviour can use Java code
through documented extension points. Start with one simulator application and
reuse the current HTTP/MQTT integration code where suitable.

For now, the proposal leaves the separate sensor/controller IoT-lab example out
of the initial scope. An extension guide is part of the deliverables. A small
demonstration with an existing coding agent is optional.

Accelerated simulation time, detailed physics, 3D visualisation, distributed
simulation, cloud deployment, and a dedicated agent interface are outside the
initial scope. Confirm acceptance criteria with the supervisors at kickoff.

## Documents

- [Nikola's scope proposal](docs/scope-proposal.md): suggested decisions and rationale.
- [Design](docs/design.md): initial structure, timing, migration, and evaluation.
- [Kickoff presentation](presentation/kickoff.html): download and open in a browser.
  Use arrow keys to navigate, N for notes, O for the overview, and F for full screen.
