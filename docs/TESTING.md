# Testing Strategy

## Existing host tests

`milestone-7/tests/Makefile` builds standalone assertion-based C++ tests plus Python schema/payload checks. `make -C milestone-7 check` runs the suite, an optional 32-bit protocol check, schema validation when `protoc` exists, and a ThreadSanitizer session-lifecycle run. Tests cover, among other areas:

- Android Auto protocol, session control/lifecycle/timeout/sensors, audio acknowledgements, and service discovery;
- wireless setup, AP candidates, USB, Bluetooth-link policies, and re-registration timing;
- audio configuration, routing, PCM/resampling/mixing, volume, and voice scenarios;
- touch, controller captures, MOST parsing/client/router/reassembly, GPS and vehicle sensors;
- J29 navigation state/SOME-IP using a fake KOMBI and 6WA/HUDTerminal using fake DSI.

On 2026-11-14, `make -C milestone-7 check` was attempted on the current Darwin ARM64 host. Compilation stopped at the first binary because the linker could not find `-lssl`; no test passed in that run. The suite and TSAN command are Linux-oriented (`setarch -R`). This is an environment failure, not a product failure.

## Recommended desktop environment

Use a versioned Linux container or VM for host tests, separate from the licensed QNX cross-build container. Install GCC/G++, OpenSSL development headers/libraries, Python 3, protobuf compiler, multilib support where feasible, and `setarch`. Mount the repository read/write and persist only test artifacts. Run:

```sh
make -C milestone-7 check
make -C milestone-7/tests run-test_session_lifecycle
make -C milestone-7/tests run-test_kombi_nav
make -C milestone-7/tests run-test_hud6wa
```

Build a scenario runner on the existing fakes rather than attempting a complete NBT Evo VM. A useful first version should consume sanitized timestamped events and drive:

1. DSI attach/register/response/detach sequences for Bluetooth GAP and MOST.
2. WPP handshake outcomes followed by TCP connect, timeout, short session, sleep/wake, and transport handoff.
3. Channel-10 navigation payload sequences for start, maneuver, reroute, cancellation, stale input, and factory-navigation yield.
4. Channel-12 metadata plus wheel/controller inputs.

Keep protocol payloads byte-exact, label their capture source/firmware, and make outputs deterministic. Add fault injection for delayed/missing responses, malformed lengths, disconnects, queue limits, and repeated reconnects. This provides high-value desktop coverage without redistributing QNX firmware or pretending to emulate the full head unit.

## Validation levels

| Level | Suitable checks | Cannot establish |
|---|---|---|
| Desktop pure/fake | parsers, state machines, retry policy, exact serialization, cleanup, race checks | QNX ABI or BMW service availability |
| QNX cross-build | compiler compatibility and target linking | runtime service behavior |
| Head unit, stationary | process launch, DSI/PPS/Screen/audio services, log collection, display handoff | driving behavior and every camera/safety interaction |
| Actual car | reconnect after sleep, controls, audio, camera priority, cluster/HUD, driving lifecycle | compatibility with other firmware/models |

## Change-specific regression requirements

- Session/wireless/Bluetooth: test success, timeout, stale registration, repeated short sessions, sleep/wake simulation, and resource cleanup.
- Video/UI: test focus transitions and state logic on host; in car verify reverse, PDC/camera, hard keys, CarPlay handoff, and return to Android Auto.
- Audio: test all affected stream formats, underflow/acks, route cancellation, and session teardown; in car verify radio interruption, prompts, calls/assistant, and volume.
- MOST/cluster: use invented addresses only in pure tests; replay sanitized captures separately. Start new vehicle support in shadow/log-only mode and prove factory-navigation yielding before writes.
- Golden fixtures: review byte-level diffs and document why protocol output changed. Never regenerate merely to clear a failure.

## Deployment and in-car checklist

Before deployment, retain the known-good daemon and RHMI package, record checksums/configuration, copy volatile logs, and follow the rename-based rollback in root `updating.md`. Do not overwrite a running binary.

Stationary validation: clean launch; phone discovery; wireless/USB connect as applicable; audio/media/guidance; knob, touch, BACK and steering-wheel controls; iDrive exit/return; reverse and all available camera/PDC views; factory navigation coexistence; disconnect/reconnect; clean stop; no rapid process spawning or memory growth. Then perform a short controlled drive, ignition-off sleep/restart test, and collect both logs before reboot if anything fails.

Record results as host, QNX build, head-unit stationary, or actual-car evidence. Include vehicle, firmware/I-step, cluster, phone/Android, transport, audio mode, environment flags, build hash, and observed failures.
