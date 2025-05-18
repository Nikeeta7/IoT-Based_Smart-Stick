#include <SoftwareSerial.h>
#include <TinyGPS++.h>

const int trigPin = 9;
const int echoPin = 10;
const int vibPin = 3;
const int buzzerPin = 12;
const int sosButtonPin = 2;

// GPS and ESP Serial
SoftwareSerial gpsSerial(4, 5);  // RX, TX for GPS
SoftwareSerial espSerial(6, 7);  // RX, TX for ESP

TinyGPSPlus gps;

unsigned long lastSendTime = 0;
const unsigned long sendInterval = 15000; // Send every 15 sec

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(vibPin, OUTPUT);
  pinMode(buzzerPin, OUTPUT);
  pinMode(sosButtonPin, INPUT_PULLUP);

  Serial.begin(9600);
  gpsSerial.begin(9600);
  espSerial.begin(115200);

  Serial.println("System Started");

  // Send Wi-Fi and ThingSpeak setup to ESP8266
  espSerial.println("AT");
  delay(1000);
  espSerial.println("AT+CWMODE=1");
  delay(1000);
  espSerial.println("AT+CWJAP=\"AndroidShare_HH\",\"02723485\"");  // ✅ Fixed
  delay(5000);
}

void loop() {
  // Read GPS data
  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  // Trigger ultrasonic pulse
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);  
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10); 
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 30000);
  float distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.print(distance);
  Serial.println(" cm");

  // Vibration & buzzer based on distance
  if (distance > 2 && distance <= 150) {
    int vibrationStrength = map(distance, 10, 150, 255, 0);
    vibrationStrength = constrain(vibrationStrength, 0, 255);
    analogWrite(vibPin, vibrationStrength);

    int beepDelay = map(distance, 10, 100, 50, 500);
    beepDelay = constrain(beepDelay, 50, 500);

    digitalWrite(buzzerPin, HIGH);
    delay(10);
    digitalWrite(buzzerPin, LOW);
    delay(beepDelay);
  } else {
    analogWrite(vibPin, 0);
    digitalWrite(buzzerPin, LOW);
    delay(200);
  }

  // SOS Check
  if (digitalRead(sosButtonPin) == LOW) {
    sendToThingSpeak("999.999", "999.999", "1"); // Emergency flag 1
    Serial.println("SOS Button Pressed!");
    delay(2000); // Debounce
  }

  // Send GPS to Thingspeak every 15 seconds
  if (millis() - lastSendTime > sendInterval && gps.location.isValid()) {
    float lat = gps.location.lat();
    float lng = gps.location.lng();
    sendToThingSpeak(String(lat, 6), String(lng, 6), "0");
    lastSendTime = millis();
  }
}

void sendToThingSpeak(String latitude, String longitude, String sosFlag) {
  String cmd = "AT+CIPSTART=\"TCP\",\"api.thingspeak.com\",80";
  espSerial.println(cmd);
  delay(2000);

  String httpRequest = 
    "GET /update?api_key=3GIBTUTZV3WM4N71&field1=" + latitude + 
    "&field2=" + longitude + 
    "&field3=" + sosFlag + 
    " HTTP/1.1\r\nHost: api.thingspeak.com\r\nConnection: close\r\n\r\n";

  espSerial.print("AT+CIPSEND=");
  espSerial.println(httpRequest.length());
  delay(1000);
  espSerial.print(httpRequest);
  delay(2000);
  Serial.println("Data Sent to ThingSpeak");
}
