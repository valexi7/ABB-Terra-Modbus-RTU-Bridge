# Modbus RTU Bridge for ABB Terra AC Charger
**Powered by ESPHome on Lolin32 Lite (ESP32)**  
Bridges Home Assistant energy data (HomeWizard P1) into Modbus RTU for the ABB Terra AC EV3 charger.

---

## 🧩 Overview
This firmware turns an ESP32 (Lolin32 Lite) into a **Modbus RTU slave** that emulates the ABB EV3 electricity meter interface.  
It exposes voltage, current, power, energy, and power factor registers compatible with the ABB Terra AC charger’s expectations.  
Values are pulled live from Home Assistant sensors (e.g. HomeWizard P1 smart meter).

The goal: allow the ABB Terra AC charger to operate in **Smart Charging / Dynamic Load Balancing mode** without an actual ABB EV3 meter, using P1 data instead.
In a typical installation, the electricity meter is located in the **main distribution board**, while the EV charger is installed in a **garage** or **outbuilding**.  
Running a long Modbus RS-485 cable between them would be impractical.  
With this project, the live phase data is transmitted wirelessly via Wi-Fi from Home Assistant, and the ESP32 converts it locally into a Modbus RTU signal inside the charger — eliminating the need for any long communication wiring.

---
## 📡 System Architecture

The diagram below illustrates how data flows wirelessly from the **HomeWizard P1** smart meter through **Home Assistant** to the **ESP32 Modbus RTU bridge**, which then provides a wired **RS-485** Modbus connection to the **ABB Terra AC** charger.

![HomeWizard P1 to ABB Terra AC](/media/HomeWizard%20P1%20to%20ABB%20Terra%20AC.png)

1. **HomeWizard P1** → sends live power and energy data to Home Assistant via Wi-Fi.  
2. **Home Assistant** → serves those values over ESPHome API.  
3. **ESP32 bridge (Lolin32 Lite)** → emulates an ABB EV3 meter on Modbus RTU.  
4. **ABB Terra AC charger** → receives those values via RS-485 and performs smart charging.

---

## 🛠️ Hardware

| Component | Purpose | Notes |
|------------|----------|-------|
| **Lolin32 Lite** | ESP32 controller running ESPHome | Powered from charger’s 3.3 V rail (or VIN 5 V) |
| **RS-485 transceiver** | Modbus RTU physical layer | Auto-flow type (MAX13487 / SP3485) |
| **ABB Terra AC charger** | Modbus RTU master | Polls voltage/current/power registers |
| **Home Assistant + HomeWizard P1** | Data source | Provides real energy and phase data via API |

### ⚡ Hardware Wiring

| Lolin32 Lite pin | RS-485 module | Description | ABB Terra AC connection |
|------------------|---------------|--------------|--------------------------|
| **GPIO17 (TX)** | TXD | Modbus TX | Plug A, **slot 1 (TX)** |
| **GPIO16 (RX)** | RXD | Modbus RX | Plug A, **slot 2 (RX)** |
| **3V3** | VCC | Power | JTAG connector **3V3** pin |
| **GND** | GND | Common ground | JTAG connector **GND** pin |

> 💡 The ABB Terra AC charger exposes its internal Modbus interface on **plug A** (pins 1 = TX, 2 = RX).  
> The ESP32 is powered directly from the charger’s **3.3 V and GND** available on the **JTAG header**, eliminating the need for an external power supply.
### 🧰 Visual Wiring Diagram

![Wiring diagram for Lolin32 Lite and RS-485 module](/media/wiring-diagram.png)

The ESP32 runs as **Modbus server (slave)** on **address 0x01**, 9600 baud, 8E1.

---
