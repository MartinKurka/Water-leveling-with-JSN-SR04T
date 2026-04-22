# Water-leveling-with-JSN-SR04T
ESPHome configuration for measuring water level in a tank using a JSN-SR04T ultrasonic sensor mounted inside a 50mm pipe. Configurable via built-in web server — no reflashing needed.


# 💧 ESPHome Water Level Sensor

ESPHome configuration for ultrasonic water level measurement using **JSN-SR04T** sensor mounted inside a 50mm pipe. Designed for tanks like IBC containers, wells, or any reservoir.

---

## Hardware

| Component | Description |
|---|---|
| ESP8266 D1 Mini Lite | Microcontroller |
| JSN-SR04T | Waterproof ultrasonic distance sensor |
| 50mm PVC pipe | Sensor housing (with foam absorber mod) |
| Foam/rubber tubing | Acoustic absorber to narrow beam angle |

### Wiring

| JSN-SR04T | D1 Mini Lite |
|---|---|
| VCC | 5V |
| GND | GND |
| TRIG | GPIO4 |
| ECHO | GPIO5 |

---

## How It Works

The sensor is mounted at the top of a pipe that goes down into the tank. It measures the distance to the water surface. The firmware converts this distance into water height and fill percentage.

```
[Sensor]
    |
    | ← pipe length (cfg)
    |
    |~~~~  ← water surface
    |
    |
   [bottom of tank]
    ↕ max tank level (cfg)
```

**Formula:**
```
water height = pipe length - measured distance + offset
fill % = water height / max tank level × 100
```

---

## The 50mm Pipe Problem & Solution

JSN-SR04T has a ~15° beam angle. Inside a 50mm pipe, the ultrasonic beam reflects off the pipe walls and causes false readings.

**Solution: printed holder with foam absorber**

A 3D printed holder centers the sensor inside the pipe. Foam is placed only around the sensor itself inside the holder — it absorbs the side lobes of the beam right at the source. The pipe itself remains empty.

> No software changes needed — purely a hardware fix.

---

## User Configuration (via Web Server)

All key parameters are adjustable at runtime through the built-in web server — no reflashing needed.

| Setting | Unit | Default | Description |
|---|---|---|---|
| Pipe length | cm | 200 | Physical distance from sensor to tank bottom |
| Max tank level | cm | 80 | Maximum water height in the tank |
| Offset | cm | 0 | Fine-tuning correction (+ or −) |
| Low level threshold | % | 20 | Trigger level for low water warning |

Values are stored in flash memory and survive power cycles.

---

## Sensors & Entities

| Entity | Type | Description |
|---|---|---|
| Water level - distance | Sensor (m) | Raw distance from sensor to water surface |
| Water level - height | Sensor (cm) | Calculated water height from tank bottom |
| Tank percentage | Sensor (%) | Fill level as percentage |
| Low water level | Binary sensor | `ON` when fill % drops below threshold |

---

## Installation

1. Copy `tankwaterlevel.yaml` to your ESPHome config directory
2. Add credentials to `secrets.yaml`:
```yaml
wifi_ssid_1: "your_ssid"
wifi_pass_1: "your_password"
wifi_ssid_2: "your_ssid_2"
wifi_pass_2: "your_password_2"
wifi_ssid_3: "your_ssid_3"
wifi_pass_3: "your_password_3"
wifi_ssid_4: "your_ssid_4"
wifi_pass_4: "your_password_4"
```
3. Flash the device:
```bash
esphome run tankwaterlevel.yaml
```
4. Open the web server at `http://tankwaterlevel.local` and set your parameters

---

## Calibration

1. Set **Pipe length** to the actual distance in cm from the sensor face to the tank bottom
2. Set **Max tank level** to the maximum possible water height in cm
3. If the reading is off, adjust **Offset** (e.g. `-3` if it reads 3 cm too high)
4. Set **Low level threshold** to the % at which you want to be alerted

---

## Notes

- Blind zone: JSN-SR04T cannot reliably measure distances below ~22 cm. If the tank is nearly full and the water surface is within 22 cm of the sensor, readings may be unreliable.
- The median filter (window size 5) smooths out reflections from water surface ripples. This introduces a ~10s delay in readings.
- `include_internal: true` is required in `web_server` for the settings input boxes to appear.

---

## Requirements

- [ESPHome](https://esphome.io) 2024.x or newer
- Home Assistant (optional — works standalone via web server)
