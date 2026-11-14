# Running the board safely

The board drives a real phone and runs a few network services. It's fine for personal/lab use;
these steps reduce exposure to anyone within Wi-Fi / Bluetooth range.

## Threat model
When the board hosts its own Wi-Fi SoftAP, the AirPlay/RTSP receiver (TCP 7000) and the video/OTA
ports are reachable by **any device that joins that hotspot**. A security review found the parsers
to be memory-safe for reads after the overflow fixes in this fork (see `ROADMAP-FORK.md`), but a
hostile device that can join the AP — or is already on your LAN in station mode — can still poke
those services.

## Recommended settings
- **Keep the SoftAP password strong.** The firmware generates random AP credentials; don't replace
  them with something weak, and don't share them.
- **Single-client AP (default in this fork).** Only one station — your phone — can associate at a
  time (`AP_MAX_CLIENTS=1`). This also stops a computer from joining the board's hotspot next to the
  phone: the CLI then stays on Bluetooth, or use `esp32agent net join` for your home Wi-Fi. Build with
  `flash.sh --multi-client` (up to 4 stations) only when you need that, e.g. for the debug harness.
- **Prefer the SoftAP over station mode** in untrusted environments: on your home LAN (station mode)
  the services are reachable by everything on that network.
- **Stop button (default in this fork).** Press the board's BOOT button to suppress all HID output
  (LED solid = stopped, slow blink = running); press again to resume. Keep it within reach while an
  agent drives the phone. `flash.sh --no-safety` removes it.
- **Use a local or trusted vision model.** Screenshots of the phone go to whatever model endpoint
  you configure; a local model (Ollama / vLLM) keeps them on your own machine.
- **Physical custody.** Mirroring stops when the phone locks, and iOS hides passcode / Apple Pay /
  DRM screens, but the board is still a keyboard + mouse — unplug it when you're not using it.

## Only automate what you own
Drive only devices and accounts you own or are authorized to use.
