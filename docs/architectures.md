# Architectures

## Architecture A (recommended): your Mac as the bridge, your own Windows app

```
iPhone  <== Apple iMessage/SMS ==>  MacBook Pro (BlueBubbles server)
                                        |
                                   Cloudflare Tunnel
                                        |
                                  Your Windows app
```

- BlueBubbles server is open source. It reads the Messages database (Full Disk Access), sends through Messages automation, and exposes a documented REST API, Socket.IO realtime events, and webhooks. Cloudflare proxy is built in, so no port forwarding and it works from any network.
- The API is already used by third-party clients (WearBubbles on Wear OS, bluebubbles-gui in Go, a community TypeScript SDK), so building your own Windows client on it is a known-shape project, not research.
- Your app handles: chat list (including groups), history with paging, attachments, send, realtime incoming messages, tapbacks, Windows notifications, QR pairing, and staying logged in forever.
- Costs to know about:
  - The MacBook has to stay on and awake (power adapter + never-sleep-while-plugged-in setting; it can run lid-closed).
  - Server setup asks for a Google sign-in (Firebase) for notifications and takes ~20 minutes.
  - Your Apple ID stays signed into Messages on the Mac.
- Worth knowing: BlueBubbles already ships a free official Windows app. Installing it is the 20-minute version of this whole project and the perfect reference app while yours is being built.

## Architecture B: fully your own stack

Your own macOS helper app (Swift) that reads `~/Library/Messages/chat.db` and sends via AppleScript, served over HTTPS through Cloudflare Tunnel, plus your Windows app. Same reach as A, but you rebuild and maintain what BlueBubbles already maintains across macOS updates. Only worth it as a phase 2 rewrite after A works.

## Architecture C: pure Bluetooth Windows bridge (the original idea)

A Windows port of the Tether/BlueFerry approach: MAP client + OBEX + RFCOMM + ANCS over the Windows Bluetooth APIs. Nothing open source exists for Windows today; all of it is Linux/BlueZ. Even finished, it fails the spec: no group chats, no attachments, no sent-folder sync, ~10 m range. Effort estimate is months of protocol work, and "pairs every time" is the hardest part: Microsoft's own implementation is famously flaky (a whole support page exists for it), and on Linux only Intel-chipset adapters manage the secure bonding ANCS needs, with Realtek and cheap dongles failing entirely. Not recommended.
