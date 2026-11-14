# Fork notes & roadmap (kewe87)

This fork adds security hardening, robustness fixes, and an optional hardware safety control to
[ESP32 iPhone Agent](https://github.com/onehumanbeing/esp32-iphone-agent) by Henry Yuan (MIT).
All upstream copyright and the LICENSE are kept, as required by the MIT License and `AGENTS.md`.

## ✅ Implemented in this fork (verified to build)

| Change | File(s) | Verified |
|---|---|---|
| **AirPlay/RTSP + bplist overflow fixes** (pre-auth OOB-read DoS) | `src/airplay/rtsp.c`, `src/bplist/bplist.c` | firmware compiles (arduino-cli, esp32 3.3.12) |
| **Windows BLE scan fix** (#114: empty `devices[]` on WinRT) | `cli/internal/ble/ble.go` | `go build` + `go vet` clean |
| **Daemon panic guard** (malformed `io` request crashed the whole daemon) | `cli/internal/daemon/ops.go`, `server.go` | `go build` + `go vet` clean |
| **Decoder.err data race** fix | `cli/internal/video/decoder.go` | clean |
| **Empty-line panic guard** | `cli/internal/daemon/conn.go` | clean |
| **Optional stop button + status LED** (`--safety`, default off) | `hardware/hid_bridge/hid_bridge.ino`, `flash.ps1` | compiles with and without the flag; **needs on-hardware test** |
| **bplist fuzz + regression harness** (libFuzzer + portable repro) | `hardware/tests/fuzz/` | builds & runs under gcc; libFuzzer+ASan in CI |
| **CI** (firmware compile, `go build`/`vet`/`test`, bplist fuzz) | `.github/workflows/fork-ci.yml` | YAML added |
| **Single-client SoftAP hardening** (`--harden`, default off) | `netcfg.cpp`, `flash.sh`, `docs/hardening.md` | firmware compiles |
| **OLED status display** (`--display`, SSD1306 I2C via U8g2) | `hid_bridge.ino`, `docs/hardware-addons.md` | compiles; **needs the OLED wired** |
| **Battery gauge** (`-DBATTERY_PIN`, on the OLED) | `hid_bridge.ino`, `docs/hardware-addons.md` | compiles; **needs a LiPo + divider** |
| **BLE discovery boost** (short name in the primary advertisement + faster interval, for WinRT #114) | `ble_link.cpp` | compiles; **needs on-hardware test** |

## 🔒 Security disclosure (please read)

The two firmware overflow fixes close a **remote DoS** reachable by any unauthenticated device
on the board's SoftAP (RTSP TCP 7000): a crafted `Content-Length` / binary-plist wraps the size
checks and drives the parser off the buffer (OOB read → panic → reboot). **No OOB write / RCE**
was found. Per upstream `SECURITY.md`, consider reporting this to the maintainer **privately**
(GitHub → Security → "Report a vulnerability") before any public PR.

## 🟡 Deferred from the audit (lower severity — tracked, not yet fixed)

Fixing these well needs runtime testing; left as follow-ups to avoid risky untested concurrency
patches:

- **Goroutine leak**: `daemon/listen.go` `transcribe` — `h.work` is never closed on Conn teardown.
- **Pointer state not serialized** per board across concurrent requests (`internal/pointer`).
- **`wifi.go fallBack`** lacks the `claim()/release()` guard that `up()` has (narrow race).
- **`ble/tcp.go`** keepalive goroutine lingers up to one tick after read-side teardown (tidy-up).

## 💡 Proposed features (need the iOS app / hardware; design only so far)

- **Secure-field typing guard** — refuse to type when a password/secure field is detected (OCR), fail-closed.
- **App allowlist** — only let the agent open/act in approved apps.
- **Action audit log** — persist what the agent did (the board already has a RAM log ring).
- **Benchy-style verification** — check the real end-state to catch "false success" (idea from ihandclaw).
- **Android dual-mode** — one "hands" firmware drivable by both hosts (see `docs/android.md`).

---
Based on [ESP32 iPhone Agent](https://github.com/onehumanbeing/esp32-iphone-agent) by Henry Yuan.
