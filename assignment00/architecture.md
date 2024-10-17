# Main technologies of architecture

## Architecture Overview


![image](https://github.com/user-attachments/assets/54482d9b-0445-4e2f-a378-bc87ed68a831)





## Eclipse Mosquitto
•Eclipse Mosquitto เป็น MQTT broker แบบ open-source ที่ทำหน้าที่เป็นตัวกลางในการส่งข้อความระหว่างอุปกรณ์ต่าง ๆ ในระบบ Internet of Things (IoT) โดยใช้โปรโตคอล MQTT (Message Queuing Telemetry Transport) ซึ่งเป็นโปรโตคอลที่เบาและเหมาะกับการสื่อสารในเครือข่ายที่มีแบนด์วิธจำกัด เช่น IoT

#หน้าที่ของ Eclipse Mosquitto
1) Broker สำหรับการส่งข้อความ (Message Broker)
2) รองรับการสื่อสารแบบเบาและรวดเร็ว
3) จัดการหัวข้อ (Topic Management)
4) รองรับ QoS (Quality of Service)
5) ความปลอดภัย (Security)
6) การเก็บสถานะล่าสุด (Retained Messages)
7) การจัดการกับการเชื่อมต่อขาด (Will Messages)
8) รองรับการทำงานในระบบขนาดเล็กและใหญ่

## Apache ZooKeeper
•Apache ZooKeeper เป็นระบบ distributed coordination service ที่ออกแบบมาเพื่อช่วยจัดการข้อมูลการกำหนดค่า (configuration) และการซิงโครไนซ์ (synchronization) ของระบบที่ทำงานแบบกระจาย (distributed systems) ให้เป็นไปอย่างมีประสิทธิภาพและสม่ำเสมอ

#หน้าที่ของ Apache ZooKeeper
1) การจัดเก็บและซิงโครไนซ์ข้อมูลแบบกระจาย
2) Distributed Lock Management
3) Leader Election (การเลือกหัวหน้าโหนด)
4) Configuration Management
5) Service Discovery (การค้นหาบริการ)
6) การซิงโครไนซ์ข้อมูลระหว่างหลายโหนด

## Apache Kafka
•Apache Kafka เป็นแพลตฟอร์มสำหรับ distributed streaming และ message queue ที่ออกแบบมาเพื่อจัดการ ข้อมูลแบบเรียลไทม์ ซึ่งรองรับการส่งข้อมูลจำนวนมหาศาลอย่างมีประสิทธิภาพและมีความน่าเชื่อถือสูง โดยใช้โมเดลการสื่อสารแบบ publish/subscribe

#หน้าที่ของ Apache Kafka
1) Message Broker (ตัวกลางในการส่งข้อมูล)
2) Streaming Data Platform
3) High Throughput (ประมวลผลข้อมูลจำนวนมากได้อย่างรวดเร็ว)
4) Data Pipeline
5) Fault-Tolerant (ความทนทานต่อความผิดพลาด)

## Apache Kafka Connect
•Apache Kafka Connect เป็น framework สำหรับการเชื่อมต่อ Kafka กับระบบภายนอก เช่น ฐานข้อมูล, ระบบจัดเก็บข้อมูล (data warehouses), APIs, และ ระบบสตรีมมิ่ง อื่น ๆ โดยไม่ต้องเขียนโค้ดใหม่เอง ทำให้การส่งข้อมูลเข้า (source) และดึงข้อมูลออก (sink) จาก Kafka ง่ายและมีประสิทธิภาพ

#หน้าที่ของ Apache Kafka Connect
1) เชื่อมต่อกับแหล่งข้อมูลและปลายทาง
2) ทำให้การจัดการข้อมูลเป็นอัตโนมัติ (Automation)
3) แปลงและจัดรูปแบบข้อมูล
4) Fault-tolerant และ Scalable

## Apache Kafka Streams
•Apache Kafka Streams เป็นไลบรารีสำหรับการพัฒนา stream processing applications ซึ่งช่วยให้คุณสามารถ ประมวลผลข้อมูลแบบเรียลไทม์ จาก Kafka topics ได้อย่างง่ายดายและมีประสิทธิภาพ แอปพลิเคชันที่สร้างด้วย Kafka Streams สามารถประมวลผลข้อมูล อย่างต่อเนื่อง และ เป็นสถานะ (stateful) ทำให้เหมาะกับงานที่ต้องมีการรวบรวมและวิเคราะห์ข้อมูลในทันที

#หน้าที่ของ Apache Kafka Streams
1) การประมวลผลข้อมูลแบบเรียลไทม์ (Real-time Stream Processing)
2) การรวมและแปลงข้อมูล (Aggregation and Transformation)
3) Stateful Processing (การประมวลผลแบบมีสถานะ)
4) การจัดการหน้าต่างเวลา (Windowing)
5) Distributed Processing (ประมวลผลแบบกระจาย)

## Prometheus
• เป็นระบบ monitoring และ alerting แบบ open-source ที่ถูกออกแบบมาสำหรับการเก็บข้อมูลจากแอปพลิเคชันและระบบต่าง ๆ ในรูปของ time-series data (ข้อมูลตามเวลา) เพื่อวิเคราะห์และติดตามประสิทธิภาพของระบบได้อย่างมีประสิทธิภาพ

#หน้าที่และความสามารถของ Prometheus
1) Monitoring และเก็บข้อมูล Time-Series Data
2) Pull Model สำหรับการดึงข้อมูล
3) Query ข้อมูลด้วย PromQL
4) Alerting
5) Multi-dimensional Data Model
6) Data Retention และการจัดเก็บ
7) การทำ Visualization

## MongoDB
• MongoDB เป็นระบบฐานข้อมูล NoSQL แบบ document-oriented ที่เก็บข้อมูลในรูปแบบ JSON-like documents มีความยืดหยุ่นสูง เหมาะสำหรับการจัดเก็บข้อมูลที่มีโครงสร้างไม่แน่นอนหรือเปลี่ยนแปลงบ่อย เช่น แอปพลิเคชันที่เติบโตเร็วหรือมีข้อมูลขนาดใหญ่

#คุณสมบัติของ MongoDB
1) Document-Oriented Storage
2) Schema-less Database (Flexible Schema)
3) Scalability (รองรับการขยายตัว)
4) Replication (การจำลองข้อมูล)
5) Querying ที่ทรงพลัง
6) Indexing เพื่อเพิ่มประสิทธิภาพ
7) Aggregation Framework
8) รองรับ Transaction

## Grafana
• Grafana เป็นเครื่องมอื ที่ใช้แสดงผลข้อมูลเป็นกราฟและแดชบอร์ด 
• สามารถดึงข้อมูลจากแหล่งต่างๆ เช่น Prometheus มาทำการแสดงผลในรูปแบบที่เข้าใจง่าย

# iot sensor
## BMP280 
เซ็นเซอร์วัดอุณหภูมิและความกดอากาศ ตัวนี้ถูกใช้ในการเก็บข้อมูลอุณหภูมิและความดันของสภาพแวดล้อม ข้อมูลเหล่านี้จะถูกอ่านจากเซ็นเซอร์ BMP280 ผ่านโปรโตคอล I2C จากนั้นค่าที่วัดได้จะถูกนำมาแสดงผลบน Serial Monitor และส่งผ่าน MQTT ในรูปแบบ JSON เพื่อให้สามารถนำไปวิเคราะห์และจัดเก็บในเซิร์ฟเวอร์

## SHT4x 
เป็นเซ็นเซอร์วัดอุณหภูมิและความชื้น ตัวนี้ถูกใช้ในการเก็บข้อมูลอุณหภูมิและความชื้นจากสภาพแวดล้อม เซ็นเซอร์นี้เชื่อมต่อผ่าน I2C และมีการตั้งค่าระบบเพื่อรีเซ็ตตัวเองและเริ่มทำงานใหม่เมื่อมีปัญหา ข้อมูลที่อ่านได้จาก SHT4x จะถูกจัดเก็บในโครงสร้าง JSON เพื่อส่งไปยัง MQTT สำหรับจัดเก็บและวิเคราะห์ต่อไป

## เซ็นเซอร์วัดแสง 
ตัวเซ็นเซอร์วัดแสงใช้ขาอนาล็อก (Pin 5) ในการเก็บข้อมูลระดับแสงภายในสภาพแวดล้อม เซ็นเซอร์นี้จะวัดระดับความเข้มของแสงโดยแปลงค่าเป็นสัญญาณอนาล็อกที่สามารถอ่านผ่านฟังก์ชัน analogRead และส่งข้อมูลไปยังเซิร์ฟเวอร์ผ่าน MQTT ในรูปแบบ JSON เช่นกัน

## NeoPixel 
ตัวนี้ไม่ได้เป็นเซ็นเซอร์ แต่เป็นอุปกรณ์แสดงผลสำหรับการเปลี่ยนสีตามสถานะของระบบ เช่น เมื่อเชื่อมต่อ WiFi, MQTT หรือตอนส่งข้อมูล จะใช้สีต่าง ๆ เพื่อบ่งบอกสถานะการทำงานของระบบ ช่วยให้ทราบว่าระบบทำงานตามที่คาดหมายหรือไม่
