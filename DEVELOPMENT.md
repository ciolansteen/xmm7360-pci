# Development Guide

## Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Stable, tested releases |
| `dev`  | Active development — GPS, eSIM, new features |

All new work goes into `dev`. Merge to `main` only after testing on hardware.

## Environment

| Machine | Role |
|---------|------|
| `hommie` | Compilation, packaging (Arch Linux x86_64) |
| `ciolaptop` | Hardware testing (has Fibocom L850-GL / Intel XMM7360) |

## Workflow

### 1. Develop on hommie

```bash
# Clone / update
git clone ssh://git@ciolan.go.ro/linuxDrivers/xmm7360-pci.git
git checkout dev

# Make changes, build
make

# Commit to dev
git add -p
git commit -m "type(scope): description"
git push origin dev
```

### 2. Build package on hommie

The PKGBUILD at `archRepo/xmm7360-pci-dkms` picks up the latest commit via
`pkgver()`. To build from `dev` instead of `main`, temporarily edit the source
URL in PKGBUILD:

```bash
# In PKGBUILD, change:
source=("git+ssh://$url#branch=dev" ...)
```

Then build:
```bash
makepkg -si
# or push to trigger CI
```

### 3. Test on ciolaptop

```bash
# Install latest build
sudo pacman -Syu xmm7360-pci-dkms xmm7360-pci-utils

# Load module
sudo modprobe xmm7360

# Run service
sudo systemctl restart xmm7360.service
journalctl -fu xmm7360.service
```

### 4. AT command testing

The modem exposes two AT ports:

```bash
# Interactive AT session
sudo picocom -b 115200 /dev/wwan0at0

# Or one-shot
echo -e "AT+XGNSS=1\r" | sudo tee /dev/wwan0at0
```

## Commit Convention

`type(scope): description` — no trailing period, imperative mood.

Common types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`

Common scopes: `rpc`, `kernel`, `gps`, `esim`, `service`, `pkgbuild`

Examples:
```
feat(gps): implement UtaMsSsLcsInit RPC wrapper
fix(rpc): handle ASN.1 BER NULL type (0x05) in unpack_unknown
chore(pkgver): bump to r7.bbd8a3f
```

**No `Co-authored-by` trailers.**

## RPC Protocol Notes

RPC call IDs are in `rpc/rpc_call_ids.py`.
Unsolicited event callbacks are in `rpc/rpc_unsol_table.py`.

Relevant IDs for upcoming features:

### GPS / Location Services (LCS)
| Call ID | Name |
|---------|------|
| `0x136` | `UtaMsSsLcsInit` |
| `0x137` | `UtaMsSsLcsMoLocationReq` |
| `0x13C` | `UtaMsCpAssistanceDataInjectReq` |
| `0x140` | `UtaMsCpPosMeasurementReq` |
| `0x144` | `UtaMsCpPosEnableMeasurementReport` |

Callbacks:
| Code | Name |
|------|------|
| `0x138` | `UtaMsSsLcsMoLocationRspCb` |
| `0x143` | `UtaMsCpPosReportMeasurementIndCb` |

Payload formats are **unknown** — require reverse engineering
(Windows Dell GNSS driver capture or RPC fuzzing).

### eSIM / eUICC
Requires full MBIM stack — long-term goal.
