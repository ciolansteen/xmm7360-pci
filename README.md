⚠️ _*In heavy development. No support provided. May not work, may crash your computer, may singe your jaffles.*_ ⚠️

# Getting started

![CI](https://github.com/xmm7360/xmm7360-pci/workflows/CI/badge.svg)

## What

Driver for Fibocom L850-GL / Intel XMM7360 (PCI ID 8086:7360).

Please see [DEVICES.md](DEVICES.md) a list of devices this has been tested on.

## How

Please see [INSTALLING.md](INSTALLING.md) for details on how to setup this driver on your system.

### Dependencies

- build-essential
- python3-pyroute2
- python3-configargparse

## Status

This release supports native IP.

To test:

- `sudo pip install --user pyroute2 ConfigArgParse`
- `make && make load`
- If your sim has pin enabled, run `echo "AT+CPIN=\"0000\"" | sudo tee -a /dev/ttyXMM1`. Replace `0000` with your pin code.
- `sudo python3 rpc/open_xdatachannel.py --apn your.apn.here` (or you can create the xmm7360.ini from the sample and edit the apn)
- pray (if applicable)

> If your sim has pin enabled, run `echo "AT+CPIN=\"0000\"" | sudo tee -a /dev/ttyXMM1`. Replace `0000` with your pin code.

You should receive a `wwan0` interface, with an IP, and a default route.

## Next

Involvement from someone involved in modem control projects like ModemManager
would be welcome to shape the kernel interfaces so it's not too horrible to
bring up.

Power management support is absent. The modem, as configured, turns off during
suspend, and needs to be reconfigured on resume.

---

## Roadmap

This section tracks known missing features and community-identified work items,
compiled from upstream [issues](https://github.com/xmm7360/xmm7360-pci/issues)
and [pull requests](https://github.com/xmm7360/xmm7360-pci/pulls).

### ✅ Recently Fixed (this fork)

| Item | Upstream issue | Notes |
|------|---------------|-------|
| ASN.1 BER NULL type `0x5` crash in `rpc.py` | [#122](https://github.com/xmm7360/xmm7360-pci/issues/122) by [@Yetiicom](https://github.com/Yetiicom) | Affects devices receiving SIM Toolkit / STK callbacks |

### 🔴 High Priority

| Item | Upstream ref | Notes |
|------|-------------|-------|
| **Suspend/Resume support** | [#1](https://github.com/xmm7360/xmm7360-pci/issues/1) — original author [@abrasive](https://github.com/abrasive) | Modem loses state on suspend; requires PM callbacks in kernel module |
| **Memory leak on extended use** | [#131](https://github.com/xmm7360/xmm7360-pci/issues/131) by [@ge0rg](https://github.com/ge0rg), [#178](https://github.com/xmm7360/xmm7360-pci/issues/178) by [@Jiivee1](https://github.com/Jiivee1) | 34 GB+ kernel allocation after ~2 weeks; skbuff_head_cache grows unbounded |
| **Kernel 6.6+ / 6.11+ compat** | [PR #244](https://github.com/xmm7360/xmm7360-pci/pull/244) by [@rabreu](https://github.com/rabreu), [PR #220](https://github.com/xmm7360/xmm7360-pci/pull/220) by [@mfournier](https://github.com/mfournier) | `hrtimer_setup` API change (6.11+), `ssize_t`/`size_t` mismatch (6.6+) |
| **FCC unlock default hash** | [#240](https://github.com/xmm7360/xmm7360-pci/issues/240) by [@discreteds](https://github.com/discreteds) | Hardcoded hash fails on factory-reset modems; should default to all-zeros |

### 🟡 Medium Priority

| Item | Upstream ref | Notes |
|------|-------------|-------|
| **GPS / GNSS support** | [#135](https://github.com/xmm7360/xmm7360-pci/issues/135), [#228](https://github.com/xmm7360/xmm7360-pci/issues/228) | Hardware supports GPS+GLONASS+BeiDou+Galileo; RPC call IDs exist (`0x136`–`0x145`); payload format unknown — needs Windows driver capture |
| **Signal strength reporting** | [#63](https://github.com/xmm7360/xmm7360-pci/issues/63) by [@zhdanow5a](https://github.com/zhdanow5a) | `AT+CSQ` returns `99,99` (unknown); need proper RSSI/RSRP reporting |
| **APN authentication (CHAP/PAP)** | [#62](https://github.com/xmm7360/xmm7360-pci/issues/62) | Username/password APNs not supported |
| **TX/RX statistics** | [PR #165](https://github.com/xmm7360/xmm7360-pci/pull/165) by [@hschauhan](https://github.com/hschauhan) | `net_device_stats` for visibility in system monitors |
| **Auto-unload `iosm` module** | [PR #201](https://github.com/xmm7360/xmm7360-pci/pull/201) by [@RealTehreal](https://github.com/RealTehreal), [#200](https://github.com/xmm7360/xmm7360-pci/issues/200) | Conflicts with in-tree `iosm` driver on kernel 5.18+ |

### 🔵 Long-term

| Item | Upstream ref | Notes |
|------|-------------|-------|
| **eSIM / eUICC support** | [#31](https://github.com/xmm7360/xmm7360-pci/issues/31) | Hardware supports GSMA SGP.22; requires full MBIM stack + LPA implementation |
| **Mainline kernel submission** | [#104](https://github.com/xmm7360/xmm7360-pci/issues/104) by [@jan-kiszka](https://github.com/jan-kiszka) | 41 checkpatch warnings to resolve before upstream submission |
| **Wake-on-WAN / SMS wakeup** | [#166](https://github.com/xmm7360/xmm7360-pci/issues/166) | Requires suspend/resume first |

### ✅ Already implemented (upstream or ecosystem)

| Item | Notes |
|------|-------|
| **ModemManager 1.26+ native support** | Implemented by [@tuxor1337](https://github.com/tuxor1337) — see [MM MR !1200](https://gitlab.freedesktop.org/mobile-broadband/ModemManager/-/merge_requests/1200) |
| **DKMS support** | Merged upstream |
| **Vanilla kernel 6.2+ RPC path** | `/dev/wwan0xmmrpc0` — documented by [@Mixaill](https://github.com/Mixaill) in [#211](https://github.com/xmm7360/xmm7360-pci/issues/211) |

---

## Related Projects

| Project | Author | Description |
|---------|--------|-------------|
| [xmm7360-usb-modeswitch](https://github.com/xmm7360/xmm7360-usb-modeswitch) | xmm7360 org | USB mode tools |
| [xmm7360_usb](https://github.com/juhovh/xmm7360_usb) | [@juhovh](https://github.com/juhovh) | USB mode kernel module |
| [xmmctl](https://github.com/mlukow/xmmctl) | [@mlukow](https://github.com/mlukow) | C rewrite of the Python RPC tool |
| [xmm7360-deamon](https://github.com/binghamfluid/xmm7360-deamon) | [@binghamfluid](https://github.com/binghamfluid) | Daemon for automated modem startup |
