# Scope proposal for team review

**Proposed by Nikola for discussion with Sejma and the supervisors.**

This proposal builds on the existing design. It keeps a shared simulation core,
separate factory behaviour, and communication adapters, while choosing a small
initial implementation. The updated README, design, and kickoff slides reflect
this proposal. Supervisor acceptance is still to be confirmed.

## Proposed outcome

Develop a reusable Java foundation by refactoring the current FischerTechnik
simulator. Demonstrate it with the existing factory and a second layout containing
two robotic manipulators and a conveyor belt. Preserve existing functionality and
evaluate maintainability, extensibility, and actual component reuse.

## Suggested decisions

| Topic | Initial proposal | Reason |
| --- | --- | --- |
| Starting point | Reuse and refactor the existing simulator incrementally. | Existing behaviour and tests provide a baseline. |
| Factory examples | FischerTechnik plus two manipulators and a conveyor. | These are the reference implementations in the original project description. |
| Separate IoT lab | Leave the sensor/controller example out for now. | Keep the two required factory models as the initial demonstrations. |
| Timing | Use real elapsed time and configurable durations from the start. | The current external application can interact using ordinary elapsed time. |
| Communication | Preserve and separate the required HTTP and MQTT behaviour. | Reuse existing integration code and keep compatibility visible. |
| Structure | One simulator application with shared components and factory modules. | Keep deployment and integration simple. |
| Configuration | Configure simple values and supported connections. Implement new behaviour in Java. | Provide an explicit extension path without designing a general behaviour language. |
| Faults | Fixed delays and a few explicitly triggered failures. | Make expected outcomes understandable and testable. |
| Evaluation | Baseline tests, two demonstrations, and documented extension tasks. | Collect concrete evidence for the architectural goals. |

## Timing in plain language

If a robot transport is configured to take three seconds:

1. The simulator receives the transport command.
2. The robot becomes busy.
3. After approximately three real seconds, the action completes.
4. The simulator updates the workpiece location and sends the expected messages.

Changing the duration to five seconds changes that wait. One shared scheduling
component handles these delays. Tests can control completion directly through a
test helper. Accelerated simulation time is deferred.

The initial timing choice is real-time pacing. Confirm required timing tolerances
and existing command/response behaviour at kickoff.

## Two models, one foundation

The first model retains the existing FischerTechnik behaviour. The second models
one route: the first manipulator puts a workpiece on a conveyor and the second
manipulator collects it at the other end. Both use the same general state and
timing functions and reuse machine behaviour where it fits.

For the separate IoT-lab model, the discussion proposal is: **shall we leave it
out of the first version and reconsider it only after the two factory models?**
This defers an additional example. The original factory-simulation objectives
remain in scope.

## Scenarios and evidence

Begin with normal operation, a longer action duration, a defined machine failure,
and temporary suppression of selected telemetry. Specify when each fault starts,
what it changes, and how to clear or reset it. Add tests alongside each change.

Show that existing flows still work and that the second factory uses shared
components. Document extension tasks such as changing a duration or adding
another instance of an existing machine type. Explain any required core changes
and the limits of reuse. Use available hardware evidence or documentation for
selected comparisons, and distinguish those from agreement with the old
simulator alone.

## Optional agent idea

Write an extension guide with an example and tests. If the main deliverables are
complete, use an existing coding agent to attempt one small extension from that
guide. Record what worked and what needed correction. Packaging the guide as a
skill can support this experiment. A dedicated agent interface is deferred.

## Presentation work plan

Keep the supplied milestone dates and revise the implementation work packages:
baseline and design, extraction of shared components, FischerTechnik migration,
second factory layout, then evaluation and demonstration. Write documentation
throughout. Confirm the supplied dates against the official course schedule.

## Decisions to confirm together

- Are the two factory layouts the agreed initial demonstrations?
- Does the proposed timing approach satisfy the required behaviour?
- Which existing features, messages, and failure cases must the migration preserve?
- Is a simple test client sufficient to demonstrate the second layout?
- Are the evaluation tasks sufficient, with the IoT-lab example deferred and the
  coding-agent demonstration optional?
