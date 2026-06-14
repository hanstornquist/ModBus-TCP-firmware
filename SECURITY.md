# Security policy

This document is the published security policy for the **ModBus TCP Gateway**
(ESP32-S2 Modbus TCP↔RTU gateway). It covers how to report vulnerabilities and how
long the product receives security updates. It is written to satisfy the relevant
parts of the **UK PSTI Act 2022** and to align with the **EU RED cybersecurity
requirements (Art. 3.3 d/e/f)** and the **EU Cyber Resilience Act (CRA)**.

Manufacturer: **SkillShot AB**
Policy contact: **security@skillshot.se**
Last updated: 2026-06-14

---

## 1. Reporting a vulnerability

If you believe you have found a security vulnerability in the gateway firmware,
hardware, or its update service, please report it to:

- **Email:** security@skillshot.se
- Please include: affected firmware version (shown on the device portal and in
  `version.json`), a description, steps to reproduce, and any proof-of-concept.
- If you need to send sensitive details, ask us for a secure channel in your first
  message.

**What you can expect from us:**

| Step | Target |
| --- | --- |
| Acknowledge receipt of your report | within **3 business days** |
| Initial assessment + triage (severity, validity) | within **10 business days** |
| Status updates while we work on a fix | at least every **30 days** |
| Fix released (or mitigation/decision) for valid issues | typically within **90 days**, sooner for actively exploited or critical issues |

We will keep you informed and, unless you prefer otherwise, credit you when the
fix is published.

## 2. Scope

In scope: the gateway firmware, the over-the-air update mechanism and its
distribution (`version.json`, signed `.bin`/`.sig`), the on-device web portal, and
the Modbus TCP/RTU handling.

Out of scope: the customer's own Wi-Fi network/router, third-party Modbus devices
(e.g. power meters) attached to the bus, and physical attacks requiring sustained
hardware access beyond what is described in our threat model.

## 3. Coordinated disclosure

Please give us a reasonable opportunity to remediate before public disclosure
(we target the timelines above). We will not pursue legal action against good-faith
research that respects this policy, avoids privacy violations and service
disruption, and does not access data beyond what is needed to demonstrate the
issue. Do not run tests against devices you do not own or operate.

## 4. Security update period (defined support period)

We will provide security updates for each product model for a **minimum of 5 years
from the date that model is last made available for sale**. For the current model
this is guaranteed **until at least 2031-06-14**; if the model continues to be sold,
the end-of-support date extends to 5 years after its final sale.

> The published copy of this policy (in the firmware distribution repo) has this
> "until at least" date **auto-stamped to the release date + 5 years on every
> release**, so the public guarantee always stays at least 5 years ahead of the
> latest firmware. The date in this source copy is only a baseline.

- Security updates are delivered **over-the-air**: the gateway checks daily for a
  newer signed firmware release and installs it automatically, with automatic
  rollback if an update is unhealthy. Updates can also be triggered from the device
  portal ("Check for updates now").
- Updates are **cryptographically signed**; the device verifies the signature before
  installing, so only firmware released by us can be applied.
- This support-period statement is made available to the buyer **before purchase**
  (product listing / packaging) and is published here.

## 5. Reference: product security features

- No universal default passwords — each unit ships with a **unique** admin password
  (on the label); it gates the web portal, OTA, and the setup Wi-Fi.
- **Signed firmware updates** with on-device signature verification.
- **Automatic rollback** of a failed/unhealthy update.
- Web portal/configuration requires authentication once the device is on the network.
- Secrets (Wi-Fi credentials, admin password) are stored on-device; see the roadmap
  for flash-encryption at rest (`documentation/secure-boot-flash-encryption.md`).
