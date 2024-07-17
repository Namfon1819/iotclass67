# IoT Docker compose


## How to start docker compose

    #เริ่มต้นการทำงานของบริการที่คุณกำหนดไว้ในไฟล์ docker-compose.yml ในโหมด foreground:
    "docker-compose up"

    #ตรวจสอบสถานะและดูคอนเทนเนอร์ที่กำลังทำงาน(คำสั่งนี้จะแสดงสถานะของแต่ละบริการที่กำหนดใน docker-compose.yml รวมถึงพอร์ตและชื่อคอนเทนเนอร์ด้วย.)
    "docker-compose ps"


## Error we found
    
    #เกิด error หลายตัวมากๆ โดยหลายตัวก็ไม่ได้ใช้
    #มีข้อผิดพลาดในการตั้งค่าบางอย่างในไฟล์ docker-compose.yml
    #เกิด error ตอนจะเปิดหน้า sensor คือมันไม่สามารถ compose-up ได้

## How to solve the problems.

    #เกิด error ตัวไหนเพื่อนสอนโดยการลบตัวนั้นออกไปเลย
    #ทดสอบการกำหนดค่าด้วย docker-compose config หรือ docker-compose config --services
    #
## Output

- [/] IoT Sensor - Dashboards - Grafana  
- [ ] UI for Apache Ka
- [ ] Mongo Expr
- [ ] Node Expor
- [ ] Prometheus Time Series Collection and Processing Ser
- [ ] Prometheus Pushgateway
- [ ] ZooNavigator


### IoT Sensor - Dashboards - Grafana URL

### UI for Apache Kafka

### Mongo Express

### Node Exporter

### Prometheus Time Series Collection and Processing Server

### Prometheus Pushgateway

### ZooNavigator