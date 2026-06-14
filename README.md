<!-- This file is the template for the PUBLIC firmware repo's README. The release
     workflow substitutes 0.6.0 and appends CHANGELOG.md, then pushes the
     result to hanstornquist/ModBus-TCP-firmware. Edit it here, not there. -->
# ModBus TCP Gateway — firmware distribution

Released firmware for an **ESP32-S2 (LOLIN/Wemos S2 Mini) transparent Modbus
TCP ↔ Modbus RTU gateway**. This repository holds **only the compiled binary and a
version manifest** — the gateways read them to self-update. **Source code is
private; nothing here is buildable source.**

The gateway exposes a Modbus TCP server on port 502 and forwards every request
verbatim onto an RS-485 (Modbus RTU) bus, relaying each device's own reply — no
device map, register mirror, or polling. It is built for Node-RED reading e.g.
SDM630 power meters.

## Current release: **v0.6.0**

| File | Purpose |
| --- | --- |
| [`version.json`](version.json) | Manifest the gateway polls (`version`, `code`, `bin`) |
| [`ModBus_TCP_Gateway.bin`](ModBus_TCP_Gateway.bin) | Compiled firmware image |
| [`flash/`](flash/) | Browser-based flasher (full-flash image + ESP Web Tools page) |

## Flash a board from your browser

To program a **blank or factory-reset board**, open the **[web flasher](flash/)**
in Chrome or Edge on a desktop, connect the board over USB, and click Install.
It writes the full firmware image over Web Serial — no tools to download.

## How updates reach a gateway

Each gateway checks `version.json` once a day (and on demand from its web portal's
**Check for updates now** button). If the manifest's `code` is newer than the
running build, it downloads `ModBus_TCP_Gateway.bin` over HTTPS and flashes it. A
freshly flashed image must prove itself healthy or the gateway **automatically
rolls back** to the previous firmware, so a bad release can't strand a device.

Releases are produced automatically by CI from the private source repository on a
version tag — this repo (binary, manifest, and this README) is fully generated.

---

# Changelog

All notable changes to the ModBus TCP Gateway firmware. This file is the single
source of truth; the release Action publishes it into the public firmware repo's
README on every tagged release.

## v0.6.0 — branded UI, signed-only updates, web flasher
- **Redesigned web portal:** a dark, card-based interface matching the SkillShot
  look — the logotype in the header (and as the browser tab icon), a connection
  status pill, an orange brand accent, and a footer. Mobile-friendly; fully
  self-contained so the setup-AP portal still works with no internet.
- **Signed-only updates by default:** the manual (unsigned) firmware-upload
  endpoint is now gated behind a build flag and is **off in released builds**, so
  a unit accepts only cryptographically signed over-the-air updates — an admin
  password alone can no longer flash arbitrary firmware.
- **Browser-based flasher:** program a blank or factory-reset board over USB
  straight from Chrome/Edge (Web Serial), no tools to install — published
  alongside each release.
- Internal: the firmware source was split from one large file into themed modules
  (no behaviour change); fixed a stray redirect to `0.0.0.0` from unknown-URL
  probes (e.g. the browser's favicon request) once the gateway is on Wi-Fi.

## v0.5.3 — setup UX & factory aids
- Setup-AP mode prints the device password in the serial heartbeat so the factory
  can reliably read it for the unit's label (the one-time boot line can be lost in
  the USB-CDC startup burst).
- Shorter unique password (8 chars, the WPA2 minimum) — easier to type on a phone;
  still excludes look-alike characters.
- Wi-Fi scan list: de-duplicates SSIDs, skips hidden networks, URL-encodes the
  pick-a-network links (fixes broken/blank result clicks), and caps the list.
- Setup (AP) portal is now a minimal page (Wi-Fi form + scan only); the full
  portal renders once connected. Avoids render failures right after a scan.

## v0.5.2 — Wi-Fi robustness
- Network scanning now runs **only in setup (AP) mode**; on the single-radio S2 a
  scan while connected could wedge the radio and drop the unit off the network.
- **Wi-Fi self-heal:** if a previously-working link stays down past a threshold
  (3 min), the gateway hard-resets (RTC) to recover a wedged radio instead of
  retrying forever — no power-cycle/site visit needed.

## v0.5.0 — security & reliability hardening
- **Signed updates:** the device verifies an ECDSA-P256 signature over each pulled
  image against a baked-in public key before flashing; CI signs every release with
  the private key. A tampered binary or a compromised public repo/token is rejected.
- **Unique per-device password:** public builds generate a random admin password on
  first boot (logged once over serial for labelling) instead of a universal default;
  it gates the web portal, OTA push, and the setup-AP Wi-Fi.
- **Authenticated portal:** config/update endpoints require the admin password once
  the device is on Wi-Fi (the setup AP stays open, gated by its unique Wi-Fi password).
- **Reliable self-update:** a found update is downloaded early on the next boot (fresh
  heap) instead of after uptime, fixing TLS/OTA allocation failures.
- **Hardware watchdog:** a hung main loop now auto-reboots.

## v0.4.1
- Device secrets (Wi-Fi credentials, OTA-push password) are seeded into NVS on a
  secrets build and read from NVS thereafter, so devices keep working across
  public self-update builds. OTA-push password is also settable from the portal.

## v0.4.0
- Self-update: the gateway checks a published `version.json` once a day (and via a
  portal **Check for updates now** button) and pulls the new firmware over HTTPS,
  guarded by the automatic rollback.

## v0.3.x
- Self-healing OTA: a new image must prove itself healthy (Wi-Fi connected) within
  a window, or the gateway automatically rolls back to the previous firmware; the
  portal shows a banner naming the reverted version.
- Diagnostics on the portal home page and `/diag`: RTU round-trip and end-to-end
  service time as 30-sample moving averages, OK/timeout/CRC counters, last bad frame.

## v0.2.x
- Transparent Modbus TCP ↔ RTU bridge: forwards any unit id / function code /
  register and relays the device's own reply (no device map or register mirror).
- Reliable RS-485 handling: direct bus I/O with frame-sync (absorbs the TX→RX
  turnaround glitch byte), request queue for pipelined clients, and retries.
- Wi-Fi setup portal (NVS), configurable RTU baud/parity, authenticated HTTP OTA.
