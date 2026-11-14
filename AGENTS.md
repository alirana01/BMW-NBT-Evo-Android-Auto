# Repository Guidelines

## Purpose and operating context

NBT-AAD projects Android Auto through the factory QNX-based NBT Evo/iDrive 6 head unit. This fork is used in a 2017 BMW G30 M550i xDrive with factory CarPlay. Android Auto connectivity, reconnect, audio, autostart, reversing camera, and normal iDrive use have been reported working in that vehicle. Preserve those behaviors. Treat other models, firmware, cluster protocols, and all untested edge cases as separate validation targets.

Begin every task with [Project Status](docs/PROJECT_STATUS.md), then read the task-specific [Architecture](docs/NBT_ARCHITECTURE.md), [Android Auto](docs/ANDROID_AUTO.md), [Testing](docs/TESTING.md), or [Ideas](docs/IDEAS.md) reference. Inspect relevant source before changing it; comments may mix car observations, reverse-engineering evidence, and inference.

## Repository layout and architecture rules

Active code is in `milestone-7/`. `src/` contains the daemon; `src/audio/`, `src/car/`, `src/most/`, and `src/cluster/` contain subsystems. `rhmiapp/` is the native iDrive app, `plugin/` the HMI plugin, `tests/` host tests and fixtures, `proto/` protocol schemas, and `tools/` target diagnostics. Root Markdown files cover installation and updates.

The target is QNX ARMv7LE with QNX SDP 6.5.0 and GCC 4.4.2-era `gnu++0x`. Keep C++11-era compatibility. The QNX Makefile is authoritative; CMake is a source/build description. Do not replace proprietary QNX/BMW interfaces with guessed equivalents. Preserve existing SPDX and required notices.

Confirmed boundaries include Android Auto framing/TLS in `aa_session.*` and `aa_cryptor.*`, wireless setup in `wireless_setup.*`, DSI Bluetooth registration in `bluetooth_manager.cpp`, QNX video in `video_manager.*`, audio backends in `src/audio/`, and BMW controls/vehicle data through `most::Client`. Current cluster drivers support J29 and an F-series 6WA profile; do not assume either matches a G30.

## Build and test commands

From the repository root:

```sh
QNX_LICENSE_DIR=/path/to/license-dir ./build.sh
./build.sh milestone-7 rhmiapp
make -C milestone-7 check
make -C milestone-7/tests run-test_pcm
```

`QNX_LICENSE_DIR` must contain a writable `licenses` file; `QNX_IMAGE` overrides Docker image `qnx650-sdk`. Host checks are Linux-oriented and require GCC/G++, OpenSSL development libraries, Python 3, and `setarch`; `protoc` and a 32-bit toolchain enable optional checks. Record skips and environment-caused failures. Never claim QNX, head-unit, or vehicle validation from host tests.

## Coding, testing, and review

Follow nearby style: four-space indentation, same-line braces, `snake_case` files, `.hpp` headers, and existing class/method naming. Add focused `test_<feature>.cpp` coverage and register linked sources in `tests/Makefile`. Prefer deterministic fakes, captured-fixture replay, state-machine tests, and golden transcripts. Never update a golden fixture merely to make a failure disappear.

Review changes for session/reconnect lifecycle, Bluetooth registration, audio routing, video/camera handoff, iDrive input, factory-navigation yielding, cleanup, resource limits, and QNX compiler compatibility. Pull requests must state host checks, skipped checks, target hardware/firmware tested, rollback steps when deployment changes, and any remaining real-car validation. Use descriptive commit subjects; release commits use the existing `Version:` convention.

## Diagnostics, safety, and documentation

The default RAM logs are `/dev/shmem/nbt-aad.log` and `/dev/shmem/nbt-aad-app.log`; they disappear on reboot. Redact phone identifiers, Wi-Fi credentials, license files, and private captures before sharing. Do not simplify reconnect or cleanup logic without a regression test and a vehicle-validation plan.

Do not invent BMW message IDs, QNX APIs, DSI/MOST layouts, protocol fields, or test results. Label evidence as source-verified, reported vehicle test, untested, proposed, or unknown. Experimental writes require captured evidence, a host-side fake/replay test, bounds and timeout handling, factory-feature yielding, and explicit vehicle approval. Do not modify ECU coding, TPMS/RDC, service records, firmware, or safety-critical traffic as incidental work.

Update `docs/PROJECT_STATUS.md` and `docs/TESTING.md` when implementation or validation status changes; update architecture/protocol docs when interfaces change. Keep temporary debugging chatter out of permanent docs.
