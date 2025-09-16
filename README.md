# <font color="cyan">EEE 5201: IoT Systems and Embedded Computing </font>
## <ins> <center> Smart Room Safety Monitor. </center> <ins></ins>

**Estimated Duration:** ~2 hours  (including setup and testing) 

**Group Size:** 6-7 students, with exemption for 2 students if absolutely necessary.
(*The Groups Might shrink for this Lab since we might have more than enough kits and sensors, but the reports will be submitted as usual.*)

**Platform:** Arduino IoT Cloud (https://create.arduino.cc/iot)(Blynk or other platforms can be used with modifications) <[Link To Blynk Platform](https://blynk.io/)>

---
---

## **Overview**
---

In our previous sessions, most groups have been developing irrigation monitoring and control systems using soil moisture sensors, learning the fundamentals of sensor-based automation and threshold control. One group explored XBee wireless communication, implementing Zigbee mesh networking protocols.
Since many groups are still finalizing their irrigation projects, we'll begin by completing those systems to ensure everyone has solid experience with sensor-to-actuator control loops and IoT cloud integration.
Once the irrigation projects are wrapped up, our XBee group will demonstrate their Zigbee mesh network to the class, explaining how wireless sensor nodes communicate and when mesh networking offers advantages over WiFi-based IoT solutions.
We'll then transition to today's main project: **The Smart Room Safety Monitor.** This system builds directly on your irrigation project experience, reading sensors, implementing decision logic, and controlling outputs, but scales up to handle multiple simultaneous inputs with coordinated intelligent responses.

The beauty of this project lies in its practical relevance. The same principles we'll implement today, multi-sensor data fusion, threshold-based decision making, and automated responses, are the backbone of real-world IoT applications from smart buildings to environmental monitoring stations.

---

## **Objectives**

By the end of this lab session, you will have:

1. **Integrated Multiple Sensors:** We will attempt to combine gas, sound, light, and vibration sensors into a unified monitoring system
2. **Implement Intelligent Logic:** We shall also create rule-based decision making that processes multiple inputs simultaneously.
3. **Build Automated Responses:** We will design a system that takes appropriate actions based on sensor data
4. **Developed IoT Dashboard:** We have done this already. That is, Created a real-time monitoring interface using Arduino IoT Cloud, As well as Blynk.
5. **Test Real-World Scenarios:** Simulated various safety conditions and verified system responses.
 
---

## **System Overview**

Our Smart Room Safety Monitor operates as an intelligent guardian for any enclosed space. It continuously monitors four key environmental parameters:

- **Air Quality** - Detects gas leaks, smoke, or chemical vapors
- **Noise Level** - Monitors for unusual sound patterns that might indicate problems
- **Light Conditions** - Tracks illumination for context-aware decision making
- **Vibration/Movement** - Detects unexpected physical disturbances

Based on these inputs, the system classifies the room's safety status into three levels:
- **SAFE** - All parameters normal, gentle green glow.
- **CAUTION** - One or more parameters elevated, yellow warning.
- **DANGER** - Critical thresholds exceeded, red alert with audible alarm.

---

## **Hardware Requirements**

**Per Group:**
- 1× ESP32 Development Board
- 1× Gas/Smoke Detection Module (MQ-2 or similar)
- 1× Sound Level Sensor Module
- 1× LDR (Light Dependent Resistor) 
- 1× Shock/Vibration Sensor
- 1× RGB LED (Common Cathode)
- 1× Active Buzzer
- 1× Relay Module (5V)
- 3× 220Ω Resistors (for RGB LED)
- Breadboard and jumper wires

---

## **Circuit Connections** (ESP32 DevKit V1 Module Pinout Reference) 
*Keep in mind that some pins are input-only (34-39) and others have special functions. Refer to the ESP32 datasheet for details. Also, avoid using pins 6-11 as they are connected to the flash memory. Thus, Make sure to use GPIO pins that do not conflict with the onboard functions of the ESP32.*

The following table outlines the recommended pin connections for each component: (Remember to adjust based on your specific ESP32 board model, as pin numbering may vary) Then these are Just suggestions; feel free to modify based on your wiring convenience.

| Component | ESP32 Pin | Notes |
|-----------|-----------|--------|
| **Sensors** |
| Gas Sensor (Analog) | GPIO34 | Use ADC1 pins for WiFi compatibility |
| Sound Sensor (Analog) | GPIO35 | Adjust sensitivity potentiometer |
| LDR (Analog) | GPIO32 | Use voltage divider with 10kΩ resistor |
| Vibration Sensor (Digital) | GPIO27 | Pull-up resistor may be needed |
| **Outputs** |
| RGB LED - Red | GPIO25 | Through 220Ω resistor |
| RGB LED - Green | GPIO26 | Through 220Ω resistor |
| RGB LED - Blue | GPIO33 | Through 220Ω resistor |
| Buzzer | GPIO14 | Active buzzer, no external components |
| Relay Control | GPIO12 | Controls external load (fan/light) |

**Important Notes:**
- Use GPIO pins 34-39 for analog sensors (ADC1 pins work better with WiFi) as they are input-only.
- Ensure common ground between ESP32 and all sensors/actuators.
- Use appropriate resistors to limit current to the RGB LED.
- Double-check relay module voltage requirements (5V vs 3.3V logic), If at all you have to use the relay module.
- Ensure proper power supply for relay module
- Test each sensor individually before integration, but this is optional, as you may have done this in previous labs.

---

## **Arduino IoT Cloud Setup**

### **Step 1: Create Your Thing**
1. Navigate to Arduino IoT Cloud and create a new Thing
2. Name it: `RoomSafetyMonitor_GroupX` (replace X with your group number)
3. Associate your ESP32 device with the Thing
4. Set up WiFi credentials in `arduino_secrets.h` file, if not done already.
We have done this before so you should be more than familiar with the process.

### **Step 2: Configure Cloud Variables** 
*Remember: Arduino IoT Cloud free tier allows maximum 5 variables Blynk on the other hand might allow more depending on the plan.* 
(*Also, for the permission types, They can vary based on the widgets you choose to have on your dashboard.*)

| Variable Name | Type | Permission | Purpose |
|---------------|------|------------|---------|
| `room_status` | String | Read Only | Current safety level ("SAFE"/"CAUTION"/"DANGER") |
| `gas_level` | int | Read Only | Gas sensor reading (0-100%) |
| `noise_level` | int | Read Only | Sound level (0-100%) |
| `light_status` | String | Read Only | Light condition ("BRIGHT"/"DIM"/"DARK") |
| `alert_count` | int | Read Only | Daily alert counter |

### **Step 3: Build Your Dashboard**
Create these widgets in your dashboard(Remember: You can customize the layout as you see fit, Creativity is highly encouraged and welcomed!):
- **Status Card** → `room_status` (with color coding)
- **Gauge Widget** → `gas_level` (0-100 range, red zone >70)
- **Gauge Widget** → `noise_level` (0-100 range, yellow zone >60)  
- **LED Indicator** → `light_status` (Green=BRIGHT, Yellow=DIM, Red=DARK)
- **Counter Widget** → `alert_count`
(Also,If you are using Blynk, you can create similar widgets there, just ensure they map to the same variables and types and be creative with the layout.)
---

## **Programming Implementation**

### **Core Functions You Need to Implement:**
>(Some of these functions may be partially implemented in the starter code provided, Just to act as a guide as I have seen some great deviation in your previous implementations. Your task is to complete and integrate them into the main loop. Feel free to modify the function signatures as needed., but ensure clarity and maintainability as well as proper commenting for each function. Remember to try and undertand the logic behind each function.)

**1. Sensor Reading Functions**
```cpp
int readGasLevel() {
    // Read analog value from gas sensor
    // Map to 0-100 percentage
    // Apply calibration if needed
}

int readNoiseLevel() {
    // Read sound sensor analog value  
    // Convert to meaningful scale
    // Consider ambient noise baseline
}

String getLightStatus() {
    // Read LDR value
    // Classify as BRIGHT/DIM/DARK
    // Use appropriate thresholds
}

bool detectVibration() {
    // Read vibration sensor
    // Implement debouncing if needed
    // Return true/false
}
```

**2. Safety Assessment Logic**
```cpp
String evaluateRoomSafety() {
    // Your decision-making algorithm goes here
    // Consider multiple sensor inputs
    // Implement threshold-based logic
    // Return "SAFE", "CAUTION", or "DANGER"
}
```

**3. Response System**
```cpp
void handleSafetyResponse(String status) {
    // Control RGB LED based on status
    // Manage buzzer activation
    // Control relay for external devices
    // Update alert counter when needed
}

void setRGBColor(int red, int green, int blue) {
    // Control RGB LED colors
    // Use PWM for smooth color transitions
}
```

### **Programming Tips To Consider:**
- Use `analogRead()` for sensor values, but remember ESP32 ADC returns 0-4095
- Map sensor readings to meaningful ranges using `map()` function
- Implement proper delays to avoid overwhelming the cloud service
- Use `Serial.println()` for debugging during development
- Consider sensor warm-up time, especially for gas sensors, and also include this in your report
- Ensure your main loop is efficient and non-blocking to maintain responsiveness

---

## **Suggested Implementation Phases**

### **Phase  1: Basic Sensor Testing And Schematics Generation**
1. Wire and test each sensor individually, This is optional as you may have done this in previous labs, you can go ahead and integrate them directly.
2. Display raw sensor values on Serial Monitor
3. Determine appropriate threshold values for your environment
4. Draw a clear circuit schematic (hand-drawn or software like Wokwi, Fritzing, etc.)

### **Phase 2: Logic Implementation.**
1. Combine all sensors into main reading function
2. Implement your safety assessment algorithm
3. Create response system for different safety levels
4. Test logic with manual sensor triggering (cover gas sensor, make noise, etc.)

### **Phase 3: IoT Integration**
1. Configure Arduino IoT Cloud variables and dashboard
2. Integrate cloud updates into your main loop
3. Test real-time dashboard updates
4. Verify bidirectional communication (if using control variables)

### **Phase 4: System Testing**
1. Run comprehensive system tests with all sensors active.
2. Document threshold values and response behaviors.
3. Create demonstration scenarios.
4. Fine-tune system parameters for reliability.

---

## **Testing Scenarios**

Demonstrate your system with but not limited to these test cases:

**Test 1: Gas Detection** (These are just suggestions, feel free to come up with your own test cases, again, creativity is highly encouraged and welcomed!)
- Cover gas sensor or introduce safe test gas
- Expected: Status → DANGER, Red LED, Buzzer ON, Relay activated

**Test 2: Noise Alert** 
- Create loud noise (clapping, talking)
- Expected: Status → CAUTION (or DANGER if very loud), Yellow/Red LED

**Test 3: Light Conditions**
- Cover LDR sensor, use flashlight
- Expected: Light status changes appropriately

**Test 4: Vibration Detection**
- Gently shake the setup
- Expected: Status may change to CAUTION depending on other sensors

**Test 5: Combined Alerts**
- Trigger multiple sensors simultaneously
- Expected: System responds to highest priority alert

---

## **Deliverables**

At the end of the lab session, each group must present:

### **1. Working Demonstration**
- System responds correctly to the test scenarios it has been tested against
- Clear indication of room status via RGB LED and buzzer
- Dashboard updates in real-time
- All sensors and actuators functioning properly, if not, provide justification.

### **2. Technical Documentation**
**Am elaborate Lab Report including:**
- Circuit diagram with pin assignments
- Threshold values used and justification
- Decision logic flowchart
- Screenshot of working dashboard and the circuit setup as it were during setup and testing. (It's always encouraged to take pictures during the lab session, for diffrent empirical observations
- Code snippets of key functions with explanations
- Challenges faced and how they were overcome
- Suggestions for future improvements or extensions), some of these will be implemented in your lab report.
- Complete Arduino sketch with comments
- Group member contributions (State who did what part of the setup, coding, testing, documentation, etc.)

>*Consider starting with a compelling introduction of your setup, including the problem statement, objectives, and any relevant research or prior work and then top it all up with some practical case studies related to this setup, It always makes the report more engaging and informative as well as fun to assess.*
---
>|

#

## **Safety Reminders**

 **Important Notes:**
- Just follow Lab Protocols such as; Working in a well-ventilated area,
- Avoid inhaling fumes from gas sensors,
- Do not use flammable gases for testing,
- Keep liquids away from electronic components,
- Disconnect power when making wiring changes
- Handle components with care
- Report any equipment issues immediately

---
>## ***Additional Resources***
>Refer to this short Video incase you want to intergrate your Project with Blynk.
[![How to set up the new Blynk app with an ESP32 board | ESP32 projects](https://img.youtube.com/vi/W1xG_XJb0FU/0.jpg)](https://www.youtube.com/watch?v=W1xG_XJb0FU)

>**A Kind Reminder:** Beyond making your setup to work, be sure to understand *WHY* it works. So I would that you to focus on the logic behind your threshold choices and the reasoning behind your system responses.



&copy; 2025 SCES, EEE Department, Strathmore University. All rights reserved.
