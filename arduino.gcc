#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <TinyGPS++.h>
#include <HardwareSerial.h>

// LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// GPS
TinyGPSPlus gps;
HardwareSerial gpsSerial(1);

// GSM
HardwareSerial gsmSerial(2);

// Pins
#define HEART_PIN 34
#define TEMP_PIN 35
#define BUZZER 25

// Thresholds
#define HEART_THRESHOLD 110
#define TEMP_THRESHOLD 38
#define MOTION_THRESHOLD 2000

int lastMotion = 0;

void setup() {
  Serial.begin(115200);

  lcd.init();
  lcd.backlight();

  pinMode(BUZZER, OUTPUT);

  gpsSerial.begin(9600, SERIAL_8N1, 16, 17);
  gsmSerial.begin(9600, SERIAL_8N1, 26, 27);

  lcd.setCursor(0, 0);
  lcd.print("Safety System");
  delay(2000);
  lcd.clear();
}

void loop() {
  // Read sensors
  int heartRate = analogRead(HEART_PIN) / 10; // Simulated BPM
  float temp = analogRead(TEMP_PIN) * (3.3 / 4095.0) * 100;
  int motion = analogRead(33);

  // GPS update
  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  // Display values
  lcd.setCursor(0, 0);
  lcd.print("HR:");
  lcd.print(heartRate);
  lcd.print(" T:");
  lcd.print(temp);

  // Panic detection
  bool panic = false;

  if (heartRate > HEART_THRESHOLD) panic = true;
  if (temp > TEMP_THRESHOLD) panic = true;
  if (abs(motion - lastMotion) > MOTION_THRESHOLD) panic = true;

  lastMotion = motion;

  if (panic) {
    triggerAlert();
  }

  delay(1000);
}

void triggerAlert() {
  digitalWrite(BUZZER, HIGH);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("PANIC ALERT!");

  String message = "Emergency! Need help. Location: ";

  if (gps.location.isValid()) {
    message += "Lat:";
    message += String(gps.location.lat(), 6);
    message += ",Lng:";
    message += String(gps.location.lng(), 6);
  } else {
    message += "Location not found";
  }

  sendSMS(message);

  delay(5000);
  digitalWrite(BUZZER, LOW);
}

void sendSMS(String msg) {
  gsmSerial.println("AT");
  delay(1000);

  gsmSerial.println("AT+CMGF=1");
  delay(1000);

  gsmSerial.println("AT+CMGS=\"+911234567890\""); // Replace number
  delay(1000);

  gsmSerial.print(msg);
  delay(1000);

  gsmSerial.write(26); // CTRL+Z to send
  delay(3000);
}
