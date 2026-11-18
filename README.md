# CANSAT · PocketQube Ground Station

A browser-based ground station for our CanSat / PocketQube. It shows live data from an **ESP32** over USB: the satellite's 3D attitude, flight instruments, telemetry charts, the GNSS track and automatic flight-event detection. It is a single HTML file, so there is nothing to install.

**▶ Live site:** <https://ground-station-production-d82bdd1b.up.railway.app>

---

## Features

| | |
|---|---|
| **3D attitude viewer** | A model of a 1P PocketQube (50 × 50 × 50 mm) that follows the GY-91's roll, pitch and yaw live. It shows the internal PCB stack, the sensor axes and an X-ray mode. Yaw can come from the compass or from the integrated gyro. Includes tare, invert-axis controls and orbit/zoom. |
| **Flight instruments** | Artificial horizon and compass heading dials. |
| **Live telemetry charts** | Altitude (barometer vs GPS), vertical speed, acceleration, angular rate, attitude, heading, magnetic field, pressure and temperature. The time window is adjustable from 10 s to 5 min, with pause, hover values and toggleable lines. |
| **Readouts** | Altitude, max altitude, vertical speed, total g, pressure, temperature, field strength and ground speed. |
| **GNSS track** | Top-down path coloured by altitude, with latitude, longitude, satellite count, HDOP and distance from start. |
| **Flight events** | Automatically logs **launch**, **free fall**, **apogee**, **high-g shocks** (e.g. parachute opening) and **landing**. |
| **Serial console** | Raw ESP32 output, one-click calibration commands and a box for custom commands. |
| **Recording** | Record a flight and export it as **CSV** (download or copy to clipboard). |
| **Demo flight** | A simulated ascent → apogee → parachute descent → landing, so the dashboard can be tried without hardware. |

## Hardware

- **ESP32** flight computer
- **GY-91** (MPU9250 IMU + BMP280 barometer)
- **Foxeer M10Q** GNSS + compass module

## Quick start

1. Open the [live site](https://ground-station-production-d82bdd1b.up.railway.app) in **Chrome or Edge** on a computer. Web Serial is not supported in Firefox, Safari or on phones.
2. Plug in the ESP32 over USB and **close the Arduino Serial Monitor/Plotter**. Only one program can use the COM port at a time.
3. Click **Connect ESP32 (USB)** and choose the COM port.
4. Data starts streaming. Press **● Record** to capture a flight, then **Download CSV**.

No hardware? Press **Demo flight**.

You can also open `index.html` directly from your computer in Chrome or Edge.

## Firmware data format

Serial at **115200 baud**. Each reading is one line of comma-separated `Name:value` pairs:

```
Rel_alt_m:12.31,VSpeed_mps:1.20,Ax_g:0.012,Ay_g:-0.004,Az_g:0.998,Atotal_g:0.998,Roll_deg:2.1,Pitch_deg:7.0,Heading_deg:292.4,Lat:30.630012,Lon:76.725034,Sats:11
```

Lines with fewer than three valid pairs are shown in the console as plain text. Field names the dashboard understands:

| Group | Fields |
|---|---|
| Altitude | `Rel_alt_m`, `GPS_rel_alt_m`, `VSpeed_mps`, `GPS_alt_m` |
| Pressure / temp | `Pressure_hPa`, `Press_change_hPa`, `BMP_temp_C`, `IMU_temp_C` |
| Accelerometer | `Ax_g`, `Ay_g`, `Az_g`, `Atotal_g` |
| Gyroscope | `Gx_dps`, `Gy_dps`, `Gz_dps` |
| Attitude | `Roll_deg`, `Pitch_deg`, `Heading_deg` |
| Magnetometer | `Mx_uT`, `My_uT`, `Mz_uT`, `Field_uT` |
| GNSS | `Lat`, `Lon`, `Sats`, `HDOP`, `Speed_kmph` |

### Commands sent to the ESP32

| Button | Sends | Purpose |
|---|---|---|
| Graph mode | `p` | Machine-readable output (sent automatically on connect) |
| Text mode | `t` | Human-readable output |
| Zero baro | `z` | Set the current altitude as 0 m |
| Gyro cal | `g` | Gyroscope calibration |
| Compass cal | `c` | Compass calibration |
| Accel cal | `a` | Accelerometer calibration |

## Hosting

The site is a static page. The included `Dockerfile` serves it with Python's built-in web server and is used for the Railway deployment:

```bash
docker build -t ground-station .
docker run -p 8080:8080 ground-station
```

Any static host with **HTTPS** works too, such as GitHub Pages or Netlify. Web Serial only runs on secure (HTTPS or localhost) pages.

## Files

```
index.html   the whole ground station (HTML + CSS + JS)
Dockerfile   tiny static server for deployment
```

Uses [three.js](https://threejs.org/) r128 for 3D and Google Fonts (Inter, JetBrains Mono). These load from a CDN, so the 3D view needs an internet connection.
