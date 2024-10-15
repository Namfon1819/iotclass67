# Data Visualization.
#การลง Flowcharting

![image](https://github.com/user-attachments/assets/1c9cefa8-5069-4b84-8ed0-1154c8f2dadc)

1) เปิดหน้่า grafana ขึ้นมาและเข้า plugin ค้นหา floecharting และทำการเลื่อนลงไปข้างล่างเพื่อเข้าไปใน link นี้

![image](https://github.com/user-attachments/assets/e14f68a7-7746-4c77-8cfd-785528d0adfc)

2) พอเข้าไปใน link github จะเจอของที่ sky frank fork ไว้

![image](https://github.com/user-attachments/assets/b0255545-e9ad-4de6-8ce5-8360981ab262)

3) จากนั้นให้เราโหลดไฟล์ Zip ออกมา
4) และเอาไฟล์ไปใส่ไว้ใน IOT ของเราที่เราตั้งชื่อไว้/grafana/data/plugnin/src
5) แตกไฟล์ด้วยคำสั่ง unzip agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip
6) จากนั้นลบ docker image volume container ทุกอย่าง
7) สร้าง container ของ iot ใหม่ทั้งหมดตั้งเเต่ต้น โดยการพิมพ์ตาม start … .sh เเต่ละตัวจนครบ (ตัว sensor ไม่ต้องก็ได้ เพราะว่าไม่เกี่ยวกัน มันเเค่การรับค่า iot มาจาก sensor)
8) เอา agenty flowcharting ของ skyfrank มาลงใน plugin
9) ไปเเก้ใข docker compose ใน หัวข้อของ grafana เป็นเเบบนี้
    ```cpp
    grafana:
    image: grafana/grafana:latest-ubuntu
    container_name: grafana
    user: '0'
    volumes:
      - ./grafana/data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    environment:
      - GF_SECURITY_ADMIN_USER=${ADMIN_USER:-admin}
      - GF_SECURITY_ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin}
      - GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-worldmap-panel,grafana-piechart-panel
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_SECURITY_ANGULAR_SUPPORT_ENABLED=true
      - GF_FEATURE_TOGGLES_ANGULARDEPRECATIONUI=false
    restart: unless-stopped
    links:
       - prometheus
    ports:
      - '8085:3000'
   ```
10) พอลงเสร็จก็ docker compose restart


#การสร้าง map visualization
1) เข้าหน้า edit panal จากนั้นก็กดตรง edit diagram
   ![image](https://github.com/user-attachments/assets/4fd98489-76ac-42b3-811a-089a052d4b73)
   
2) จากนั้นให้เราไปบันทึกหรือหารูปแบบ plan ที่เราจะทำมา แล้วมา  export plan ออกมาเป็น svg เพื่อเอาไปใช้ในขั้นตอนต่อไป
3) จากนั้นให้เราใช้กล่อง text สร้าง ชื่อกล่องว่า room1-10 ไปวางตามห้องต่างๆ
4) จากนั้น เราก็จะลงไปล่างสุดกดตรงปุ่ม explan เพื่อดูว่าห้องแต่ละห้องที่เราสร้างตรงกับเลขอะไร
5) จากนั้นเราก็จะมาสร้าง rule ให้แต่ละห้อง
   ![image](https://github.com/user-attachments/assets/c6bcb75e-d46a-40ff-a0ba-a1425786ccc1)

6) แต่ละ rule เราจะมีการ metric type ละมีการเลือกชื่อของ sensor ว่าห้องนี่จะให้ sensor ตัวไหนบอกอุณหภูมิ และมีการกำหนดอุณภูมิและสีว่าถ้าอุณหภูมิเกิน 28 ให้เปลี่ยนเป็นสีแดง
7) ต่อมามาเปลี่ยนหัวข้อ query ตรง option ตอนเเรกมันเป็น json  ให้ลบออก เเล้ว extract เเต่ sensor name ตามในรูป
   ![image](https://github.com/user-attachments/assets/be939980-7f5a-42f5-98a0-afd3bc14a904)

8) ให้ใส่ข้างบนเป็น {{sensor_name}} เพราะว่าจะได้ง่ายต่อการใช้งานตอนเลือกชื่อในช่อง rule 
9) จากนั้นก็กด save ด้วยน่ะเดี๋ยวที่สร้างมาหายหมด
    
# นำข้อมูลอะไรมาแสดงในส่วนของ Visualization บ้าง

1. ข้อมูลในส่วนแผนผังบ้านหรือสวนต่างๆที่เราออกแบบและวาง sensor ไว้
2. ข้อมูลของ IOT Sensor Humidity
3. ข้อมูลของ IOT Sensor AVG Temperature
4. ข้อมูลของ IOT Sensor Luminosity
5. ข้อมูลของ IOT Sensor Pressure
   
![Screenshot 2024-08-28 164621](https://github.com/user-attachments/assets/fa580613-0962-434d-94dd-eb0da87dea05)

![Screenshot 2024-08-28 164627](https://github.com/user-attachments/assets/bd2ac6aa-a64e-46e3-a1b3-649f8ca0a252)


----IoT Sensor Humidity:------

แสดงค่าความชื้นสัมพัทธ์จากเซ็นเซอร์ IoT หลายตัว โดยใช้ Time-series graph (กราฟเส้นตามเวลา)
เซ็นเซอร์ที่แสดงข้อมูล: iot_sensor_1, iot_sensor_2, … , iot_sensor_10
กราฟเส้นแสดงการเปลี่ยนแปลงของความชื้นในแต่ละช่วงเวลา
เหตุผลในการใช้: กราฟนี้ช่วยแสดงการเปลี่ยนแปลงของความชื้นตามเวลาจริง (real-time) เหมาะสำหรับการติดตามแนวโน้มของค่าความชื้นในสถานที่ที่ใช้เซ็นเซอร์หลายตัว

----IoT Sensor AVG Temperature:------

แสดงค่าอุณหภูมิเฉลี่ยจากเซ็นเซอร์หลายตัวในรูปแบบของ Gauge Chart (กราฟเกจ)
เกจแสดงเป็นโซนสีเขียว (ค่าปกติ) และโซนสีแดง (เกินขอบเขตปกติ)
เหตุผลในการใช้: Gauge Chart แสดงค่าในรูปแบบที่เข้าใจง่าย โดยการใช้สีเพื่อให้เห็นชัดเจนว่าค่าอุณหภูมิอยู่ในช่วงที่เหมาะสมหรือไม่

----IoT Sensor Luminosity:------

แสดงค่าความสว่าง (Luminosity) จากหลายเซ็นเซอร์ในรูปแบบของ Bar Graph (กราฟแท่ง)
เหตุผลในการใช้: กราฟแท่งเหมาะสำหรับการแสดงค่าที่ต้องการเปรียบเทียบระหว่างเซ็นเซอร์ต่างๆ ในช่วงเวลาเดียวกัน

----IoT Sensor Pressure:-----

แสดงค่าความดันอากาศจากเซ็นเซอร์ในรูปแบบของ Time-series graph และ Gauge Chart
มีกราฟเส้นแสดงการเปลี่ยนแปลงของค่าความดันตามเวลาจริง และ Gauge Chart แสดงค่าเป็น bar/mbar
เหตุผลในการใช้: การใช้กราฟเส้นช่วยให้ติดตามการเปลี่ยนแปลงของค่าความดันในแต่ละเซ็นเซอร์อย่างต่อเนื่อง ขณะที่ Gauge Chart ช่วยให้เห็นภาพรวมของค่าความดันปัจจุบันอย่างชัดเจน
