# Optional hardware add-ons

All off by default; each needs the part physically wired. The firmware support is built in — enable
it with a build flag. Needs on-hardware validation.

## Status OLED (128×64 SSD1306, I2C) — `--display`
A ~2 € 0.96" SSD1306 OLED shows board name + firmware, net mode + IP, peer/BLE count, and
**RUNNING / STOPPED** (ties into the `--safety` stop button).

Wiring (XIAO ESP32-S3 default I2C pins):

| OLED | XIAO |
|------|------|
| VCC  | 3V3  |
| GND  | GND  |
| SDA  | D4 (GPIO5) |
| SCL  | D5 (GPIO6) |

Build: `flash.sh --display` / `flash.ps1 -Display`. Uses the U8g2 library (installed automatically in
CI; locally run `arduino-cli lib install U8g2` once — already done on this machine).

## Battery gauge (LiPo) — `-DBATTERY_PIN=<adc pin>`
Reads the LiPo voltage through a divider on an ADC pin and shows a rough % on the OLED.

- Wire `BAT+ → R → ADC pin → R → GND` (two equal resistors = a 2.0 divider; keep the pin ≤ 3.3 V).
- Build e.g. `flash.ps1 -Display -BatteryPin 1` (GPIO1 / A0); set `-DBATTERY_DIVIDER=<ratio>` if not 2.0.
- The % curve is a rough 3.3 V = 0 % … 4.2 V = 100 %; calibrate for your cell.
- The XIAO ESP32-S3 charges a LiPo on its BAT pads, so board + phone can run portable.

## Case
Etsy has XIAO ESP32-S3 cases (plain, USB-C end open). With an OLED/LiPo, pick a case variant with an
OLED window / battery bay, or print one.
