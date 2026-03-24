# Doorbell-with-blynk
Doorbell with blynk

#define BLYNK_TEMPLATE_ID "TMPL3gsIRApFJ"
#define BLYNK_TEMPLATE_NAME "Door bell"
#define BLYNK_AUTH_TOKEN "ejriQ3-EZ1-zW5xm7N1Ufa1k-CyWaueK"

#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>

// HC-SR04 pins
#define TRIG_PIN 5
#define ECHO_PIN 18

// Buzzer pin
#define BUZZER_PIN 19  // You can change this to any digital GPIO pin

char ssid[] = "dlink-BA53";
char pass[] = "ekwtb96726";

long duration;
float distance;

BlynkTimer timer;

void sendNotificationIfClose() {
  // Trigger the sensor
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  // Read echo time
  duration = pulseIn(ECHO_PIN, HIGH);
  distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  Blynk.virtualWrite(V0, distance); // Send to Blynk app

  if (distance < 5.0) {
    Blynk.logEvent("Door_bell", "⚠️ Someone is at the door!");
    Serial.println("🔔 Notification sent + Buzzer ON");
    for(int i=0 ; i<5 ; i++)
    {
      digitalWrite(BUZZER_PIN, HIGH);  // Turn ON buzzer
      delay(100);
      digitalWrite(BUZZER_PIN, LOW);
      delay(100);
    }
  } else {
    digitalWrite(BUZZER_PIN, LOW);   // Turn OFF buzzer
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(BUZZER_PIN, LOW);  // Ensure buzzer is off initially

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  // Check every 500ms (0.5 sec)
  timer.setInterval(500L, sendNotificationIfClose);
}

void loop() {
  Blynk.run();
  timer.run();
}
