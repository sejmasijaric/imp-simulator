# Towards a Modular IoT Factory Simulation Platform: Requirements and Evaluation

**Integrative Master's Project, University of St. Gallen**  
**Team:** Sejma Sijaric, Nikola Milosavljevic  
**Supervisors:** Prof. Barbara Weber, Dr. Amine Abbad-Andaloussi  
**Status:** Draft v4.1 for discussion on 30 September 2026  
**Updated:** 22 September 2026

The requirements below describe our proposed core. Possible extensions and open points are collected in Section 9.

## 1. Motivation

Developers of factory process software need to check whether machine commands, transfers and responses work together. For example, a robot should only collect a workpiece after the preceding machine has made it available. Developers also need to examine what happens when a movement fails or a response is delayed.

Tests against physical machines depend on access to the lab and its equipment. A simulator provides a software environment for developing these interactions and repeating selected scenarios. The existing FischerTechnik simulator already supports development away from the lab for its original application.

The remaining problem is adaptation: another layout requires changes to factory-specific locations, machine mappings and behaviour spread across the implementation. Some components are already reusable, but reuse across different layouts has not been demonstrated. Developers need a clearer way to assemble another factory model and check its interactions without duplicating the simulation foundation.

## 2. Goal and contribution

**Goal:** Turn the existing simulator into a reusable foundation so that developers can set up another factory layout, test the interactions between its machines and repeat selected failure cases without access to the physical lab.

**What is new compared to the existing simulator:**

- A shared core that runs a second factory layout next to FischerTechnik
- Central timing that makes machine actions controllable in tests
- Repeatable fault scenarios, including suppressed machine messages
- An evaluation of how well the simulator can be reused and extended

The existing FischerTechnik behaviour is preserved and serves as the baseline. Possible extensions, such as faster-than-real-time execution, are listed in Section 9.

**Users:** Developers and students testing factory process software.  
**Fidelity:** Commands, machine states, workpiece transfers and observable messages.

## 3. Requirements overview

| ID | Requirement | Addresses | Evaluation |
| --- | --- | --- | --- |
| FR-01 | Preserve the existing FischerTechnik interfaces and behaviour | Compatibility, G2 | EVAL-01 |
| FR-02 | Run two layouts using the same core | G1, G5 | EVAL-02 |
| FR-03 | Define and observe valid machine interactions | G2 | EVAL-03 |
| FR-04 | Centralise timing and support controlled tests | G3 | EVAL-04 |
| FR-05 | Support selected, repeatable fault scenarios | G4 | EVAL-05 |
| NFR-01 | Keep supported layout changes outside the core | G1, G2, G5 | EVAL-06 |
| NFR-02 | Provide repeatable runs and inspectable results | G4, G5 | EVAL-07 |

## 4. Baseline and gaps

### 4.1 Existing / Extend / New

The baseline is `edpo-project` at commit `868d848`. The classifications refer to the inspected source; runtime results will be recorded during baseline validation. Evidence links are in the appendix.

| Capability | Status | Evidence | Treatment |
| --- | --- | --- | --- |
| Standalone simulator, HTTP commands and MQTT telemetry | Existing | [E1], [E7], [E8] | Preserve and reuse |
| Workpieces, locations, occupancy checks and transfers | Extend | [E2] | Separate the fixed layout from state handling |
| Manipulator and transport services already shared within FischerTechnik | Extend | [E3], [E4] | Reuse across both layouts |
| Configurable action durations | Existing | [E5] | Retain through a common timing boundary |
| Shared timing abstraction | New | Direct sleeps and clock reads in services [E4], [E6] | Centralise action timing and test control |
| VGR movement failure | Extend | [E4], [E7] | Reuse in selected scenarios |
| Selected telemetry suppression | New | No scenario suppression hook identified in the inspected publisher [E8] | One narrowly scoped communication-fault effect |
| Second runnable layout | New | IMP proposals only [E10] | Assemble one route on the shared core |
| Tests and result reporting | Extend | 16 existing Java test files [E9] | Reuse tests and add focused acceptance cases |

### 4.2 Gaps and their impact

| Gap | Impact for developers | Code evidence | Requirements |
| --- | --- | --- | --- |
| G1: Fixed layout definitions | Another layout requires edits to factory-specific definitions in several places | Fixed locations, machine IDs and oven constants [E2], [E3], [E6] | FR-02, NFR-01 |
| G2: Factory-specific message mappings | Reusing behaviour and interpreting interactions requires separating factory message meaning from shared logic | Sensor mappings and sorter-specific publication behaviour [E4], [E8] | FR-01, FR-03, NFR-01 |
| G3: Distributed timing code | Durations are configurable, but tests lack a common way to control action completion | Individual services sleep and read the clock [E4], [E6] | FR-04 |
| G4: Limited scenario controls | Existing delay settings and the VGR failure do not form a shared approach to selected fault scenarios | VGR failure API; no selected-message suppression hook found [E5], [E7], [E8] | FR-05, NFR-02 |
| G5: Cross-layout reuse not demonstrated | The proposed generalisation still needs evidence from another model | IMP design documents without an implementation [E10] | FR-02, NFR-01, NFR-02 |

## 5. Scope

**Core proposal:** One Java simulator; the existing FischerTechnik behaviour; one route from input through manipulator A, conveyor and manipulator B to output; shared state, transfer and timing components; selected fault scenarios; tests, run instructions and extension documentation.

Layout-specific configuration, Java components and adapters are allowed. A small test client can drive the second route. Existing interfaces and test output provide the starting point for observing interactions.

**Outside this proposal:** Additional lab models, detailed physics, 3D visualisation, a layout editor, an arbitrary behaviour language, distributed simulation, changes to the existing business application, a custom chaos platform, automatic recovery, dashboards, cloud deployment and hot reset of active runs.

## 6. Requirements and acceptance criteria

### FR-01: Preserve FischerTechnik interfaces and behaviour

The refactored simulator shall retain the machine commands, HTTP responses and MQTT telemetry used by the existing application.

**Acceptance:** Compare six representative baseline groups before and after refactoring:

1. Item creation, listing, movement and deletion, plus relevant status queries.
2. VGR and workstation-transport (WT) commands.
3. Oven burn.
4. Milling movement and the `mill` wrapper.
5. Sorter transport, colour detection and motor-status behaviour.
6. VGR failure control.

Reuse existing tests where suitable. Check required response fields, statuses, item movements and the five documented MQTT topic structures. Preserve existing blocking behaviour; timestamps may vary. Record pre-existing failures separately from regressions. The groups bound the evaluation coverage without requiring replacement of the existing test suite.

### FR-02: Run two layouts on the same core

Both layouts shall use the same shared state, transfer and timing components.

**Acceptance:** One representative FischerTechnik flow and one complete second-layout route succeed with a single workpiece. In the final implementation, selecting either layout uses its configuration or module without editing the shared core. Record core changes made while developing the second model.

The second route is input -> manipulator A -> conveyor -> manipulator B -> output. Section 9 addresses its physical reference.

### FR-03: Define and observe machine interactions

The demonstrated routes shall have defined commands, states, transfers and observable outcomes.

**Acceptance:** Specify the input, relevant state changes and completion condition. Check the nominal route, an occupied destination and an invalid transfer. A failed transfer preserves valid item locations without loss, duplication or overwrite. Retain existing FischerTechnik waiting behaviour; define waiting or rejection explicitly for the second model.

Use existing HTTP/MQTT observations, state snapshots or test assertions as evidence. Check required causal ordering, such as an item being available before the next machine picks it up. Independent observations may occur in a different order.

| Interaction | Observation needed |
| --- | --- |
| A places the workpiece on the conveyor | Item available at conveyor input |
| Conveyor completes the transfer | Item available at conveyor output |
| B picks up and places the workpiece | Item reaches the final destination |

### FR-04: Centralise timing and support controlled tests

Machine actions shall obtain their timing through a shared abstraction, with configurable durations and controllable completion in tests.

**Acceptance:** Movement and oven processing use the common timing boundary. A controlled test starts an action, checks its busy state before completion, advances to completion and checks the resulting state without waiting for the full real duration. Normal operation preserves the existing interface behaviour.

### FR-05: Support selected fault scenarios

The simulator shall support the agreed delay, movement-failure and telemetry-suppression scenarios from Section 7, with defined triggers, effects and clearing procedures.

**Acceptance:** Demonstrate each selected effect on one representative target. Record its trigger, observable effect and item state; after clearing, a subsequent normal run succeeds. Configuration, existing APIs or a test script may trigger the scenario. Confirm the final scenario selection in Section 9.

### NFR-01: Keep supported layout changes outside the core

Factory-specific locations, machine instances, connections and message mappings shall be separate from shared state, transfer and timing logic.

**Acceptance:** Provide a component map for both layouts and document two changes: adding an instance of an existing machine type and changing a supported route. Both changes use configuration or layout-module wiring rather than core edits. New machine behaviour can use a new Java component. Record changed files and any limits encountered.

### NFR-02: Make runs repeatable and results inspectable

A documented procedure shall establish a known state, execute a scenario and expose its result.

**Acceptance:** Repeat each selected scenario three times from a fresh start. Compare final item locations, outcome and required causal order. A short result table or existing test report records scenario, inputs, expected result, actual result and pass/fail. Provide run instructions for both layouts. A fresh process per run is sufficient.

## 7. Proposed scenario set

| Scenario | Setup | Expected result |
| --- | --- | --- |
| S0: Normal run | One workpiece, known start state, no fault | Route completes with the correct output and required causal order |
| S1: Delay | Increase one action duration before the run | Machine remains busy until completion; route still completes correctly |
| S2: Movement failure | Enable existing VGR failure before pickup | Item stays at its valid pre-transfer location; a normal run succeeds after clearing |
| S3: Telemetry suppression | Suppress one outgoing topic for at least two publication intervals | Selected topic is silent; another topic continues; publishing resumes after clearing |

Run S0 on both layouts. For faults, one representative target per effect is sufficient; there is no requirement to test every combination of layout, machine and fault. S2 retains the existing skipped-transfer behaviour while the task still completes. S3 resumes with current state rather than replaying suppressed messages.

## 8. Evaluation plan

| Evaluation | Requirement | Evidence | Success criterion |
| --- | --- | --- | --- |
| EVAL-01 | FR-01 | Before/after baseline results | Required behaviour preserved; deviations resolved or agreed |
| EVAL-02 | FR-02 | Two routes and component map | Both layouts use the same core and can be selected without editing it |
| EVAL-03 | FR-03 | Nominal and invalid-transfer checks | Valid item locations and required causal order |
| EVAL-04 | FR-04 | Controlled timing tests | Correct busy/completion states at configured test times |
| EVAL-05 | FR-05 | Selected S1-S3 results | Effects match their specifications and can be cleared |
| EVAL-06 | NFR-01 | Two documented extension tasks | Changes remain in configuration or layout-module wiring |
| EVAL-07 | NFR-02 | Repeated runs and run instructions | Same defined outcomes from the same initial state |

Reuse runs across evaluations. Compare the two extension tasks with the corresponding baseline code to explain what changed. Evidence supports the demonstrated layouts and tasks; physical accuracy requires a separate hardware reference. Runtime test results and any useful duration measurements will be collected during implementation.

## 9. Open points for 30 September

| Topic | Current view | Input we would like |
| --- | --- | --- |
| Priorities and evaluation scope | Sections 2 and 3 describe the core | Which parts matter most for the lab, and can we agree on six baseline groups, the selected fault scenarios with one target per effect and two extension tasks as the evaluation scope? |
| Faster-than-real-time execution | Possible extension on top of the shared timing (FR-04); not part of the core | Is accelerated execution relevant for the intended use? |
| Dedicated interaction trace | Possible extension; existing messages, state checks and test results may be sufficient | Would a separate trace of machine events add enough value? |
| Fault scenarios | S1-S3 as a starting set, triggered through existing controls or scripts | Are these the right scenarios? |
| Second layout | Two manipulators and a conveyor | Is there a matching setup in the lab? If not, is this route sufficient as a representative model with an agreed interaction contract? |

Accelerated execution would also affect fault intervals, message timestamps and the visibility of short states at the 400 ms MQTT interval, so it would need its own scope definition if selected. Additional lab models remain outside the proposed scope.

## 10. Five-minute storyline

| Step | Message | Time |
| --- | --- | --- |
| 1. User problem | Developers need to test machine interactions and repeat selected failure cases; physical testing depends on lab access. | 40 s |
| 2. Existing solution and gap | The current simulator enables offline development, but adaptation still involves factory-specific code. Show two examples. | 60 s |
| 3. Goal and requirements | Generalise the simulator for two layouts and make the interactions between machines testable, while preserving existing behaviour. | 70 s |
| 4. Solution | Shared state, transfer and timing components with separate layout definitions and adapters. | 65 s |
| 5. Evaluation and next step | Show how requirements map to checks, the agreed scope and the next implementation step. | 65 s |

The previous project supplies the baseline in step 2. Understanding its business application is not needed to follow the presentation.

## 11. Preparation until 8 October

| Date | Work | Output |
| --- | --- | --- |
| 22-27 September | Review gaps, core requirements and representative routes together | Team position and questions for Amine |
| 28-29 September | Prepare the storyline and first slide draft | Discussion-ready concept and slides |
| 30 September | Discuss Section 9 with Amine | Recorded scope decisions and updated acceptance cases |
| 1-4 October | Apply feedback and align the existing design documents | Consistent concept and presentation |
| 5 October | Review with Barbara and Amine | Final corrections |
| 6-7 October | Finalise and rehearse | Stable slides and agreed speaking parts |
| 8 October | Progress presentation | Concept, evidence from the baseline review and implementation plan |

The preparation deliverable is the concept and presentation. A benchmark or prototype is not planned before 8 October. Baseline execution and implementation start once the scope is agreed.

## Appendix: Evidence

Code references point to `edpo-project` at commit `868d848` and `imp-simulator` at commit `ddc8e3d`.

- **[E1]** [factory-simulator/README.md](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/README.md): simulator features and operation
- **[E2]** [FactorySimulatorService.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/service/FactorySimulatorService.java): `initializeSinks`, `addItem`, `tryMoveItemBetweenSinks`
- **[E3]** [VacuumGripperConfiguration.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/service/VacuumGripperConfiguration.java): existing reuse and fixed wiring
- **[E4]** [VacuumGripperService.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/service/VacuumGripperService.java) and [OneWayPointToPointTransportService.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/service/OneWayPointToPointTransportService.java): transport, timing and failure behaviour
- **[E5]** [FactorySimulationProperties.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/service/FactorySimulationProperties.java) and [application.yaml](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/resources/application.yaml): durations and publish interval
- **[E6]** [OvenService.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/service/OvenService.java): oven locations and timing
- **[E7]** [FactoryApiController.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/controller/FactoryApiController.java) and [machine controllers](https://github.com/sejmasijaric/edpo-project/tree/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/controller): item, status and failure API
- **[E8]** [VacuumGripperMqttPayloadFactory.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/mqtt/VacuumGripperMqttPayloadFactory.java), [AbstractMqttPublisher.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/mqtt/AbstractMqttPublisher.java), [SinkMovementMqttListener.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/mqtt/SinkMovementMqttListener.java) and [MqttTimestampFactory.java](https://github.com/sejmasijaric/edpo-project/blob/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/main/java/org/unisg/ftengrave/factorysimulator/mqtt/MqttTimestampFactory.java): MQTT communication
- **[E9]** [Simulator test tree](https://github.com/sejmasijaric/edpo-project/tree/868d848f9f2d6d4cdfde790ee014e90bbbd6f910/factory-simulator/src/test/java/org/unisg/ftengrave/factorysimulator): existing tests
- **[E10]** [README](https://github.com/sejmasijaric/imp-simulator/blob/ddc8e3d3d19ccd08fd5376803f1e81a198047ad5/README.md), [design.md](https://github.com/sejmasijaric/imp-simulator/blob/ddc8e3d3d19ccd08fd5376803f1e81a198047ad5/design.md), [docs/design.md](https://github.com/sejmasijaric/imp-simulator/blob/ddc8e3d3d19ccd08fd5376803f1e81a198047ad5/docs/design.md), [scope-proposal.md](https://github.com/sejmasijaric/imp-simulator/blob/ddc8e3d3d19ccd08fd5376803f1e81a198047ad5/scope-proposal.md) and [project-plan.html](https://github.com/sejmasijaric/imp-simulator/blob/ddc8e3d3d19ccd08fd5376803f1e81a198047ad5/presentation/project-plan.html): current IMP proposals
