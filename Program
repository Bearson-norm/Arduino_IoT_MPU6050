/*
 * Program Monitoring Keseimbangan Bidang Datar Menggunakan Sensor MPU 6050
 * dengan Komunikasi Serial SDA dan SCL, Perangkat yang digunakan adalah 
 * WeMos D1R1 yang disambungkan komunikasinya ke sistem IoT yaitu Thingspeak
 */
#include <ESP8266WiFi.h>
#include <Wire.h>
#include <Servo.h>
#define buzzerPin D7

String Gabungan;
float RateRoll, RatePitch, RateYaw;
float AccX, AccY, AccZ, AccXp, AccZp;
float AngleRoll, AnglePitch;
float LoopTimer;

void gyro_signals(void) {
  Wire.beginTransmission(0x68);
  Wire.write(0x1A);
  Wire.write(0x05);
  Wire.endTransmission();
  Wire.beginTransmission(0x68);
  Wire.write(0x1C);
  Wire.write(0x10);
  Wire.endTransmission();
  Wire.beginTransmission(0x68);
  Wire.write(0x3B);
  Wire.endTransmission(); 
  Wire.requestFrom(0x68,6);
  int16_t AccXLSB = Wire.read() << 8 | Wire.read();
  int16_t AccYLSB = Wire.read() << 8 | Wire.read();
  int16_t AccZLSB = Wire.read() << 8 | Wire.read();
  Wire.beginTransmission(0x68);
  Wire.write(0x1B); 
  Wire.write(0x8);
  Wire.endTransmission();                                                   
  Wire.beginTransmission(0x68);
  Wire.write(0x43);
  Wire.endTransmission();
  Wire.requestFrom(0x68,6);
  int16_t GyroX=Wire.read()<<8 | Wire.read();
  int16_t GyroY=Wire.read()<<8 | Wire.read();
  int16_t GyroZ=Wire.read()<<8 | Wire.read();
  RateRoll=(float)GyroX/65.5;
  RatePitch=(float)GyroY/65.5;
  RateYaw=(float)GyroZ/65.5;
  AccXp=(float)AccXLSB/4096;
  AccX = AccXp - 0.11;
  AccY=(float)AccYLSB/4096;
  AccZp=(float)AccZLSB/4096;
  AccZ = AccZp - 0.07;
  AngleRoll=atan(AccY/sqrt(AccX*AccX+AccZ*AccZ))*1/(3.142/180);
  AnglePitch=-atan(AccX/sqrt(AccY*AccY+AccZ*AccZ))*1/(3.142/180);
}

// ThingSpeak settings
String apiKey = "NDQ8ZR22VFGBK8JY";
const char* server = "api.thingspeak.com";

// WiFi settings
const char* ssid = "@staff.fst.uinjkt.ac.id";
const char* password = "@kreditas1A";

WiFiClient client;

void setup() {
  pinMode(buzzerPin, OUTPUT);
  Serial.begin(115200);
  delay(10);

  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("WiFi connected");

  Wire.setClock(400000);
  Wire.begin();
  delay(250);
  Wire.beginTransmission(0x68);
  Wire.write(0x6B);
  Wire.write(0x00);
  Wire.endTransmission();
}

void loop() {
  gyro_signals();
  Serial.print(" Kemiringan X [g]= ");
  Serial.print(AccX);
  Serial.print(" Kemiringan Y [g]= ");
  Serial.print(AccY);
  Serial.print(" Kemiringan Z [g]= , ");
  Serial.println(AccZ);
  delay(500);
    if (AccX > 0.2 || AccX < -0.2 || AccY > 0.2 || AccY <-0.2){
      digitalWrite(buzzerPin, HIGH);
      delay(50);
      digitalWrite(buzzerPin, LOW);
      delay(50);
    } else{
      digitalWrite(buzzerPin, LOW);
      delay(500);
    }
    // Send data to ThingSpeak
    if (client.connect(server, 80)) {
      String postStr = apiKey;
      postStr += "&field4=" + String(AccX);
      postStr += "&field5=" + String(AccY);
      postStr += "&field6=" + String(AccZ);
      postStr += "&field7=" + String(Gabungan);
      postStr += "\r\n\r\n";

      client.print("POST /update HTTP/1.1\n");
      client.print("Host: api.thingspeak.com\n");
      client.print("Connection: close\n");
      client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
      client.print("Content-Type: application/x-www-form-urlencoded\n");
      client.print("Content-Length: ");
      client.print(postStr.length());
      client.print("\n\n");
      client.print(postStr);

      Serial.println("Data sent to ThingSpeak");
    } else {
      Serial.println("Failed to connect to ThingSpeak");
    }
    client.stop();
    
  // Delay before sending the next set of data
  delay(1000); // 1 minute
}
