# IoT Docker compose


## How to start docker compose

    #เริ่มต้นการทำงานของบริการที่คุณกำหนดไว้ในไฟล์ docker-compose.yml ในโหมด foreground:
    "docker-compose up"

    #ตรวจสอบสถานะและดูคอนเทนเนอร์ที่กำลังทำงาน(คำสั่งนี้จะแสดงสถานะของแต่ละบริการที่กำหนดใน docker-compose.yml รวมถึงพอร์ตและชื่อคอนเทนเนอร์ด้วย.)
    "docker-compose ps"


## Error we found
    
    #เกิด error หลายตัวมากๆ โดยหลายตัวก็ไม่ได้ใช้
    #มีข้อผิดพลาดในการตั้งค่าบางอย่างในไฟล์ docker-compose.yml
    #เกิด error ตอนจะเปิดหน้า sensor คือมันไม่สามารถ compose-up ได้ หน้า kafna ไม่ขึ้นอะไรเลย
    #เกิดปัญหา ip มันชนกันและไม่เสถียรมากๆ ชอบหลุด

## How to solve the problems.

    #เกิด error ตัวไหนเพื่อนสอนโดยการลบตัวนั้นออกไปเลย
    #ทดสอบการกำหนดค่าด้วย docker-compose config หรือ docker-compose config --services
    #เราต้องทำการ Stop all container และ Delete all containers, Delete all volumes, Delete network และใช้คำสั่ง sh   start_00 และกด tab มันจะได้ตัวที่เราต้องการ ทำไปทั้งหมดถึงตัว 03 จากนั้นให้ไปลองเปิดตัว kafna ดูมันจะหน้า kafna ปกติได้
    #เราเลยทำการเปลี่ยนและแบ่ง server กันใช้ในแต่ละกลุ่ม โดยเราจะเข้า rootของเราและ cd เข้า /etc/netplan/ และ vim เข้าไฟล์ 00-installer-config.yml และเข้าไปเปลี่ยนเป็น ip ที่เราต้องการจากนั้นใช้คำสั่ง cat 00-installer-config.yaml
        นี่คือสิ่งที่ได้: 
        network:
            ethernets:
                enp1s0:
                dhcp4: false
                addresses: [172.16.46.55/24]
                nameservers:
                    addresses: [172.16.46.254, 8.8.8.8]
                routes:
                    - to: default
                        via: 172.16.46.254
                enp2s0:
                dhcp4: true
            version: 2
    จากนั้นใช้คำสั่ง netplan try และลองเข้า ubuntu ของเราได้เลย

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