# Android Auto Implementation

## Connection lifecycle

1. `BluetoothManager` registers the `AndroidAuto` SPP service through BMW's QNX DSI Bluetooth GAP service.
2. `main.cpp::wirelessThread()` scans `/dev/btspp*`, avoids links identified as BMW companion-app traffic, reads either the CarPlay hotspot or optional Wi-Fi Direct configuration, and runs the wireless projection protocol in `wireless_setup.*`.
3. The phone joins the selected network and connects to `TcpServer` on the configured Android Auto port (default 5277). Optional USB transport is implemented by `usb_accessory.*`/`usb_direct.*` and serialized with TCP by `SessionOwner`.
4. `startSession()` binds the accepted descriptor to `AASession`. Channel 0 performs version negotiation, TLS authentication through `AACryptor`, authentication completion, and service discovery.
5. Media channels are opened on demand. Session end clears transport ownership, audio/video state, queued frames, and pending routing work before another connection is accepted.

The service discovery response is built by `sdr_builder.*`; do not infer services solely from generic Android Auto documentation.

## Channels used by this implementation

| Path | Source behavior |
|---|---|
| Control, auth, focus | Channel 0 in `aa_session.cpp` |
| Main H.264 display | Channel 6; frames go to `VideoManager` |
| Touch input | Advertised touchscreen channel; events originate in `touch_probe.*` |
| Guidance audio | Channel 8 when an AA audio sink is advertised |
| Media audio | Channel 9 when the selected audio mode advertises it |
| Navigation status | Channel 10; callback to `KombiNav` |
| Media status/metadata | Channel 12; callback to `MediaState` |
| Microphone uplink | Channel 14 |

## Wireless and reconnection

`SppReregTimer` controls automatic Bluetooth service re-registration while no Android Auto SPP link appears; `NBT_SPP_REREG_S=0` disables it. `BluetoothManager::reregister()` deregisters and registers again, refuses overlapping in-flight requests for 15 seconds, and clears stale initial registrations. The historical G30 log showing a retry at 90 seconds is retained as reported evidence in [Project Status](PROJECT_STATUS.md); current code does not define one universal 90-second re-registration rule.

After wireless setup, the Bluetooth link remains serviced while the phone joins Wi-Fi and connects over TCP. A separate 90-second wait limit applies at that stage. Session outcome, AP candidates, interface state, and short-session handling guide recovery; see `main.cpp`, `tcp_server.cpp`, and `wireless_setup.cpp`.

## Rendering, audio, controls, and navigation

`VideoManager` feeds H.264 to QNX `mm-renderer`, owns Screen/LayerManager presentation state, and cooperates with RHMI so factory UI and camera views can take priority. This is a critical regression surface.

Audio behavior depends on `NBT_AUDIO_MODE`: `bt` (default) leaves phone media on A2DP and requests the factory Bluetooth source; `browser` mixes AA PCM channels through the head-unit browser audio path; `a2i` is a legacy staged path whose source comments say it was not audible on the tested car.

iDrive/touch/steering-wheel events are translated in `main.cpp`, `car/ccs_input.*`, and `touch_probe.*`, then posted to `AASession`. Channel-12 metadata is merged by `audio/media_state.hpp` and exposed to RHMI/control consumers.

The phone's channel-10 guidance status, maneuver, distance, and related payloads are decoded by `cluster/nav_msg.hpp` and processed by `KombiNav`. Current output drivers are J29 and 6WA only. The callback boundary does not identify Waze versus Google Maps. Payload completeness and G30 display compatibility require capture and vehicle-specific validation.

## Logs and failure classes

Use `/dev/shmem/nbt-aad.log` for daemon lifecycle and `/dev/shmem/nbt-aad-app.log` for RHMI behavior; both are volatile. USB logging and `NBT_LOG_FILE` are described in `milestone-7/readme.md`.

- No SPP node: registration, phone Bluetooth/A2DP state, or a competing companion-app link.
- Handshake but no TCP: AP credentials/interface, firewall/listener, or phone join failure.
- Repeated short sessions: TLS, service discovery, or channel failure; inspect the recorded session outcome.
- Black/hidden video: `mm-renderer`, Screen/LayerManager surface, focus, or iDrive handoff.
- Silent audio: selected backend, `audioconn`, A2DP source, PCM device/rate, or focus.
- No HUD guidance: setting disabled, absent channel-10 data, incomplete app payload, factory-nav yield, or unsupported cluster profile.

Never diagnose from one log line or reuse a BMW DSI/MOST identifier on another firmware without confirming its service version and payload.
