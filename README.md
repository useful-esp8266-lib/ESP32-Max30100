
# ESP32 with MAX30100 (Pulse Oximeter and Heart Rate Sensor)

This project demonstrates how to interface an **ESP32** microcontroller with the **MAX30100** sensor to measure heart rate and blood oxygen saturation (SpO2). The MAX30100 integrates a pulse oximeter and a heart rate monitor and communicates with the ESP32 via I2C.

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Wiring](#wiring)
- [Software Requirements](#software-requirements)
- [Installation](#installation)
- [Code Example](#code-example)
- [Usage](#usage)
- [License](#license)

## Hardware Requirements

- **ESP32** development board
- **MAX30100** Pulse Oximeter and Heart Rate Sensor module
- Jumper wires
- Breadboard (optional)

## Wiring

Connect the **MAX30100** to the **ESP32** using the following connections:

| MAX30100 Pin | ESP32 Pin    | Description        |
|--------------|--------------|--------------------|
| **VCC**      | 3.3V or 5V   | Power supply       |
| **GND**      | GND          | Ground             |
| **SCL**      | GPIO 22      | I2C Clock (SCL)    |
| **SDA**      | GPIO 21      | I2C Data (SDA)     |
| **INT**      | Optional     | Interrupt (optional)|

**Note:** The MAX30100 module should work with both 3.3V and 5V depending on the specific model. Check your sensor module specifications before connecting power.

## Software Requirements

- **Arduino IDE** with ESP32 board support (Follow [this guide](https://github.com/espressif/arduino-esp32#installation-instructions) to install ESP32 in the Arduino IDE).
- **MAX30100 library** by oxullo (can be installed via Arduino Library Manager).

## Installation

### 1. Install the ESP32 Board Support
   - Open **Arduino IDE**.
   - Go to **File > Preferences**.
   - In the "Additional Boards Manager URLs" field, add:
     ```
     https://dl.espressif.com/dl/package_esp32_index.json
     ```
   - Go to **Tools > Board > Boards Manager**, search for "ESP32" and install the ESP32 board package.

### 2. Install the MAX30100 Library
   - Go to **Sketch > Include Library > Manage Libraries**.
   - Search for **MAX30100 by oxullo**.
   - Click **Install**.

### 3. Code Example
Use the following code to interface with the MAX30100 sensor and display heart rate and SpO2 readings on the serial monitor.

```cpp
#include <Wire.h>
#include "MAX30100_PulseOximeter.h"

// Define the I2C pins (default GPIO 21 for SDA and GPIO 22 for SCL)
#define REPORTING_PERIOD_MS     1000

PulseOximeter pox;
uint32_t tsLastReport = 0;

// Callback routine for when a pulse is detected
void onPulseDetected()
{
    Serial.println("Pulse detected!");
}

void setup()
{
    Serial.begin(115200);

    // Initialize I2C communication
    Wire.begin(21, 22);

    Serial.print("Initializing Pulse Oximeter...");

    // Initialize the Pulse Oximeter
    if (!pox.begin()) {
        Serial.println("FAILED");
        for(;;);
    } else {
        Serial.println("SUCCESS");
    }

    // Set the callback for when a pulse is detected
    pox.setOnBeatDetectedCallback(onPulseDetected);
}

void loop()
{
    // Update the sensor readings
    pox.update();

    // Report the heart rate and SpO2 readings every second
    if (millis() - tsLastReport > REPORTING_PERIOD_MS) {
        Serial.print("Heart rate:");
        Serial.print(pox.getHeartRate());
        Serial.print(" bpm / SpO2:");
        Serial.print(pox.getSpO2());
        Serial.println(" %");

        tsLastReport = millis();
    }
}
