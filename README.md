# ESPHome Duepi with Level Sensor 🔥⛽

An ESPHome configuration for smart pellet stoves using the **Duepi EVO / Artel** controller boards, combined with an integrated **HC-SR04** ultrasonic sensor to monitor the pellet fuel level in real-time. 

This project runs on a powerful **ESP32 WROOM 32 (USB-C)** and merges the excellent core communication components from @mvroosmalen1970 with fuel level calculations inspired by SmartHomeScene.

---

## 🛠️ Hardware Requirements

* **Microcontroller:** NodeMCU ESP32 WROOM 32 (with CH340 USB-C chip).
* **Level Sensor:** HC-SR04 Ultrasonic Distance Sensor (3.3V compatible version / RCWL-9610 chipset).
* **Pellet Stove:** Supported Artel / Duepi EVO controller based pellet stove.

---

## 🔌 Wiring & Pinout

Because we are using an ESP32, the UART communication to the stove is moved to hardware serial pinnen `GPIO16` and `GPIO17`. This prevents conflicts with the CH340 USB chip while flashing or testing via USB-C.

### HC-SR04 Ultrasonic Level Sensor

| Sensor Pin | ESP32 Pin | Description |
| :--- | :--- | :--- |
| **VCC** | **3V3** | Power supply (Safe 3.3V for this specific sensor module) |
| **TRIG** | **GPIO23 (G23)** | Trigger Pulse |
| **ECHO** | **GPIO22 (G22)** | Echo Return |
| **GND** | **GND** | Ground Reference |

### Duepi Pellet Stove Motherboard

| Duepi Pin | ESP32 Pin | Description |
| :--- | :--- | :--- |
| **+5V** | **5V / VIN** | Powers the ESP32 directly from the stove |
| **GND** | **GND** | Ground Reference |
| **RX** | **GPIO17 (TX2)** | ESP Transmit -> Stove Receive |
| **TX** | **GPIO16 (RX2)** | Stove Transmit -> ESP Receive |

---

## 🚀 Getting Started & Calibration

## 🚀 Getting Started, Calibration & External Temperature Sensor

### 1. General Setup & Wi-Fi
1. Clone or copy the `pelletstove.yaml` configuration into your ESPHome dashboard.
2. Change the Wi-Fi credentials to match your home network.
3. Install/Flash the code onto your ESP32.

### 2. Level Sensor Calibration
Open the ESPHome node's **Logs** to calibrate the pellet hopper depth:
* **Empty Hopper:** Look at the distance measured when empty (e.g., `0.45 m`). Update `- 0.45 -> 0` in your `pelletstove.yaml` config.
* **Full Hopper:** Look at the distance measured when full (e.g., `0.10 m`). Update `- 0.10 -> 100` in your `pelletstove.yaml` config.
* Re-flash the ESP32 to apply your exact calibration.

### 3. Smart External Temperature Sensor (with Automatic Failover) 🌡️🛡️
By default, this configuration can use a wireless room temperature sensor from Home Assistant (e.g., Zigbee, BLE, or Wi-Fi) to control the stove's climate thermostat instead of the hot sensor behind the stove. 

To keep your configuration completely **update-proof**, you can change this setting at the very top of your `pelletstove.yaml` file under the `substitutions:` block:

```yaml
substitutions:
  # Replace with the exact entity ID of your external Home Assistant sensor:
  externe_ha_temperatuursensor: "sensor.woonkamer_temperatuur"
```

#### How the Automatic Backup works:
* **Normal Operation:** The ESP32 reads the live state of your external room sensor. The pellet stove will modulate perfectly based on the temperature where you placed your room sensor.
* **Emergency Failover:** If your Wi-Fi drops, Home Assistant crashes, or the external sensor's battery dies, the ESP32 will instantly detect the missing data (`NAN`). Within 10 seconds, it will gracefully switch back to the stove's internal ambient temperature sensor. Your house stays safe and warm, and the stove will never run wild!
* **Status Monitoring:** You can monitor which sensor is currently driving your stove via the auto-generated `sensor.actieve_temperatuurbron` entity in Home Assistant.


---

## 🤝 Credits
* Base Duepi components by [@mvroosmalen1970](https://github.com)
* Ultrasonic level logic inspired by [SmartHomeScene](https://smarthomescene.com)
