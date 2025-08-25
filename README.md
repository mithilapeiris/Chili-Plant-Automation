# 🌱 Smart Sprout – IoT Soil Monitoring System

## 📌 Project Overview
Smart Sprout is an **IoT-based soil and environment monitoring system** built using the **NodeMCU ESP8266** microcontroller.  
The system uses multiple sensors to measure **temperature, humidity, and soil moisture**, and applies **logic-based thresholds** to detect abnormal conditions.  
Currently, the project **reads data and prints alerts via Serial Monitor**. (Firebase/Cloud integration can be added later.)

---

## ⚡ Features
- Reads **temperature & humidity** using **DHT11 sensor**
- Reads **soil moisture percentage** using a **capacitive soil moisture sensor**
- Converts raw analog values into a percentage scale (0–100%)
- Checks values against **user-defined thresholds**
- Prints **sensor readings and warnings** on the Serial Monitor
- Modular code with **separate functions** for readability

---

## 🛠️ Hardware Components
1. **NodeMCU ESP8266** – WiFi-enabled microcontroller  
2. **DHT11 Sensor** – Measures temperature and humidity  
3. **Capacitive Soil Moisture Sensor** – Measures soil moisture level  
4. **Jumper wires**  
5. **USB cable** (for programming & powering NodeMCU)

---

## 🔌 Wiring Connections

### DHT11 Sensor
| DHT11 Pin | NodeMCU Pin |
|-----------|-------------|
| VCC       | 3.3V        |
| GND       | GND         |
| DATA      | D2 (GPIO4)  |

### Capacitive Soil Moisture Sensor
| Soil Sensor Pin | NodeMCU Pin |
|-----------------|-------------|
| VCC             | 3.3V        |
| GND             | GND         |
| AO (Analog Out) | A0          |

⚠️ **Important:** The NodeMCU operates on **3.3V logic**. Do **NOT** connect 5V sensors directly.

---

## 💻 Code (Arduino Sketch)

```cpp
#include "DHT.h"

// -------------------- DHT11 --------------------
#define DHTPIN 4     // GPIO4 (D2 on NodeMCU)
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

// -------------------- Soil Moisture --------------------
#define MOISTURE_PIN A0  // ESP8266 has only one ADC pin

// -------------------- Threshold Variables --------------------
float tempThresh = 30.0;   // Example: 30°C
float humMin = 40.0;       // Example: 40%
float humMax = 70.0;       // Example: 70%
float moistMin = 30.0;     // Example: 30%
float moistMax = 70.0;     // Example: 70%

// -------------------- Setup --------------------
void setup() {
  Serial.begin(115200);
  dht.begin();
  Serial.println("Sensor setup completed.");
}

// -------------------- Function: Read Sensors --------------------
void readSensors(float &temperature, float &humidity, int &moisture) {
  // DHT11
  temperature = dht.readTemperature();
  humidity = dht.readHumidity();

  // Soil Moisture
  moisture = analogRead(MOISTURE_PIN);
  moisture = map(moisture, 0, 1023, 0, 100);  // convert to %
}

// -------------------- Function: Check Logic --------------------
void checkLogic(float temperature, float humidity, int moisture) {
  if (temperature > tempThresh) {
    Serial.println("⚠️ Temperature too high!");
  }
  if (humidity < humMin || humidity > humMax) {
    Serial.println("⚠️ Humidity out of range!");
  }
  if (moisture < moistMin || moisture > moistMax) {
    Serial.println("⚠️ Soil Moisture out of range!");
  }
}

// -------------------- Loop --------------------
void loop() {
  static unsigned long lastMillis = 0;
  if (millis() - lastMillis > 5000) {  // every 5 seconds
    lastMillis = millis();

    // 1. Read sensors
    float temperature, humidity;
    int moisture;
    readSensors(temperature, humidity, moisture);

    // 2. Print readings
    Serial.print("🌡 Temp: "); Serial.print(temperature); Serial.println(" °C");
    Serial.print("💧 Hum: "); Serial.print(humidity); Serial.println(" %");
    Serial.print("🌱 Moisture: "); Serial.print(moisture); Serial.println(" %");

    // 3. Check logic
    checkLogic(temperature, humidity, moisture);

    Serial.println("-----------------------------");
  }
}
