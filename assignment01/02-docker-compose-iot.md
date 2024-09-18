
# IoT Docker compose
>> ให้นำไฟล์ docker-compose.yaml มาอธิบายว่า แต่ละส่วนคืออะไร โดยใช้การ comment ในไฟล์ docker-compose.yaml
>>  กำหนด volumes สำหรับเก็บข้อมูลที่ต้องการให้คงอยู่แม้คอนเทนเนอร์จะหยุดทำงาน
```cpp
volumes:
    prometheus_data: {}  # เก็บข้อมูลของ Prometheus
    grafana_data: {}     # เก็บข้อมูลของ Grafana
    zookeeper-data:      # เก็บข้อมูลของ ZooKeeper
      driver: local      # ใช้ driver local สำหรับเก็บข้อมูล
    zookeeper-log:       # เก็บ log ของ ZooKeeper
      driver: local      # ใช้ driver local สำหรับเก็บ log
    kafka-data:          # เก็บข้อมูลของ Kafka
      driver: local      # ใช้ driver local สำหรับเก็บข้อมูล

services:

   ZooKeeper เป็นบริการที่ช่วยในการจัดการข้อมูลการตั้งค่า
   การตั้งชื่อ และการซิงโครไนซ์แบบกระจาย
  zookeeper:
    image: confluentinc/cp-zookeeper  # ใช้ภาพ Docker ของ ZooKeeper
    container_name: zookeeper          # ตั้งชื่อคอนเทนเนอร์
    restart: unless-stopped            # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    volumes:
      - zookeeper-data:/var/lib/zookeeper/data  # เชื่อมต่อ volume สำหรับข้อมูล
      - zookeeper-log:/var/lib/zookeeper/log    # เชื่อมต่อ volume สำหรับ log
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181  # พอร์ตที่ ZooKeeper จะฟัง
      # การตั้งค่าต่างๆ สำหรับการบันทึก log ของ ZooKeeper
      ZOOKEEPER_LOG4J_ROOT_LOGLEVEL: INFO  # ระดับ log
      ZOOKEEPER_LOG4J_PROP: INFO,ROLLINGFILE  # การตั้งค่าการบันทึก log
      ZOOKEEPER_LOG_MAXFILESIZE: 10MB  # ขนาดไฟล์ log สูงสุด
      ZOOKEEPER_LOG_MAXBACKUPINDEX: 10  # จำนวนไฟล์ log สำรองสูงสุด
      ZOOKEEPER_SNAP_COUNT: 10  # จำนวน snapshot ที่เก็บ
      ZOOKEEPER_AUTOPURGE_SNAP_RETAIN_COUNT: 10  # จำนวน snapshot ที่เก็บไว้
      ZOOKEEPER_AUTOPURGE_PURGE_INTERVAL: 3  # ระยะเวลาการลบ snapshot

   Kafka เป็นแพลตฟอร์มการสตรีมข้อมูลแบบกระจาย
  kafka:
    image: confluentinc/cp-kafka  # ใช้ภาพ Docker ของ Kafka
    container_name: kafka          # ตั้งชื่อคอนเทนเนอร์
    volumes:
      - kafka-data:/var/lib/kafka  # เชื่อมต่อ volume สำหรับข้อมูล
    restart: unless-stopped        # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181  # เชื่อมต่อกับ ZooKeeper
      KAFKA_NUM_PARTITIONS: 1                    # จำนวนพาร์ติชัน
      KAFKA_COMPRESSION_TYPE: gzip                # ประเภทการบีบอัด
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1   # ปัจจัยการทำสำเนา
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092  # ที่อยู่ที่ Kafka จะประกาศ
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'true'    # อนุญาตให้สร้างหัวข้ออัตโนมัติ
    links:
      - zookeeper  # เชื่อมโยงกับบริการ ZooKeeper

   Kafka REST Proxy ให้ API REST สำหรับ Kafka
  # kafka-rest-proxy:
    image: confluentinc/cp-kafka-rest:latest  # ใช้ภาพ Docker ของ Kafka REST Proxy
    container_name: kafka-rest-proxy           # ตั้งชื่อคอนเทนเนอร์
    environment:
      KAFKA_REST_ZOOKEEPER_CONNECT: zookeeper:2181  # เชื่อมต่อกับ ZooKeeper
      KAFKA_REST_LISTENERS: http://0.0.0.0:8082/      # ที่อยู่ที่ REST Proxy จะฟัง
      KAFKA_REST_BOOTSTRAP_SERVERS: kafka:9092        # ที่อยู่ของ Kafka
    restart: unless-stopped                           # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    ports:
      - "9999:8082"                                   # เปิดพอร์ต 9999 สำหรับการเข้าถึง

   Kafka Connect เป็นเฟรมเวิร์คสำหรับเชื่อมต่อ Kafka กับระบบภายนอก
  # kafka-connect:
    image: confluentinc/cp-kafka-connect:latest  # ใช้ภาพ Docker ของ Kafka Connect
    container_name: kafka-connect                  # ตั้งชื่อคอนเทนเนอร์
    environment:
      CONNECT_BOOTSTRAP_SERVERS: "kafka:9092"     # ที่อยู่ของ Kafka
      CONNECT_GROUP_ID: kafka-connect-group        # กลุ่มที่เป็นเอกลักษณ์ของ Connect
      CONNECT_CONFIG_STORAGE_TOPIC: kafka-connect-meta-configs  # ชื่อหัวข้อสำหรับการเก็บข้อมูลการกำหนดค่า
      CONNECT_OFFSET_STORAGE_TOPIC: kafka-connect-meta-offsets    # ชื่อหัวข้อสำหรับการเก็บข้อมูลออฟเซ็ต
      CONNECT_STATUS_STORAGE_TOPIC: kafka-connect-meta-status      # ชื่อหัวข้อสำหรับสถานะ
      CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter  # ตัวแปลงคีย์
      CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter  # ตัวแปลงค่า
    restart: unless-stopped                         # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    volumes:
      - ./kafka_connect/data:/data                  # เชื่อมต่อ volume สำหรับข้อมูล
    command: 
      - bash 
      - -c 
      - |
        echo "Launching Kafka Connect worker"  # แสดงข้อความเมื่อเริ่มทำงาน
        /etc/confluent/docker/run &  # เริ่ม Kafka Connect
        echo "Waiting for Kafka Connect to start listening on http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors ⏳"  # รอจนกว่า Kafka Connect จะเริ่มทำงาน
        while [ $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) -ne 200 ] ; do  # ตรวจสอบสถานะ HTTP
          echo -e $$(date) " Kafka Connect listener HTTP state: " $$(curl -s -o /dev/null -w %{http_code} http://$$CONNECT_REST_ADVERTISED_HOST_NAME:$$CONNECT_REST_PORT/connectors) " (waiting for 200)"  # แสดงสถานะ
          sleep 5  # รอ 5 วินาที
        done
        nc -vz $$CONNECT_REST_ADVERTISED_HOST_NAME $$CONNECT_REST_PORT  # ตรวจสอบการเชื่อมต่อ
        echo -e "\n--\n+> Creating Kafka Connect MongoDB sink Current PATH ($$PWD)"  # แสดงข้อความเมื่อสร้าง MongoDB sink
        /data/scripts/create_mongo_sink.sh  # เรียกใช้สคริปต์สร้าง MongoDB sink
        echo -e "\n--\n+> Creating MQTT Source Connect Current PATH ($$PWD)"  # แสดงข้อความเมื่อสร้าง MQTT Source
        /data/scripts/create_mqtt_source.sh  # เรียกใช้สคริปต์สร้าง MQTT Source
        echo -e "\n--\n+> Creating Kafka Connect Prometheus sink Current PATH ($$PWD)"  # แสดงข้อความเมื่อสร้าง Prometheus sink
        /data/scripts/create_prometheus_sink.sh  # เรียกใช้สคริปต์สร้าง Prometheus sink
        sleep infinity  # รอไม่สิ้นสุด
    depends_on:
      - zookeeper  # ขึ้นอยู่กับ ZooKeeper
      - kafka      # ขึ้นอยู่กับ Kafka

   MongoDB สำหรับเก็บข้อมูล
  # mongo:
    image: mongo:4.4.20                           # ใช้ภาพ Docker ของ MongoDB
    container_name: mongo                          # ตั้งชื่อคอนเทนเนอร์
    env_file:
      - .env                                       # โหลดตัวแปรสภาพแวดล้อมจากไฟล์ .env
    restart: unless-stopped                        # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    environment:
      - MONGO_INITDB_ROOT_USERNAME=${MONGO_ROOT_USER}  # ชื่อผู้ใช้ราก
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}  # รหัสผ่านราก
      - MONGO_INITDB_DATABASE=${MONGO_DB}          # ชื่อฐานข้อมูลเริ่มต้น

   Grafana สำหรับการวิเคราะห์และแสดงผลข้อมูล
  # grafana:
    image: grafana/grafana:latest-ubuntu         # ใช้ภาพ Docker ของ Grafana
    container_name: grafana                       # ตั้งชื่อคอนเทนเนอร์
    user: '0'                                    # รันด้วยสิทธิ์ผู้ใช้ 0
    volumes:
      - ./grafana/data:/var/lib/grafana          # เชื่อมต่อ volume สำหรับข้อมูล
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards  # เชื่อมต่อสำหรับแดชบอร์ด
      - ./grafana/datasources:/etc/grafana/provisioning/datasources  # เชื่อมต่อสำหรับแหล่งข้อมูล
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}  # ชื่อผู้ใช้ผู้ดูแลระบบ
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}  # รหัสผ่านผู้ดูแลระบบ
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel  # ติดตั้งปลั๊กอิน
      - GF_USERS_ALLOW_SIGN_UP=false                  # ไม่อนุญาตให้ลงทะเบียนผู้ใช้ใหม่
    restart: unless-stopped                           # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    ports:
      - '8085:3000'                                   # เปิดพอร์ต 8085 สำหรับการเข้าถึง Grafana

   Prometheus สำหรับการตรวจสอบและแจ้งเตือน
  # prometheus:
    image: prom/prometheus:latest                   # ใช้ภาพ Docker ของ Prometheus
    container_name: prometheus                      # ตั้งชื่อคอนเทนเนอร์
    volumes:
      - ./prometheus/:/etc/prometheus/              # เชื่อมต่อ volume สำหรับการกำหนดค่า
      - prometheus_data:/prometheus                 # เชื่อมต่อ volume สำหรับข้อมูล
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'  # กำหนดไฟล์การตั้งค่า
      - '--storage.tsdb.path=/prometheus'           # กำหนดที่เก็บข้อมูล
      - '--web.console.libraries=/etc/prometheus/console_libraries'  # กำหนดไลบรารีสำหรับคอนโซล
      - '--web.console.templates=/etc/prometheus/consoles'  # กำหนดเทมเพลตสำหรับคอนโซล
      - '--storage.tsdb.retention.time=200h'        # กำหนดระยะเวลาการเก็บข้อมูล
      - '--web.enable-lifecycle'                     # เปิดใช้งาน lifecycle API
    restart: unless-stopped                         # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    ports:
      - '8086:9090'                                 # เปิดพอร์ต 8086 สำหรับการเข้าถึง Prometheus
        # Exporter for machine metrics    
  # nodeexporter:
    image: prom/node-exporter:v0.18.1  # ใช้ภาพ Docker ของ Node Exporter
    container_name: nodeexporter        # ตั้งชื่อคอนเทนเนอร์
    hostname: nodeexporter              # ตั้งชื่อโฮสต์
    volumes:
      - /proc:/host/proc:ro            # เชื่อมต่อ /proc ของโฮสต์เป็น read-only
      - /sys:/host/sys:ro              # เชื่อมต่อ /sys ของโฮสต์เป็น read-only
    command:
      - '--path.procfs=/host/proc'     # กำหนดที่อยู่ของ procfs
      - '--path.rootfs=/rootfs'         # กำหนดที่อยู่ของ rootfs
      - '--path.sysfs=/host/sys'        # กำหนดที่อยู่ของ sysfs
      - '--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc)($$|/)'  # กำหนดจุดที่ไม่ต้องการตรวจสอบ
    restart: unless-stopped             # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    ports:
      - '9100:9100'                     # เปิดพอร์ต 9100 สำหรับการเข้าถึง Node Exporter

   Kafka exporter for Prometheus
  # kafka-exporter:
    image: bitnami/kafka-exporter:latest  # ใช้ภาพ Docker ของ Kafka Exporter
    container_name: kafka-exporter         # ตั้งชื่อคอนเทนเนอร์
    hostname: kafka-exporter               # ตั้งชื่อโฮสต์
    command:
      - '--kafka.server=kafka:9092'       # ที่อยู่ของ Kafka
      - '--web.listen-address=kafka-exporter:9308'  # ที่อยู่ที่ Kafka Exporter จะฟัง
      - '--web.telemetry-path=/metrics'   # เส้นทางสำหรับการเก็บข้อมูลเมตริก
      - '--log.level=debug'                # ระดับ log
    restart: unless-stopped                # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน

   IoT Sensor 1
  iot_sensor_1:
    # image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT  # ใช้ภาพ Docker ของ IoT Sensor 1
    build:
      context: ./microservices/iot_sensor        # กำหนดที่อยู่ของ Dockerfile
      args:
        - MQTT_SERVER=${MQTT_SERVER}             # ส่งค่า MQTT_SERVER
    container_name: iot_sensor_1                 # ตั้งชื่อคอนเทนเนอร์
    restart: unless-stopped                       # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    environment:
      - sensor.id=${IOT_SENSOR_1_ID}            # รหัสเซ็นเซอร์
      - sensor.name=${IOT_SENSOR_1_NAME}        # ชื่อเซ็นเซอร์
      - sensor.place.id=${IOT_SENSOR_1_PLACE_ID} # รหัสสถานที่
      - sensor.mqtt.username=${IOT_SENSOR_1_USERNAME} # ชื่อผู้ใช้ MQTT
      - sensor.mqtt.password=${IOT_SENSOR_1_PASSWORD} # รหัสผ่าน MQTT
      - MQTT_SERVER=${MQTT_SERVER}               # ส่งค่า MQTT_SERVER

   IoT Sensor 2
  iot_sensor_2:
    image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT  # ใช้ภาพ Docker ของ IoT Sensor 2
    container_name: iot_sensor_2                  # ตั้งชื่อคอนเทนเนอร์
    restart: unless-stopped                        # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    environment:
      - sensor.id=${IOT_SENSOR_2_ID}             # รหัสเซ็นเซอร์
      - sensor.name=${IOT_SENSOR_2_NAME}         # ชื่อเซ็นเซอร์
      - sensor.place.id=${IOT_SENSOR_2_PLACE_ID} # รหัสสถานที่

   IoT Sensor 3
  iot_sensor_3:
    image: ssanchez11/iot_sensor:0.0.1-SNAPSHOT  # ใช้ภาพ Docker ของ IoT Sensor 3
    container_name: iot_sensor_3                  # ตั้งชื่อคอนเทนเนอร์
    restart: unless-stopped                        # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    environment:
      - sensor.id=${IOT_SENSOR_3_ID}             # รหัสเซ็นเซอร์
      - sensor.name=${IOT_SENSOR_3_NAME}         # ชื่อเซ็นเซอร์
      - sensor.place.id=${IOT_SENSOR_3_PLACE_ID} # รหัสสถานที่

   IoT Processor
  iot-processor:
    image: ssanchez11/iot_processor:0.0.1-SNAPSHOT  # ใช้ภาพ Docker ของ IoT Processor
    container_name: iot-processor                     # ตั้งชื่อคอนเทนเนอร์
    restart: unless-stopped                           # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    ports:
      - '8080:8080'                                   # เปิดพอร์ต 8080 สำหรับการเข้าถึง IoT Processor
    depends_on:
      kafka-connect:
        condition: service_started                     # รอจนกว่า Kafka Connect จะเริ่มทำงาน
        restart: true                                  # ให้รีสตาร์ทอัตโนมัติ

   Pushgateway สำหรับ Prometheus
  pushgateway:
    image: prom/pushgateway:v0.8.0                  # ใช้ภาพ Docker ของ Pushgateway
    container_name: pushgateway                      # ตั้งชื่อคอนเทนเนอร์
    restart: unless-stopped                          # ให้รีสตาร์ทอัตโนมัติเมื่อคอนเทนเนอร์หยุดทำงาน
    ports:
      - '9091:9091'                                  # เปิดพอร์ต 9091 สำหรับการเข้าถึง Pushgateway

   Kafka UI สำหรับการจัดการ Kafka
  kafka-ui:
    container_name: kafka-ui                        # ตั้งชื่อคอนเทนเนอร์
    image: provectuslabs/kafka-ui:latest           # ใช้ภาพ Docker ของ Kafka UI
    ports:
      - 18080:8080                                   # เปิดพอร์ต 18080 สำหรับการเข้าถึง Kafka UI
    depends_on:
      - kafka                                        # ขึ้นอยู่กับบริการ Kafka
    environment:
      DYNAMIC_CONFIG_ENABLED: 'true'                # เปิดใช้งานการตั้งค่าแบบไดนามิก
      KAFKA_CLUSTERS_0_NAME: wizard_test            # ชื่อคลัสเตอร์ Kafka
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka:9092 # ที่อยู่ของ Kafka
```

      




## start-service #0
  ในหน้าจอแรก มีการเรียกใช้สคริปต์ start_0zookeeper_kafka.sh ซึ่งสื่อถึงการเปิดใช้งาน ZooKeeper และ Kafka:

  ZooKeeper: ใช้ในการจัดการและทำการซิงโครไนซ์ Kafka brokers
  Kafka: เป็นแพลตฟอร์มการส่งข้อความที่ช่วยในการจัดการและสตรีมข้อมูลจาก IoT devices

>>service
zookeeper
kafka

## start-service #1
ในหน้าจอที่สอง มีการเรียกใช้สคริปต์ start_1kafka_server.sh ซึ่งเป็นการเปิดใช้งาน Kafka Server เพื่อให้สามารถส่งและรับข้อมูลจาก clients ผ่าน topics ต่าง ๆ ที่ 
 กำหนดไว้ใน Kafka

>>service
kafka-rest-proxy
kafka-connect
mosquitto
mongo
grafana
prometheus

## start-service #2
หน้าจอที่สาม ใช้สคริปต์ start_2iot_processor.sh เพื่อเปิดการทำงานของ IoT Processor ซึ่งอาจเป็นบริการที่ทำหน้าที่ประมวลผลข้อมูลที่ถูกส่งเข้ามาผ่าน Kafka และทำการวิเคราะห์หรือดำเนินการกับข้อมูล IoT ที่ได้รับมา

>>service
iot-processor

## start-service #3
ในหน้าจอที่สี่ ใช้สคริปต์ start_3iot_sensor.sh เพื่อเปิดการทำงานของ IoT Sensor Processor ซึ่งอาจเป็นบริการที่รับข้อมูลจากเซนเซอร์ IoT และทำการประมวลผลเบื้องต้นก่อนที่จะส่งต่อไปยัง Kafka สำหรับการประมวลผลเพิ่มเติม

>>service
iot_sensor



![image](https://github.com/user-attachments/assets/1b3b68cc-68d3-48c0-be73-f4f76a7cedf0)
