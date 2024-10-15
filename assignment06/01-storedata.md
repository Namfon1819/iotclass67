# Store data.

Kafka ไปยัง MongoDB
ภาพรวม
กระบวนการนี้มีจุดประสงค์เพื่อย้ายข้อมูลที่ผ่านการประมวลผลแล้วจากหัวข้อ Kafka ไปยังคอลเลกชันที่เกี่ยวข้องใน MongoDB เพื่อใช้ในการวิเคราะห์และดูข้อมูลผ่านเครื่องมือ เช่น MongoDB-Express ได้ในอนาคต

มีการตั้งค่า MongoDBSinkConnector จำนวน 3 อินสแตนซ์ โดยแต่ละอินสแตนซ์จะดึงข้อมูลจากหัวข้อ Kafka และจัดเก็บในคอลเลกชันที่ระบุไว้ใน MongoDB ดังนี้:

ย้ายข้อมูลจากหัวข้อ iot-frames ไปยังคอลเล็กชัน iot_frames ในฐานข้อมูล iot
ย้ายข้อมูลจากหัวข้อ iot-aggregate-metrics-sensor ไปยังคอลเล็กชัน iot_aggregate_metrics_sensor
ย้ายข้อมูลรวมตามสถานที่จากหัวข้อ iot-aggregate-metrics-place ไปยังคอลเล็กชัน iot_aggregate_metrics_place
การกำหนดค่า
1. MongoDB Sink Connector สำหรับ iot-frames
ตัวเชื่อมต่อนี้จะย้ายข้อมูลจากหัวข้อ iot-frames ไปยังคอลเล็กชัน iot_frames ใน MongoDB โดยข้อมูลอยู่ในรูปแบบ JSON โดยไม่ต้องใช้ schema เช่น Avro หรือ JSON-Schema เพื่อให้กระบวนการง่ายขึ้น

```cpp
{
   "name":"iot-frames-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-frames",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_frames",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.json.JsonConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable":false
   }
}
```

2. MongoDB Sink Connector สำหรับ iot-aggregate-metrics-sensor
ตัวเชื่อมต่อนี้จะย้ายข้อมูลเมตริกที่รวมตามเซ็นเซอร์จากหัวข้อ iot-aggregate-metrics-sensor ไปยังคอลเล็กชัน iot_aggregate_metrics_sensor ใน MongoDB

```cpp
{
   "name":"iot-aggregate-metrics-sensor-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-sensor",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_sensor",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable": false
   }
}
```
3. MongoDB Sink Connector สำหรับ iot-aggregate-metrics-place
ตัวเชื่อมต่อนี้จะย้ายข้อมูลเมตริกที่รวมตามสถานที่จากหัวข้อ iot-aggregate-metrics-place ไปยังคอลเล็กชัน iot_aggregate_metrics_place ใน MongoDB

```cpp
{
   "name":"iot-aggregate-metrics-place-mongodb-sink",
   "config":{
      "connector.class":"com.mongodb.kafka.connect.MongoSinkConnector",
      "tasks.max":1,
      "topics":"iot-aggregate-metrics-place",
      "connection.uri":"mongodb://devroot:devroot@mongo:27017",
      "database":"iot",
      "collection":"iot_aggregate_metrics_place",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter.schemas.enable": false,
      "key.converter.schemas.enable": false
   }
}
```
Kafka ไปยัง Prometheus
สำหรับการใช้งานร่วมกับ Prometheus ต้องกำหนดค่าตัวเชื่อมต่อเพื่อย้ายข้อมูลจากหัวข้อ iot-metric-time-series โดย Prometheus จะรับข้อมูลผ่านการดึงข้อมูล (scraping) ดังนั้นตัวเชื่อมต่อจะเปิดใช้งานเซิร์ฟเวอร์ HTTP เพื่อให้ Prometheus ค้นหาข้อมูล

ตัวเชื่อมต่อ Kafka Connect Prometheus Metrics Sink ทำให้ข้อมูลนี้พร้อมสำหรับการขูดข้อมูลโดยเซิร์ฟเวอร์ Prometheus โดยตัวเชื่อมต่อรองรับข้อมูลในรูปแบบ JSON แบบไม่มีโครงร่างจาก Kafka

```cpp
{
  "name" : "prometheus-connector-sink",
  "config" : {
   "topics":"iot-metrics-time-series",
   "connector.class" : "io.confluent.connect.prometheus.PrometheusMetricsSinkConnector",
   "tasks.max" : "1",
   "confluent.topic.bootstrap.servers":"kafka:9092",
   "prometheus.scrape.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "prometheus.listener.url": "http://0.0.0.0:8084/iot-metrics-time-series",
   "value.converter": "org.apache.kafka.connect.json.JsonConverter",
   "key.converter": "org.apache.kafka.connect.json.JsonConverter",
   "value.converter.schemas.enable": false,
   "key.converter.schemas.enable":false,
   "reporter.bootstrap.servers": "kafka:9092",
   "reporter.result.topic.replication.factor": "1",
   "reporter.error.topic.replication.factor": "1",
   "behavior.on.error": "log"
  }
}
```
