# Wanderung

Browser control page for a "RunnerT" walking pad (Runner 2.0), sold with the
Artiwares treadmill app. It uses **Web Bluetooth**, so there is nothing to
install: open the page, click **Connect**, pick the pad.

**Live page:** https://dasbasti.github.io/wanderung/

## Features

- Live speed, time, distance and energy
- Start, pause, stop and speed control (1.0–6.0 km/h)
- Distance tally (today, week, month, all time), 7-day chart and session list,
  stored in the browser, with JSON export/import

## Browser support

Chromium-based browsers (Chrome, Chromium, Edge). Firefox and Safari do not
support Web Bluetooth. On Linux, enable
`chrome://flags/#enable-experimental-web-platform-features` or start the
browser with `--enable-features=WebBluetooth`.

The pad accepts one Bluetooth connection at a time, so close the vendor app
first.

## Protocol

Reverse-engineered from the vendor app ("protocol version 1"):

| Characteristic | Use |
|---|---|
| `e54eaa55-371b-476c-99a3-74d267e3edae` | write `0x88` after connecting, or the pad disconnects |
| `e54eaa56-…` | notify: 12-byte status frames, every byte encoded |
| `e54eaa57-…` | write: 7-byte commands |

- Decode each notification byte: `(((b - 37) ^ 2) - 38) & 0xFF`
- Status frame (byte 11 = 0): status, speed (0.1 km/h), duration (s, 2 bytes),
  distance (m, 2 bytes), energy (2 bytes), heart rate, slope, target speed
- Command `[cmd, p1, p2, 0, 0, c1, c2]` with `c1 = ((cmd + p1) ^ p2) + 37`,
  `c2 = (p1 ^ cmd) + p2 + 2`. Commands: 1 start, 2 stop, 3 speed (p1 = 0.1 km/h),
  4 slope, 5 pause, 8 device info

Use at your own risk: this controls a moving belt.
