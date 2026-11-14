# Project Status

Status labels in this document are evidence classifications, not maturity ratings.

## Vehicle baseline

- **REPORTED VEHICLE TEST** — Target vehicle: 2017 BMW M550i xDrive (G30), NBT Evo, iDrive 6, factory CarPlay, accessed through ENET. Android Auto connects while driving, reconnects after an ignition-off/refueling stop, and starts automatically.
- **REPORTED VEHICLE TEST** — Audio works and was subjectively better than the previous external bridge. Reversing camera and tested BMW functions continued to operate normally.
- **UNKNOWN** — Exact NBT Evo firmware/I-step, cluster type, enabled environment settings, phone model/Android version, and whether the tested audio path was `bt` or `browser`. Capture these before interpreting model-specific results.

## Current behavior and limitations

- **VERIFIED IN SOURCE** — `milestone-7/src/main.cpp` initializes wireless/optional USB transport, an `AASession`, video, an audio backend, BMW input consumers, and optional cluster navigation. `rhmiapp/nbt_aad_app.cpp` implements the iDrive entry, autostart flag, settings, and a boot-loop guard.
- **VERIFIED IN SOURCE** — Channel 10 navigation messages are forwarded by `AASession::setNavStatusCallback()` to `nbt::kombinav::KombiNav`. Current display drivers are J29 SOME/IP and F-series 6WA MOST HUDTerminal (`cluster/nav_core.cpp`). There is no G30-specific driver/profile in the current tree.
- **REPORTED VEHICLE TEST** — Waze guidance did not appear in the G30 HUD.
- **UNTESTED** — Google Maps HUD behavior on this G30. Also untested/documented here: whether navigation forwarding was enabled, whether the G30 cluster exposes a compatible service, lane guidance, factory-navigation conflict handling, media list browsing, and all supported firmware variants.
- **VERIFIED IN SOURCE** — Channel 12 media status/metadata is parsed and published through `/dev/shmem/nbt-aad-media`; steering-wheel previous/next and optional cluster media paths exist in `main.cpp` and `src/car/`.
- **UNKNOWN** — Which media metadata and thumbwheel behaviors work on this G30. Existing J29-specific media work must not be attributed to the BMW test without evidence.

## Bluetooth milestone

- **REPORTED VEHICLE TEST** — A diagnostic run showed repeated lack of an Android Auto SPP link, then `SppUuidDeregister`/`SppUuidRegister` using service `AndroidAuto` and UUID `4DE17A00-52CB-11E6-BDF4-0800200C9A66`; the connection succeeded shortly afterward. This was an important debugging milestone, but does not prove causation or universal message IDs.
- **VERIFIED IN SOURCE** — `bluetooth_manager.cpp` currently registers through DSI service `SGBluetoothGap`, clears stale registrations, and can deregister/register again. `main.cpp` uses `SppReregTimer`; the schedule is configurable by `NBT_SPP_REREG_S`. A separate 90-second limit applies after a successful wireless handshake while waiting for the phone's TCP connection. Do not describe the current retry policy as a fixed 90-second re-registration.
- **VERIFIED IN SOURCE** — Daemon and app RAM logs default to `/dev/shmem/nbt-aad.log` and `/dev/shmem/nbt-aad-app.log`. Useful read-only checks include `tail -n 50 /dev/shmem/nbt-aad.log` and `grep Build /dev/shmem/nbt-aad.log`.

## Development priorities

1. **PROPOSED** — Record a reproducible G30 baseline: firmware, cluster, phone, effective environment, startup-to-session logs, and a sanitized MOST/DSI observation set.
2. **PROPOSED** — Build a desktop scenario runner around existing `fake_dsi_client`, fake KOMBI, WPP/session tests, and captured event fixtures. Start with Bluetooth registration/reconnect states and navigation channel-10 replay.
3. **PROPOSED** — Identify the G30 cluster/HUD service read-only, compare it with current J29/6WA interfaces, and add a shadow/log-only profile before any display write.
4. **PROPOSED** — Verify channel-12 metadata and steering-wheel behavior on the G30, then isolate missing presentation or control mappings.
5. **PROPOSED** — Research service history and tire configuration read-only; no factory-record, RDC/TPMS, coding, or safety-setting changes are authorized.

## Open questions

- What G30 cluster/HUD protocol and profile are present, and can factory navigation activity be detected reliably?
- Do Waze and Google Maps emit equivalent channel-10 messages in this implementation? Which fields are populated?
- Which audio mode produced the successful BMW test, and what rollback package is installed?
- Does the G30 expose media text/list services compatible with current J29 work?
- Can sanitized head-unit captures be added as fixtures without personal or proprietary data?

See [Ideas](IDEAS.md) for candidate work and [Testing](TESTING.md) for validation levels.
