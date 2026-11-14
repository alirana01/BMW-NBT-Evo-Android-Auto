# Engineering Backlog

These are research directions, not implementation commitments.

## Desktop simulation — P0

- **Goal:** make routine connection, protocol, navigation, and media development vehicle-free.
- **Evidence:** the repository already has `fake_dsi_client`, fake KOMBI, captured controller fixtures, WPP/session tests, and golden navigation transcripts.
- **Unknowns:** which G30 events can be captured and shared safely; gaps between host fakes and target timing.
- **Approach:** containerize host prerequisites; add a timestamped scenario runner and fault injection around existing fakes. Keep the QNX cross-build separate.
- **Dependencies and risks:** sanitized captures and explicit fixture provenance; never imply full QNX/NBT emulation.
- **Success:** one command reproduces connect/reconnect, navigation, media, and failure scenarios deterministically; full host suite passes.

## G30 HUD and cluster navigation — P0 research, P1 implementation

- **Goal:** show turns, distance, road names, and supported lanes without disrupting factory navigation.
- **Evidence:** AA channel 10 reaches `KombiNav`; J29 SOME/IP and F-series 6WA MOST drivers exist. Waze produced no visible G30 HUD guidance in the reported test.
- **Unknowns:** G30 cluster type/firmware, service transport/profile, settings used in the test, channel-10 payloads, and safe factory-nav ownership detection.
- **Approach:** record app-tagged channel-10 logs; identify the G30 interface read-only; compare with J29/6WA; implement a shadow profile and fake transport before guarded writes.
- **Dependencies and risks:** wrong profiles may overwrite factory guidance or leave stale graphics. Never hardcode an unverified CAN/MOST/SOME-IP message.
- **Success:** shadow mode predicts correct start/reroute/cancel events; stationary test yields cleanly to factory navigation; controlled drive confirms display and cleanup.

## Waze and Google Maps comparison — P1

- **Goal:** determine whether missing guidance is app payload, configuration, or G30 output related.
- **Evidence:** the callback is app-agnostic; source handles status, maneuver, distance, road/lane data where present.
- **Unknowns:** actual payload completeness and timing for each app/version.
- **Approach:** capture sanitized channel-10 traces for the same route and replay them into `KombiNav` tests.
- **Dependencies and risks:** phone/app versions change behavior; avoid generic Android Auto assumptions.
- **Success:** a field/timing matrix and replay tests for start, turn, reroute, and cancellation.

## Media metadata and thumbwheel — P1

- **Goal:** reliable title/artist/album display, previous/next, and supported thumbwheel selection/list behavior.
- **Evidence:** channel-12 parsing, `/dev/shmem/nbt-aad-media`, steering-wheel previous/next, J29-oriented cluster media history, and an optional media-browser probe exist.
- **Unknowns:** G30 display/list service, current thumbwheel mapping, phone browse-tree support, and safe ownership alongside factory media.
- **Approach:** log/replay metadata and controls; map read-only factory behavior; test acknowledgment and stale-state cleanup; add a G30 adapter only after evidence.
- **Dependencies and risks:** duplicate skips, focus changes, stale metadata, or interference with calls/radio.
- **Success:** exact host tests plus stationary G30 validation across track changes, app switches, pause/resume, reconnect, and source changes.

## Service-history research — P3

- **Goal:** read/export service entries and maintain clearly separate personal records.
- **Evidence:** no matching implementation was found in the current tree.
- **Unknowns:** data owner, storage/service API, access control, signature/provenance, and legal constraints.
- **Approach:** documentation/static analysis and read-only observation first; store personal records outside factory data.
- **Dependencies and risks:** corrupting records or misrepresenting unofficial entries. Factory writes require separate explicit approval and a safety design.
- **Success:** a documented read-only schema/provenance and export prototype using test data; no vehicle mutation.

## Tire/wheel configuration research — P3

- **Goal:** understand whether the OEM UI can represent 245/35 R20 front and 275/30 R20 rear winter tires.
- **Evidence:** no tire/RDC/TPMS configuration implementation was found in this repository.
- **Unknowns:** configuration owner, valid option tables, coding interface, load/pressure constraints, and whether the combination is approved for this vehicle.
- **Approach:** read-only documentation, UI-resource, and diagnostic-schema research. Consult authoritative tire/vehicle specifications before any proposal.
- **Dependencies and risks:** safety, legal approval, RDC/TPMS behavior, and vehicle coding. No changes are authorized by this task.
- **Success:** evidence-backed ownership and constraints document; no vehicle mutation.

## Persistent project knowledge — P1

- **Goal:** keep evidence and decisions usable across Codex sessions.
- **Evidence:** source comments are detailed but combine multiple vehicles and evidence types.
- **Approach:** update [Project Status](PROJECT_STATUS.md) after each test, [Testing](TESTING.md) after harness changes, and architecture docs after interface changes. Record vehicle/firmware/build with every target result.
- **Risk:** stale or vehicle-ambiguous claims.
- **Success:** every feature has a source reference, evidence label, validation level, and unresolved questions.
