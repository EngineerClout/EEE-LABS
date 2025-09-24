# Raspberry Pi 400 Introduction Lab


### Welcome to Your First Raspberry Pi 400 Experience

In embedded systems and IoT, you've worked with various microcontrollers and development boards. Today, we introduce you to something different: the Raspberry Pi 400, a complete Linux computer built into a keyboard.

## A Brief History & What Makes Pi 400 Special

The Raspberry Pi Foundation launched the original Pi in 2012 to make computing accessible worldwide. The Pi 400, released in 2020, represents a unique approach: instead of a separate board requiring peripherals, it integrates a Raspberry Pi 4 directly into a keyboard.

**Key Differences from Traditional Development Kits:**
- Complete desktop computer in a keyboard form factor
- Full Linux OS (Raspberry Pi OS) with desktop environment
- Dual 4K display support via micro-HDMI
- Gigabit Ethernet and dual-band WiFi built-in
- Full 40-pin GPIO header for hardware interfacing
- Pre-installed development tools (Python, Thonny IDE)

Unlike Arduino or bare-metal microcontroller boards, the Pi 400 runs a full operating system, making it ideal for IoT applications requiring networking, file systems, and complex processing.


[![Watch the video](https://img.youtube.com/vi/-JuNcc2nrX8/hqdefault.jpg)](https://youtu.be/-JuNcc2nrX8)

---

![The Octocat](https://raw.githubusercontent.com/EngineerClout/GRAPHMODEL_SCREENSHOTS/refs/heads/main/RPI-400-US_main.jpg)


## Lab Activity 1: Unboxing and Initial Setup

**Step 1: Examine Your Kit**
On your desk, you should find:
- Raspberry Pi 400 keyboard
- Power supply (USB-C, 5V/3A)
- Micro-HDMI to HDMI cable
- MicroSD card (32GB with Raspberry Pi OS)
- GPIO breakout board and jumper wires
- LED and 220Ω resistor

![The Octocat](https://raw.githubusercontent.com/EngineerClout/GRAPHMODEL_SCREENSHOTS/refs/heads/main/RPI400_PowerButton_a.jpg)

**Step 2: Physical Connections**
1. Insert the microSD card into the slot on the underside of the Pi 400
2. Connect the HDMI cable to the first micro-HDMI port (closest to power)
3. Connect USB mouse to any USB port
4. Connect Ethernet cable (if available) or prepare for WiFi setup
5. **Connect power last** - the Pi will boot automatically

![The Octocat](https://raw.githubusercontent.com/EngineerClout/GRAPHMODEL_SCREENSHOTS/refs/heads/main/RPI400_System.jpg)

![The Octocat](https://raw.githubusercontent.com/EngineerClout/GRAPHMODEL_SCREENSHOTS/refs/heads/main/RPi400vs4BNov2020.png)

---

## Lab Activity 2: Remote Access via VNC

For more details on VNC and remote access, refer to the official documentation: [Remote Access](https://www.raspberrypi.com/documentation/computers/remote-access.html)
![The Octocat](https://www.raspberrypi.com/documentation/computers/remote-access.html)

While your Pi boots, we'll set up remote access for easier development.

**Enable VNC on Your Pi 400:**
1. Wait for the desktop to load completely
2. Click the Pi logo menu → Preferences → Raspberry Pi Configuration
3. Go to the "Interfaces" tab
4. Enable VNC and SSH
5. Click OK and reboot when prompted

**Connect via VNC from Your Laptop:**
1. Note the Pi's IP address: Menu → Accessories → Terminal, then type:
   ```bash
   hostname -I
   ```
2. On your laptop, download VNC Viewer (free from RealVNC)
3. Connect using the IP address, username: `pi`, default password: `raspberry`

**Security Note:** Change the default password immediately:
```bash
passwd
```
Install RealVNC
![The Octocat](https://picockpit.com/raspberry-pi/wp-content/uploads/2023/10/image-63.png)



This requires opening a terminal and typing in:

```bash 
sudo raspi-config
```

Then the following screen will pop up:

![The Octocat](https://picockpit.com/raspberry-pi/wp-content/uploads/2023/10/softconfig.jpg)

Raspberry Pi Software Configuration
Then you need to scroll down to Advanced Options, because you need to switch over to X11.
![Th](https://picockpit.com/raspberry-pi/wp-content/uploads/2023/10/x11.jpg)

Switch from Wayland to X11
And then you’ll need to hit <finish> and reboot your Raspberry Pi.

Now all you need to do is exactly what you do on Bullseye OS: applications menu -> preferences -> raspberry pi configuration -> interfaces and then click VNC.
![The Octocat](https://picockpit.com/raspberry-pi/wp-content/uploads/2023/10/raspberrypiconfi.jpg)

Configuring VNC on Bookworm OS
That will enable RealVNC automatically. And then you’re set:

RealVNC on Bookworm OS
Now you just need the RealVNC viewer on your main computer and you should have no problem remotely accessing your Raspberry Pi Bookworm OS desktop!


---

## Lab Activity 3: GPIO LED Blink Experiment

Now for the hands-on embedded part. We'll create a simple LED blink program to demonstrate GPIO control.

**Hardware Setup:**
1. Locate the 40-pin GPIO header on the back of the Pi 400
2. Wire your circuit:
   - GPIO Pin 26 (BCM 7) → 220Ω Resistor → LED Long Leg (Anode)
   - LED Short Leg (Cathode) → Ground Pin 6
3. Double-check connections before proceeding

**Software Development:**

**Step 1: Open Terminal** (This is crucial - run ALL commands here)
Click the terminal icon in the taskbar or Menu → Accessories → Terminal

**Step 2: Create Project Directory**
```bash
mkdir ~/gpio_lab
cd ~/gpio_lab
```

**Step 3: Create Virtual Environment** (Run these commands exactly in terminal)
```bash
python3 -m venv venv
source venv/bin/activate
```
You should see `(venv)` appear in your terminal prompt.

**Step 4: Install Required Libraries** (Still in terminal with venv active)
```bash
pip install --upgrade pip
pip install RPi.GPIO
```

**Step 5: Create the LED Blink Script**
```bash
nano led_blink.py
```

Copy and paste this code into nano:
```python
#!/usr/bin/env python3
import time
import RPi.GPIO as GPIO

LED_PIN = 7  # BCM 7 = Physical Pin 26

def main():
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(LED_PIN, GPIO.OUT)
    
    try:
        print("LED Blinking on GPIO 7 (Pin 26). Press Ctrl+C to stop.")
        counter = 0
        while True:
            GPIO.output(LED_PIN, GPIO.HIGH)
            print(f"LED ON - Blink {counter + 1}")
            time.sleep(0.8)
            
            GPIO.output(LED_PIN, GPIO.LOW)
            print(f"LED OFF - Blink {counter + 1}")
            time.sleep(0.8)
            counter += 1
            
    except KeyboardInterrupt:
        print("\nProgram stopped by user")
    finally:
        GPIO.cleanup()
        print("GPIO cleanup completed")

if __name__ == "__main__":
    main()
```

Save and exit nano: `Ctrl+X`, then `Y`, then `Enter`

**Step 6: Run Your First Pi 400 GPIO Program**
```bash
python3 led_blink.py
```

**Expected Result:** Your LED should blink every 0.8 seconds with corresponding terminal messages. Press `Ctrl+C` to stop.

---

## Testing Alternative Development Environments

**Option A: Using Thonny IDE**
1. Menu → Programming → Thonny Python IDE
2. Tools → Options → Interpreter
3. Select "Alternative Python 3 interpreter"
4. Browse to: `/home/pi/gpio_lab/venv/bin/python3`
5. Open your `led_blink.py` file and click Run

**Option B: Using the System Python** (If venv causes issues)
```bash
# Deactivate venv first
deactivate
# Install system-wide (not recommended for projects, but works for testing)
sudo apt install python3-rpi.gpio
python3 led_blink.py
```

---

## Common Issues and Solutions

**Problem:** `ModuleNotFoundError: No module named 'RPi'`
**Solution:** Ensure you're in the virtual environment (`venv` should show in prompt) and RPi.GPIO is installed there.

**Problem:** Permission denied accessing GPIO
**Solution:** The `pi` user should have GPIO access by default. Avoid using `sudo` with GUI applications.

**Problem:** LED doesn't light up
**Solution:** Check wiring polarity (long leg to resistor/GPIO, short leg to ground) and ensure resistor is in series.

---

## Key Takeaways
The Pi 400 bridges traditional embedded programming with full computer capabilities, making it ideal for IoT gateway applications, edge computing, and rapid prototyping.

**Next Steps:**
- Experiment with different blink patterns
- Try controlling multiple LEDs
- Explore the sensor libraries available in the Raspberry Pi ecosystem
---
---

# YOUR TASK FOR TODAY
---

For this lab, you will need to connect a DHT11 temperature and humidity sensor to your Pi 400. The DHT11 is a widely used sensor that communicates over I2C.

1. Connect the sensor to the I2C bus of your Pi 400.
2. Write a Python script that reads the temperature and humidity from the sensor every second.
3. Print the readings to the console.
4. Plot the temperature and humidity readings over time.
5. Save your script as `dht11_sensor.py`.
6. Run the script using the command `python3 dht11_sensor.py`.
7. The script should print the temperature and humidity readings to the console and plot the data using matplotlib.

Note: Make sure to install the necessary libraries (`sudo apt install python3-rpi.gpio python3-smbus`) before running the script.

---

## Cleanup and Shutdown

Before leaving:
1. Stop your Python program (`Ctrl+C`)
2. Safely shutdown: Menu → Shutdown → Shutdown
3. Disconnect power and pack your kit as it was carefully
4. Submit your lab report

**Important:** Always shutdown properly to prevent SD card corruption.