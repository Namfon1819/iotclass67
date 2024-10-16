# MQTT
# Ingest and store real-time data from IoT sensors
อธิบาย 3 ส่วนนี้ สร้างมาได้อย่างไร

# iot-sensor-1
IoT sensor 1 เป็น sensor ที่ถูกจําลองด้วยไมโครเซอร์วิสที่ใช้ใน Spring Boot (ผ่านไลบรารี Eclipse Paho MQTT) ที่ถูกติดตั้งอยู่บนเซิฟเวอร์ โดยจะส่งข้อมูล telemetry ไปยังโบรกเกอร์ Eclipse Mosquitto ข้อมูลที่ถูกจำลองนี้ generate ค่า ทุกอย่างภายใน payload มาจาก Callable โดยจะถูกสร้างขึ้นทุกวินาทีและ มี payload ในรูปแบบที่สร้างขึ้นให้ตรงกัน
# iot-sensor-2
เป็น sensor ที่ถูกจําลองด้วยไมโครเซอร์วิสที่ใช้ใน Spring Boot (ผ่านไลบรารี Eclipse Paho MQTT)เช่นเดียวกันกับ sensor 1 เพียงแต่ติดตั้งอยู่ในเครื่องของคนในทีม

ทำ ot-sensor-1 และ iot-sensor-2 ได้ที่ pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.dreamsoftware</groupId>
    <artifactId>iotframesingest</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>iot_sensor</name>
    <description>Simulate IoT Sensor</description>

    <properties>
        <java.version>1.8</java.version>
        <eclipse-paho-mqtt.version>1.2.0</eclipse-paho-mqtt.version>
        <jackson-bind.version>2.9.8</jackson-bind.version>
    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.eclipse.paho</groupId>
            <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
            <version>${eclipse-paho-mqtt.version}</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson-bind.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <finalName>iot_sensor</finalName>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>1.18.12</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>

            <plugin>
                    <groupId>com.spotify</groupId>
                    <artifactId>dockerfile-maven-plugin</artifactId>
                    <version>1.4.6</version>
                    <dependencies>
                        <dependency>
                            <groupId>com.github.jnr</groupId>
                            <artifactId>jnr-unixsocket</artifactId>
                            <version>0.38.14</version>
                        </dependency>
                    </dependencies>
            </plugin>
        </plugins>
    </build>

</project>

```

หลังจากนั้นเข้าไปยังโฟลเดอร์ และ `complie` 
```bash
cd iot_event_streaming_architecture/microservices/iot_sensor

# Use maven complie iot sensor for arm
docker run -it --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven arm64v8/maven:3.8-jdk-8 mvn clean install

# Use maven complie iot sensor for x86
docker run -it --rm --name my-maven-project -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven:3.8-openjdk-8 mvn clean install
```

# iot-sensor-3-10
ถูกสร้างขึ้นโดยใช้บอร์ดไมโครคอนโทรลเลอร์ Cucumber ESP32 S2 ที่เชื่อมต่อกับเซ็นเซอร์ BMP280 สำหรับวัดอุณหภูมิและความดัน, SHT4x สำหรับวัดความชื้น และเซ็นเซอร์วัดแสง ข้อมูลจากเซ็นเซอร์จะถูกอ่านผ่านการสื่อสารแบบ I2C และขาอะนาล็อก จากนั้นส่งผ่านโปรโตคอล MQTT ไปยังเซิร์ฟเวอร์ในรูปแบบ JSON เพื่อจัดเก็บและวิเคราะห์ นอกจากนี้ ใช้ NeoPixel เพื่อแสดงสถานะการทำงานของระบบ เช่น การเชื่อมต่อ WiFi, การเชื่อมต่อ MQTT, และการส่งข้อมูล พร้อมกับใช้ NTP เพื่อซิงโครไนซ์เวลาแบบเรียลไทม์ ข้อมูลทั้งหมดจะถูกประมวลผลและส่งต่อแบบเรียลไทม์.
ที่มาของข้อมูล: เซ็นเซอร์เหล่านี้มาจาก Cucumber ESP32 S2 ซึ่งเป็นบอร์ดไมโครคอนโทรลเลอร์ที่ใช้ในโปรเจกต์ IoT
การทำงาน: เซ็นเซอร์เหล่านี้สามารถเก็บข้อมูลจากสภาพแวดล้อม เช่น อุณหภูมิ, ความชื้น, แสง, และอื่น ๆ ข้อมูลจะถูกส่งผ่าน MQTT ไปยังเซิร์ฟเวอร์ในรูปแบบเรียลไทม์
ประโยชน์: เซ็นเซอร์หลายตัวช่วยให้สามารถเก็บข้อมูลจากหลายจุดในพื้นที่เดียวกัน ทำให้สามารถวิเคราะห์และตรวจสอบข้อมูลได้อย่างละเอียด
![Screenshot 2024-08-28 164621](https://github.com/user-attachments/assets/cbaf6861-920b-4d75-8fd6-c566da254eb1)
![S__5251090](https://github.com/user-attachments/assets/fc0fb339-10ea-47b4-aaaa-d7133184cd88)



