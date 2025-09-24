# Raspberry Pi 400 — Quick Start Guide (virtualenv, Python libraries, blink an LED with Thonny or VS Code)

**Introduction — Raspberry Pi 400**  
The Raspberry Pi 400 is a Raspberry Pi 4 built into a keyboard: a compact, accessible desktop platform for learning and projects. It exposes the full 40-pin GPIO header (same layout as Pi 4) so you can connect sensors, LEDs and other hardware and control them from Python. See the pin reference/layout here for a quick visual:  
https://www.theelectronics.co.in/2021/02/raspberry-pi-400-full-pc-in-keyboard.html

This guide shows a safe, tested set of steps to get your Pi 400 ready for GPIO Python development, create a virtual environment, install the correct libraries, and run a first script to blink an LED on **BCM 7 (physical pin 26)**. It includes only commands that work correctly for the scenario (no `--user` while venv is active, correct package names, etc.). It also shows how to run from **Thonny** (default on Raspberry Pi OS) or **VS Code**.

---

## Before you start — hardware safety & wiring

- **Never** connect an LED directly between a GPIO pin and GND without a resistor. Use a current-limiting resistor (220–330 Ω).
- Your wiring (as used in this guide):
  - **Physical pin 26** → **Resistor 220 Ω** → **LED anode (long leg)**  
  - **LED cathode (short leg)** → **Physical pin 6 (GND)**  
  - Physical pin 26 corresponds to **BCM GPIO 7**, so in Python we will use `LED_PIN = 7`.

Double-check orientation (long leg = anode) and connections before applying power.

---

## 0) System update and prerequisites

Open a terminal on the Pi and run:

```bash
sudo apt update
sudo apt upgrade -y

# Install tools that are useful for building Python packages (only needed if native-code packages must compile)
sudo apt install -y python3-pip python3-venv python3-dev build-essential libssl-dev swig
```

- `python3-venv` provides `python3 -m venv` (virtual environment support).
- Installing `swig` and `libssl-dev` helps if packages need native compilation.

---

## 1) Create project folder and virtual environment (recommended)

```bash
mkdir -p ~/Desktop/iotprojects
cd ~/Desktop/iotprojects

# create venv (one-time)
python3 -m venv venv

# activate the venv (every session where you work on the project)
source venv/bin/activate
# prompt will show: (venv) pi@raspberrypi:~/Desktop/iotprojects $
```

> **Important:** When the prompt shows `(venv)` you are inside the virtual environment. All `pip` installs now go into `venv` (do **not** use `--user` while venv is active).

---

## 2) Upgrade pip and install the required Python packages (inside the venv)

With the venv active:

```bash
python3 -m pip install --upgrade pip setuptools wheel

# Install packages required for this guide:
python3 -m pip install RPi.GPIO requests arduino-iot-cloud
```

**Notes**
- `RPi.GPIO` is the standard GPIO library for Raspberry Pi (works on Raspberry Pi OS).
  - If `RPi.GPIO` installation via pip fails for any reason, you can install the system package instead (see Troubleshooting).
- `requests` is included because it’s commonly useful in projects (network interaction).
- `arduino-iot-cloud` is optional here; this guide focuses on local GPIO blinking. (If you later use Arduino Cloud you can use their Python docs: https://docs.arduino.cc/arduino-cloud/guides/python/)

Verify the installs:

```bash
python3 -c "import RPi.GPIO, requests, arduino_iot_cloud; print('OK: RPi.GPIO, requests, arduino_iot_cloud')"
```

If that prints without errors, you’re ready.

---

## 3) Create the LED blink script

Create a file `pi_led_blink.py` in `~/Desktop/iotprojects` with this content:

```python
#!/usr/bin/env python3
# pi_led_blink.py — Blink LED on BCM GPIO7 (physical pin 26)

import time
import RPi.GPIO as GPIO

LED_PIN = 7  # BCM 7 = physical pin 26

def main():
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(LED_PIN, GPIO.OUT)

    try:
        print("Blinking LED on GPIO7 (pin 26). Press Ctrl+C to stop.")
        while True:
            GPIO.output(LED_PIN, GPIO.HIGH)
            time.sleep(0.5)
            GPIO.output(LED_PIN, GPIO.LOW)
            time.sleep(0.5)
    except KeyboardInterrupt:
        print("\nStopped by user")
    finally:
        GPIO.cleanup()

if __name__ == "__main__":
    main()
```

Make it executable (optional):

```bash
chmod +x pi_led_blink.py
```

---

## 4) Run the script from the terminal (venv must be active)

```bash
# ensure you're in the project folder and the venv is active
source ~/Desktop/iotprojects/venv/bin/activate

# run the blink script
python3 pi_led_blink.py
```

Expected: LED blinks ON/OFF every 0.5 seconds. Press `Ctrl+C` to stop. If you see `ModuleNotFoundError: No module named 'RPi'` or similar, see the Troubleshooting section below.

---

## 5) Run the script from **Thonny** (recommended for learners)

**Configure Thonny to use the venv Python (so installed packages are visible):**

1. Open **Thonny** (Menu → Programming → Thonny).
2. Tools → Options → Interpreter.
3. Under *Interpreter*, choose **"Alternative interpreter or virtual environment"**.
4. Browse to and select the venv Python executable:
   ```
   /home/pi/Desktop/iotprojects/venv/bin/python3
   ```
5. Click OK. (Restart Thonny if you are prompted.)
6. Open `pi_led_blink.py` in Thonny and press Run ▶.

Thonny’s shell will show the printed messages; the LED should blink.

---

## 6) Run the script from **VS Code** (optional)

1. Install **Python** extension in VS Code.
2. Open the folder `~/Desktop/iotprojects` in VS Code.
3. Select the Python interpreter (Ctrl+Shift+P → **Python: Select Interpreter**) → choose:
   ```
   /home/pi/Desktop/iotprojects/venv/bin/python3
   ```
4. Open `pi_led_blink.py` and run (F5) or click Run ▶.

---

## Troubleshooting & common fixes

**1. `ModuleNotFoundError: No module named 'RPi'`**  
- If you are inside the venv, install `RPi.GPIO` into the venv:

  ```bash
  # with venv active
  python3 -m pip install RPi.GPIO
  ```

- If pip install fails (rare on Pi OS), install the system package:

  ```bash
  sudo apt update
  sudo apt install python3-rpi.gpio
  ```

  Then ensure Thonny or VS Code uses the system interpreter, or use the system python to run the script.

**2. Thonny can't see the packages you installed**  
- Confirm Thonny is configured to use the same interpreter (venv) used for installation: Tools → Options → Interpreter → set to `/home/pi/.../venv/bin/python3`.

**3. Permission errors accessing GPIO**  
- Normally the `pi` user can use GPIO. If you get permission errors, avoid running GUI editors as `sudo`. As a temporary diagnostic only, you can try running the script with `sudo`, but better is to fix group permissions or run with the venv python as the `pi` user.

**4. If `arduino-iot-client` (or other SDK) fails to install**  
- `arduino-iot-client` can require native dependencies; the `arduino-iot-cloud` package is often lighter. For Arduino Cloud integration via Python see the official guide here (includes diagrams and examples):  
  https://docs.arduino.cc/arduino-cloud/guides/python/

---

## Quick-check commands

```bash
# inside your project and venv
which python3
python3 -V
python3 -m pip --version
python3 -m pip list

# test imports quickly
python3 -c "import RPi.GPIO as GPIO, requests; print('RPiGPIO OK, requests OK')"
```

---

## Further reading

- Pi 400 pin layout / reference: https://www.theelectronics.co.in/2021/02/raspberry-pi-400-full-pc-in-keyboard.html  
- Arduino IoT Cloud Python guide (if you later integrate cloud features): https://docs.arduino.cc/arduino-cloud/guides/python/

---

## Closing notes

- This guide focuses on a **minimal, reliable, reproducible path** to get a Pi 400 ready for GPIO work and to blink an LED using Python inside a venv, with instructions for both Thonny and VS Code.
- If you want, I can:
  - produce a printable PDF version, OR
  - add a small section that shows how to use the Arduino Cloud REST method to control the LED from the cloud.
