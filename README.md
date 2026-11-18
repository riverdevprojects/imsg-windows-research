# iPhone Messages on Windows: Research

What it would take to build River's own app for sending and receiving iPhone messages (iMessage + SMS) on Windows.

## The requirements

1. Send and receive messages from the Windows PC
2. Works when the iPhone and PC are on different Wi-Fi networks
3. Session stays logged in, no re-pairing rituals
4. Group chats required
5. Some message history (a little is fine)
6. Pairing that works every time, QR-scan simple

## TL;DR

- The full requirement set is achievable, but not over Bluetooth and not with the iPhone alone. Apple gives apps zero access to messages, so every working design needs a bridge device that Apple does trust with messages. Two kinds of bridge exist: Bluetooth proximity (the PC pretends to be a car) and a Mac signed into your Apple ID.
- Only the Mac-bridge design meets all six requirements. Bluetooth-only fails #2 (10 m range), #4 (no group iMessage over Bluetooth), and #5 (inbox snapshot only, your own sent texts never sync).
- Recommended build: run a BlueBubbles server on the MacBook Pro (plugged in, always awake), then build your own Windows app on its documented API. Your app pairs by scanning one QR code (the server's URL + password). That QR is just data, not a Bluetooth handshake, so it works every time. The build spec lives in the sister repo `imsg-windows`.
- The pure-Bluetooth Windows app (the original idea) is possible for the first time as of iOS 26.5, and Linux projects prove it works, but even when finished it cannot do group chats. It fails your own spec.

## What iOS actually allows

- No app API for messages. There is no way for an iOS app to read or send SMS/iMessage. Any "app on the iPhone" design is dead on arrival.
- Bluetooth is the one official pipe: MAP (Message Access Profile, classic Bluetooth) for messages, ANCS (Apple Notification Center Service, BLE) for notifications. This is how car dashboards and smartwatches do texts.
- New in iOS 26.5 (2026): iOS exposes iMessage and RCS over MAP, not just SMS. EU smartwatch-interop rules (Digital Markets Act) forced this open. Proven in the wild by iphonebridge, BlueFerry, and Tether on Linux. No region lock has been reported by those projects; verify on your own phone before relying on it.
- The trick everyone uses: the computer spoofs its Bluetooth Class-of-Device as a car entertainment system, so the iPhone offers the "Show Message Notifications" toggle and grants MAP access. Apple cannot block this without breaking every car and smartwatch.
- MAP hard limits (Apple's stack, not bugs):
  - 1-to-1 only. No group iMessage, no MMS groups, no RCS groups.
  - No attachments, reactions, read receipts, or typing indicators.
  - Inbox only. Texts you send from the iPhone itself never sync to the bridge; only messages sent from the bridge are recorded.
  - Recent messages only, no full history.

## Requirement-by-requirement

| Requirement | Bluetooth bridge | Mac server bridge |
|---|---|---|
| Different Wi-Fis | Yes (Bluetooth ignores Wi-Fi), but only within ~10 m | Yes, anywhere with internet (Cloudflare Tunnel) |
| Stays logged in | Bluetooth bond persists and auto-reconnects in range, but reliability depends on the adapter and drivers | Server session never dies; the client stores a URL + password once |
| Group chats | No. MAP is 1-to-1; the ANCS side-channel hack only reconstructs some group context on receive | Yes, full: groups, attachments, tapbacks |
| History | Recent inbox only, no sent folder | Full history (reads the Messages database) |
| QR pairing | QR only bootstraps; the real pairing is iOS Bluetooth dialogs (this is the step Phone Link keeps failing) | QR carries URL + password, deterministic |
| Send + receive | Both | Both |

## More detail

- [docs/architectures.md](docs/architectures.md): the three architectures in full, including the recommended BlueBubbles-on-Mac design.
- [docs/sources.md](docs/sources.md): prior art and every source.
