<!-- This file is the template for the PUBLIC firmware repo's README. The release
     workflow substitutes 0.5.1 and appends CHANGELOG.md, then pushes the
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

## Current release: **v0.5.1**

| File | Purpose |
| --- | --- |
| [`version.json`](version.json) | Manifest the gateway polls (`version`, `code`, `bin`) |
| [`ModBus_TCP_Gateway.bin`](ModBus_TCP_Gateway.bin) | Compiled firmware image |

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
