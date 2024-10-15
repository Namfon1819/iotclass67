# Ingest and store real-time data from IoT sensors.
1. การเริ่มต้นเซ็นเซอร์
- เซ็นเซอร์ BMP280: ใช้วัดอุณหภูมิและความดัน ถูกเริ่มต้นในฟังก์ชัน init_BMP280()
- เซ็นเซอร์ SHT4x: ใช้วัดความชื้นและอุณหภูมิ ถูกเริ่มต้นในฟังก์ชัน init_SHT4x()
- เซ็นเซอร์ Analog: อ่านค่าจากพินที่ถูกกำหนดเป็น sensorPin
2. การเก็บข้อมูล
-ในฟังก์ชัน loop() จะมีการอ่านข้อมูลจากเซ็นเซอร์ BMP280, SHT4x และเซ็นเซอร์ Analog อย่างสม่ำเสมอ
-ใช้ bmp280.readTemperature() และ bmp280.readPressure() เพื่อดึงค่าอุณหภูมิและความดันจากเซ็นเซอร์ BMP280
-ใช้ sensor.measureLowestPrecision(temp, humid) เพื่อดึงค่าความชื้นและอุณหภูมิจากเซ็นเซอร์ SHT4x
-ใช้ analogRead(sensorPin) เพื่ออ่านค่าจากพิน Analog ที่กำหนด
3. การจัดรูปแบบและการส่งข้อมูล
- ข้อมูลจะถูกจัดรูปแบบเป็น JSON โดยใช้คลาส StaticJsonDocument และ JsonObject
ข้อมูลที่ส่งไปยัง MQTT ประกอบด้วย:
- "id": รหัสที่ไม่ซ้ำสำหรับเซ็นเซอร์
- "name": ชื่อของเซ็นเซอร์
- "place_id": รหัสที่ระบุสถานที่ของเซ็นเซอร์
- "date": วันที่และเวลาปัจจุบันที่ได้จาก NTP
- "timestamp": เวลาปัจจุบันในรูปแบบ epoch time
- "payload": ออบเจ็กต์ย่อยที่เก็บข้อมูลเซ็นเซอร์ เช่น อุณหภูมิ ความชื้น ความดัน และแสง (ค่าจากเซ็นเซอร์ Analog)
4. การส่งข้อมูลไปยัง MQTT
- ข้อมูลจะถูกแปลงเป็นรูปแบบ JSON และส่งไปยังหัวข้อ (topic) iot-frames ของ MQTT
- ฟังก์ชัน client.publish("iot-frames", jsonData) จะส่งข้อมูล JSON ที่แปลงแล้วไปยังหัวข้อที่กำหนด
  
## MQTT Topic and MQTT Payload
Topic (หัวข้อ) "iot-frames"
หัวข้อคือช่องทางในการส่งและรับข้อมูลผ่าน MQTT Broker ซึ่งลูกค้าสามารถส่งข้อมูลไปยังหัวข้อนี้ และลูกค้าอื่นสามารถสมัครสมาชิกเพื่อรับข้อมูลจากหัวข้อนี้ได้
Payload (ข้อมูลที่ส่ง) ข้อมูล JSON ที่ถูกส่งไปยัง MQTT ประกอบด้วย
- "id": รหัสที่ไม่ซ้ำสำหรับเซ็นเซอร์
- "name": ชื่อของเซ็นเซอร์
- "place_id": รหัสที่ระบุสถานที่ของเซ็นเซอร์
- "date": วันที่และเวลาที่จัดรูปแบบแล้ว
- "timestamp": เวลาปัจจุบันในรูปแบบ epoch time
- "payload": ข้อมูลที่ได้รับจากเซ็นเซอร์ เช่น อุณหภูมิ ความชื้น ความดัน และแสง

ตัวอย่าง Playload
{{"id":"12434347","name":"iot_sensor_3","place_id":"32347983","date":"2024-08-23T17:30:28","timestamp":1724409028,"payload":{"temperature":30.91210747,"humidity":35.73914719,"pressure":100753,"luminosity":8064}}

## ESP32


![iot](https://github.com/user-attachments/assets/9bfe9b17-ff2b-49e5-b319-213eb42c6d9c)


```cpp
#include <Wire.h>  
#include <Adafruit_BMP280.h>  
#include <Adafruit_MPU6050.h>  
#include <SensirionI2cSht4x.h>  
#include <Adafruit_NeoPixel.h> 
#include <ESPNtpClient.h>  
#include <PubSubClient.h> 
#include <ArduinoJson.h> 
#include <WiFiUdp.h>  
#include <WiFi.h> 
#include <SPI.h>  

/// การตั้งค่า WiFi
const char* ssid = "TP-Link_CA30";  
const char* password = "29451760"; 
const char* mqtt_server = "172.16.46.55";  
const int mqtt_port = 1883; 

/// สำหรับผู้ใช้และรหัสผ่าน
const char* mqtt_user = "ginano03"; 
const char* mqtt_password = "1234";  

/// การตั้งค่าเครือข่าย NTP
const PROGMEM char* ntpServer = "158.108.212.149";  

/// การตั้งค่าเซ็นเซอร์
// BMP280
Adafruit_BMP280 bmp280; 

// SHT4x (DHT 11 ในที่นี้)
SensirionI2cSht4x sensor; 

// เซ็นเซอร์วัดแสง
int sensorPin = 5; 
int analogValue = 1;  

/// ค่าคงที่สำหรับการจัดการข้อผิดพลาด
static char errorMessage[64]; 
static int16_t error; 
#define NO_ERROR 0  

/// การตั้งค่าเครือข่าย
WiFiClient espClient;  
PubSubClient client(espClient);  
WiFiServer server(1883);  
IPAddress local_IP(172, 16, 46, 56); 
IPAddress gateway(172, 16, 46, 254);  
IPAddress subnet(255, 255, 255, 0); 

// การตั้งค่า NeoPixel
#define NUM_PIXELS  (8)     
#define WS2812_PIN  (GPIO_NUM_18) 

// กำหนดสีต่างๆ สำหรับแสดงสถานะ
const uint32_t COLOR_CONNECTING = 0xFFFFFF; 
const uint32_t COLOR_CONNECTED = 0x00FF00; 
const uint32_t COLOR_CONNECTION_FAILED = 0xFF0000; 
const uint32_t COLOR_MQTT_CONNECTING = 0x00FFFF;
const uint32_t COLOR_MQTT_CONNECTED = 0x00008B; 
const uint32_t COLOR_MQTT_FAILED = 0xFFFF00; 
const uint32_t COLOR_PUBLISHING = 0x0000FF; 
const uint32_t COLOR_PUBLISHING_2 = 0x800080;
const uint32_t COLOR_FAILED_TO_PUBLISH = 0xFFA500;

// สร้างวัตถุจากคลาส 'Adafruit_NeoPixel' 
Adafruit_NeoPixel pixels(NUM_PIXELS, WS2812_PIN, NEO_RGB + NEO_KHZ800);

// ฟังก์ชันเพื่อกำหนดสีของ NeoPixel
void setPixelColor(uint32_t color) {
  pixels.fill(color);  
  pixels.setBrightness(20); 
  pixels.show();  
}

/// ฟังก์ชันเชื่อมต่อ WiFi
void setup_wifi() {
  delay(100);  
  Serial.println(); 
  Serial.print("Connecting to "); 
  Serial.println(ssid);  

  WiFi.begin(ssid, password);  

  // การวนลูป 5 วินาทีสำหรับพยายามเชื่อมต่อ WiFi
  unsigned long startAttemptTime = millis();

  while (WiFi.status() != WL_CONNECTED && millis() - startAttemptTime < 5000) {
    delay(500);  
    Serial.print("_");  
    setPixelColor(COLOR_CONNECTING);  
    delay(300);  
    setPixelColor(COLOR_CONNECTION_FAILED);  
  }

  if (WiFi.status() != WL_CONNECTED) { 
    Serial.println(" ");  
    Serial.println("Connection failed!");  
    setPixelColor(COLOR_CONNECTION_FAILED);
    delay(3000); 
    Serial.println("Retrying...");  
    setup_wifi(); 
  } else {
    // หากเชื่อมต่อสำเร็จ
    Serial.println(" "); 
    Serial.println("Success to connect with WiFi!");  
    Serial.println("IP address: ");  
    Serial.println(WiFi.localIP());  
    setPixelColor(COLOR_CONNECTED);
    delay(2000);
  }
}

void CheckWifi(){
   // ตรวจสอบการเชื่อมต่อ WiFi ก่อน    
   if(WiFi.status() != WL_CONNECTED) {
    Serial.println("WiFi lost connection, trying to reconnect...");
    setPixelColor(COLOR_CONNECTION_FAILED); 
    delay(3000);  
    setup_wifi();  
    return;  
    }
}
/// ฟังก์ชันจัดการการเชื่อมต่อกับ MQTT
void mqtt_handle() {
  while (!client.connected()) {
    CheckWifi();
    Serial.print("Connecting with MQTT...");  
    setPixelColor(COLOR_MQTT_CONNECTING);  
    delay(300);  
    setPixelColor(COLOR_CONNECTING);  
    delay(300);  
    setPixelColor(COLOR_MQTT_CONNECTING);  
    delay(300);  

    if (client.connect("ClientNGN", mqtt_user, mqtt_password)) {  
      Serial.println("Already connected"); 
      client.subscribe("esp32/sensorData"); 
      setPixelColor(COLOR_MQTT_CONNECTED);  
      delay(2000);  
     
    } else {
      setPixelColor(COLOR_MQTT_FAILED);  
      Serial.print("Fails to connect with MQTT");  
      Serial.print("Client State: ");  
      Serial.println(client.state());  
      delay(5000); 
      Serial.println("Try in 10 sec");  
    }
  }
}

/// ฟังก์ชันเริ่มต้นการทำงานของ BMP280
void init_BMP280() {
  if (!bmp280.begin(0x76)) {  
    Serial.println(F("BMP280 Down"));  
    while (1) delay(10);  

  bmp280.setSampling(Adafruit_BMP280::MODE_NORMAL,
                     Adafruit_BMP280::SAMPLING_X2,
                     Adafruit_BMP280::SAMPLING_X16,
                     Adafruit_BMP280::FILTER_X16,
                     Adafruit_BMP280::STANDBY_MS_500);  
  Serial.println("BMP280 ready");  
  }
}
/// ฟังก์ชันเริ่มต้นการทำงานของ SHT4x
void init_SHT4x() {
  sensor.begin(Wire, SHT40_I2C_ADDR_44);  
  sensor.softReset(); 
  delay(10);  
  uint32_t serialNumber = 0;  
  error = sensor.serialNumber(serialNumber);  
  checkError(error, "Error to call serialNumber()");  
  Serial.print("serialNumber: ");  
  Serial.println(serialNumber);  
}

/// ฟังก์ชันจัดการข้อผิดพลาด
void checkError(int16_t error, const char* errorMsg) {
  if (error != NO_ERROR) {  
    Serial.print(errorMsg);  
    errorToString(error, errorMessage, sizeof errorMessage);  
    Serial.println(errorMessage);  
  }
}

/// ฟังก์ชันเริ่มต้นการทำงานของ NTP
void init_NTP() {
  NTP.setTimeZone(TZ_Asia_Bangkok);  
  NTP.setInterval(600);  
  NTP.setNTPTimeout(5000);  
  NTP.begin(ntpServer);  
}

// ฟังก์ชัน setup สำหรับการตั้งค่าเริ่มต้น
void setup() {
  Wire.begin(41, 40);  
  Serial.begin(115200);  

  // การตั้งค่าเครือข่าย
  setup_wifi();  
  client.setServer(mqtt_server, mqtt_port);  
  WiFi.begin(ssid, password);  
  while (WiFi.status() != WL_CONNECTED) {  
    delay(500);  
    Serial.print(".");  
  }
  server.begin();  

  // ตรวจสอบว่าการกำหนดค่า WiFi ล้มเหลวหรือไม่
  if (!WiFi.config(local_IP, gateway, subnet)) {
    Serial.println("WiFi Failed to configure");  
    delay(5000); 
    setPixelColor(COLOR_CONNECTION_FAILED);  
  }

  // การตั้งค่าเริ่มต้นของเซ็นเซอร์
  init_BMP280();  
  init_SHT4x();  

  // การตั้งค่า NTP
  init_NTP();  
}

/// ฟังก์ชันเพื่อรับเวลา Epoch ปัจจุบัน
unsigned long Get_EpochTime() {
    time_t now;  
    struct tm timeinfo;  
    if (!getLocalTime(&timeinfo)) {  
      return 0;  
    }
    time(&now);  
    return now;  
}

/// ฟังก์ชัน loop หลักที่ทำงานตลอดเวลา
void loop() {
  
  CheckWifi();
  if (!client.connected()) {  
    mqtt_handle();  
  }
  client.loop();  

  // อ่านค่าข้อมูลจากเซ็นเซอร์ต่างๆ
  float temp = bmp280.readTemperature();  
  float pressure = bmp280.readPressure()/100;  
  float aTemperature = 0.0;  
  float humid = 0.0;  
  error = sensor.measureLowestPrecision(temp, humid);  
  checkError(error, "Error to call STHX4");  
  int analogval = analogRead(sensorPin);  
  unsigned long epochTime = Get_EpochTime();  

  // พิมพ์ค่าข้อมูลที่อ่านได้จากเซ็นเซอร์ต่างๆ ลง Serial Monitor
  Serial.println(temp);  
  Serial.println(pressure);  
  Serial.println(humid);  
  Serial.println(analogval);  

  // สร้างเอกสาร JSON เพื่อส่งข้อมูล
  StaticJsonDocument<512> jsonDoc;  
  jsonDoc["name"] = "iot_sensor_3";
  jsonDoc["place_id"] = "32347983";
  jsonDoc["date"] = NTP.getTimeDateString(time(NULL), "%Y-%m-%dT%H:%M:%S");  
  jsonDoc["timestamp"] = epochTime;  

  // สร้าง JSON object สำหรับ payload
  JsonObject payload = jsonDoc.createNestedObject("payload");  
  payload["temperature"] = temp;  
  payload["humidity"] = humid;  
  payload["pressure"] = pressure;  
  payload["luminosity"] = analogval;  

  char jsonData[512];  
  serializeJson(jsonDoc, jsonData);  

  // ส่งข้อมูลผ่าน MQTT
  if (client.publish("iot-frames", jsonData)) {  

    setPixelColor(COLOR_PUBLISHING);  
    delay(300);  
    setPixelColor(COLOR_PUBLISHING_2); 

  } else {

    setPixelColor(COLOR_FAILED_TO_PUBLISH);  
    delay(500);  
  }

  // พิมพ์เวลาในรูปแบบ ISO 8601 ลง Serial Monitor
  Serial.print("Time:");  
  Serial.print(NTP.getTimeDateString(time(NULL), "%Y-%m-%dT%H:%M:%S"));  
  Serial.println(""); 
  delay(3000);  
}



```
