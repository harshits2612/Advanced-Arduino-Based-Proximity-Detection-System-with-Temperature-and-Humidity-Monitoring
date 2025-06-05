# Advanced-Arduino-Based-Proximity-Detection-System-with-Temperature-and-Humidity-Monitoring
This Arduino project uses an ultrasonic sensor for distance measurement, a DHT22 sensor for temperature and humidity monitoring, and displays the results on an LCD. It also features LED indicators and an active buzzer for alerts, providing real-time environmental data.
# 🔧 Features
🧭 Distance Measurement
Utilizes the HC-SR04 ultrasonic sensor to detect object proximity.

# 🌡️ Temperature & Humidity Monitoring
Employs the DHT22 sensor to report ambient temperature and humidity.

# 📟 LCD Display
Shows distance, temperature, and humidity values on a 16x4 I2C LCD display.

# 🚥 LED Indicators

Green: Safe zone (20–50 cm)

Yellow: Warning zone (10–20 cm)

Red: Danger zone (<10 cm)

# 🔊 Active Buzzer
Audible alert when objects are detected in the danger zone (<10 cm).

# 🧩 Components Used
Arduino Uno

HC-SR04 Ultrasonic Sensor

DHT22 Temperature & Humidity Sensor

16x4 LCD with I2C Module

LEDs (Green, Yellow, Red) + 220Ω Resistors

Active Buzzer

Breadboard

Jumper Wires (Male-to-Male and Male-to-Female)

# 🛠️ Setup Instructions
# 🧠 Arduino Pin Configuration:

HC-SR04:

Trig → D9

Echo → D10

DHT22 → D7

LCD I2C:

SDA → A4

SCL → A5

LEDs:

Green → D13

Yellow → D12

Red → D11

Buzzer → D8

# 📤 Upload the Code:

Open the Arduino IDE.

Install required libraries:

LiquidCrystal_I2C

DHT sensor library by Adafruit

Copy and paste the code provided below.

Upload the code to the Arduino Uno.

# ⚙️ How It Works
The ultrasonic sensor calculates the distance to the nearest object by measuring the time taken for a pulse to return.

The DHT22 sensor monitors and displays temperature and humidity every second.

LED color and buzzer status change based on proximity:

Red LED + Buzzer → Danger (<10 cm)

Yellow LED → Caution (10–20 cm)

Green LED → Safe (20–50 cm)

# 💻 Code
Paste the provided Arduino code here — you've already written a great one. Consider wrapping it in a code block using triple backticks (```cpp) when uploading to GitHub for proper formatting.
Code

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

// Initialize the LCD with I2C address
LiquidCrystal_I2C lcd(0x27, 16, 2); // Change 0x27 if needed

// Define pins for LEDs and buzzer
const int greenLED = 13;  // Green LED pin
const int yellowLED = 12; // Yellow LED pin
const int redLED = 11;    // Red LED pin
const int buzzer = 8;     // Buzzer pin

// Define pins for HC-SR04
const int trigPin = 9;    // HC-SR04 Trigger pin
const int echoPin = 10;   // HC-SR04 Echo pin

// Define pin for DHT22
const int dhtPin = 7;     // DHT22 data pin

// Create DHT object
DHT dht(dhtPin, DHT22);

// Variables for distance measurement
long duration;
int distance;
const int numReadings = 10; // Number of readings for averaging
int readings[numReadings];  // Array to store distance readings
int index = 0;              // Current index in the readings array
int total = 0;              // Sum of the readings
int averageDistance = 0;    // Average distance

// Timing variables
unsigned long previousDHTMillis = 0;
const long dhtInterval = 1000; // Read DHT every second

void setup() {
  // Initialize the LCD
  lcd.begin(16, 2);
  lcd.backlight(); // Turn on the backlight

  // Set LED and buzzer pins as output
  pinMode(greenLED, OUTPUT);
  pinMode(yellowLED, OUTPUT);
  pinMode(redLED, OUTPUT);
  pinMode(buzzer, OUTPUT);

  // Set HC-SR04 pins
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // Initialize DHT sensor
  dht.begin();

  // Initialize readings array
  for (int i = 0; i < numReadings; i++) {
    readings[i] = 0;
  }
}

void loop() {
  // Trigger the ultrasonic sensor
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // Read the echo pin
  duration = pulseIn(echoPin, HIGH);
  distance = duration * 0.0344 / 2;

  // Update the readings array for averaging
  total -= readings[index];
  readings[index] = distance;
  total += readings[index];
  index = (index + 1) % numReadings;
  averageDistance = total / numReadings;

  // Non-blocking timing for DHT readings
  unsigned long currentMillis = millis();
  if (currentMillis - previousDHTMillis >= dhtInterval) {
    previousDHTMillis = currentMillis;
    float temperature = dht.readTemperature();
    float humidity = dht.readHumidity();

    // Display on LCD
    lcd.setCursor(0, 0); // Set cursor to the first row
    lcd.print("Dist: ");
    lcd.print(averageDistance);
    lcd.print(" cm   "); // Space to clear previous text

    lcd.setCursor(0, 1); // Set cursor to the second row
    lcd.print("T: ");
    lcd.print(temperature);
    lcd.print("C H: ");
    lcd.print(humidity);
    lcd.print("%   "); // Space to clear previous text
  }

  // Control LEDs and buzzer based on average distance
  if (averageDistance < 10) { // Danger zone
    digitalWrite(redLED, HIGH);
    digitalWrite(yellowLED, LOW);
    digitalWrite(greenLED, LOW);
    digitalWrite(buzzer, HIGH);
  } else if (averageDistance >= 10 && averageDistance < 20) { // Warning zone
    digitalWrite(redLED, LOW);
    digitalWrite(yellowLED, HIGH);
    digitalWrite(greenLED, LOW);
    digitalWrite(buzzer, LOW);
  } else if (averageDistance >= 20 && averageDistance <= 50) { // Safe zone
    digitalWrite(redLED, LOW);
    digitalWrite(yellowLED, LOW);
    digitalWrite(greenLED, HIGH);
    digitalWrite(buzzer, LOW);
  } else { // Distance greater than 50 cm
    digitalWrite(redLED, LOW);
    digitalWrite(yellowLED, LOW);
    digitalWrite(greenLED, LOW);
    digitalWrite(buzzer, LOW);
  }

  // Small delay to avoid overwhelming the loop
  delay(5); // Reduced to 5 ms for faster updates
}

