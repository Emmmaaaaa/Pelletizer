Opensource Pelletizer — Pellets 15
---
## 📖 Project Description
 
This project is an open-source filament pelletizer designed to recycle 3D printing waste back into usable feedstock for screw-based 3D printers. The machine takes standard 1.75mm (or 2.85mm) plastic filament from a spool, feeds it through a precision stepper-driven extruder, and chops it into consistent ~3mm pellets using a high-speed rotary blade — all synchronized by an ESP32 microcontroller.
 
The goal is to close the loop on 3D printing plastic waste in a low-cost, reproducible, and open-source way. The entire housing is FDM 3D printed in PETG, and the total build cost is approximately **$233.62 CAD**.

![Outline of finalized design](https://media.discordapp.net/attachments/1257901389582172223/1518386515305042000/image.png?ex=6a39bb23&is=6a3869a3&hm=583dffc3bbe136aeb5d3428026d4349e332f045c8d91530427892bb592990a4c&=&format=webp&quality=lossless&width=960&height=371)
 
### Key Features
 
- **Precision feed control** — A stepper motor drives a filament extruder ("puller roller") at a tunable speed, directly determining pellet length
- **High-speed rotary cutting** — A DC motor spins dual utility blades; filament is guided through a throat/funnel to ensure clean shear cuts
- **ESP32 synchronization** — The microcontroller modulates the ratio between feed speed and cutting frequency to consistently hit the 3mm target
- **Web interface** — A browser-based dashboard (hosted on the ESP32) lets the user control both motors remotely, adjust speeds, reverse directions to clear jams, and trigger an emergency stop
- **Hardware E-Stop** — A physical latching button and internal limit switch cut all power to the motors instantly
- **Vacuum dust extraction** — An integrated vacuum pulls fines and incomplete cuts into a secondary canister, keeping output pellets clean
- **Modular design** — The machine disassembles quickly for cleaning when switching between materials (e.g., PLA → PETG)
- **Slot-in collection** — Standardized guide rails accept Tupperware-style containers to catch finished pellets
---
 
## Code Overview
 
The firmware runs on an **ESP32 Dev Board**.

![Electrical Schematic illustrating the interconnection between the ESP32 microcontroller and dual-motor driver stages](https://media.discordapp.net/attachments/1257901389582172223/1518386574625341510/image.png?ex=6a39bb31&is=6a3869b1&hm=64cf844ad62e64477deed3ec48768c89b84b04bbb5db8e1e85a725e9816630a5&=&format=webp&quality=lossless&width=983&height=420
)
 
### What the code does
 
1. **Hosts a Wi-Fi web server** — The ESP32 creates a web server that serves the control dashboard HTML page to any browser on the same network. The user connects by entering the ESP32's IP address in the settings panel of the app.
2. **Controls the stepper motor (L298N driver)** — The stepper drives the filament extruder. Speed is set via a `stepDelay_us` value (microsecond delay between steps). The web interface maps the "Speed" slider directly to this value. Direction (forward/backward) is controlled by the `stepperDir` boolean, allowing the operator to reverse the extruder to clear jams.
3. **Controls the DC chopper motor (BTS7960 driver)** — The high-speed rotary cutter is driven via PWM (`ledcWrite`). The "Power" slider in the DC Motor panel maps 0–100% to the PWM range. The `dcMotorReverse()` function lets the operator reverse blade direction if needed.
4. **Handles the Emergency Stop** — The E-Stop is hardware-mapped; the limit switch (wired normally-closed) is monitored by the ESP32. When triggered, the `stepperOff()` and `dcMotorStop()` helper functions are called immediately, cutting power to both motors. The web interface also sends an `'x'` / `'h'` command string to `/command` on the ESP32 for a software-side global stop.
5. **Processes HTTP POST commands** — The `/command` endpoint receives single-character command strings from the web dashboard:
   - `'h'` — halt stepper
   - `'x'` — stop DC motor
   - Forward/backward/speed commands for each motor
6. **Safety interlock** — The limit switch inside the housing acts as a secondary hardware interlock. When the blade guard is removed or the switch is triggered, the ESP cuts motor power regardless of software state.
---
 
## 🔌 Hardware
 
| Component | Description |
|---|---|
| ESP32-DEVKITC-32E | Main microcontroller / web server |
| L298N | Stepper motor driver (extruder) |
| BTS7960 | H-bridge DC motor driver (chopper) |
| Stepper Motor | Drives the filament extruder/puller |
| High-Speed DC Motor | Drives the rotary blade |
| Limit Switch (NC) | Hardware interlock on blade assembly |
| Power Supply (12V) | Powers both motor drivers |

---
 
## 🚀 Getting Started
 
### Prerequisites
 
- [Arduino IDE](https://www.arduino.cc/en/software) or PlatformIO
- ESP32 board package installed ([installation guide](https://docs.espressif.com/projects/arduino-esp32/en/latest/installing.html))
### Upload Instructions
 
1. Clone this repo
2. Open the main `.ino` file in Arduino IDE
3. Select board: **ESP32 Dev Module**
4. Set your Wi-Fi SSID and password in the config section at the top of the sketch
5. Connect the ESP32 via micro-USB and upload
6. After uploading, power the ESP32 from a 5V wall adapter (micro-USB)
7. Open the Serial Monitor to find the ESP32's assigned IP address
8. Enter that IP in the Settings cog on the web dashboard to establish connection

---
 
## ⚙️ Tuning Tips
 
- To get consistent **3mm pellets**, adjust the ratio between the stepper speed and DC motor power using the sliders
- **Faster extruder / slower chopper** → longer pellets
- **Slower extruder / faster chopper** → shorter pellets
- Use the built-in material presets for PLA, PETG, and TPU as a starting point
- Enable the vacuum system before starting a run to keep fines out of the final output
---
 
## 🛡️ Safety
 
- Always ensure the **E-Stop is disengaged** (off position) before starting motors
- Never reach into the cutting zone while motors are powered
- The blade guard must be in place during operation
- Turn off all motors and unplug before performing any maintenance or blade changes
---
 
