# Data Visualization.

1. ก่อนที่เราจะทำการเพิ่ม Data Visualization หรือการเพิ่ม dashboard นั้นอย่างแรกเราต้องทำการลง plugin ที่เราจะใช้ก่อน
2. เข้าไปใน link github ที่อยู่ จะเจอ link ของ sky frank แล้เราทำการ fork มา
   ![image](https://github.com/user-attachments/assets/bb619ad2-82de-4e14-9cfe-426a6c084de5)
3. จากนั้นทำการโหลดไฟล์ที่ชื่อว่า agenty-.........
   ![image (1)](https://github.com/user-attachments/assets/9f8b1f51-776d-406b-b5b1-7e02d18a12b0)
4.แล้วเอาไฟล์ zip นั้นมาแตกไฟล์ด้วยคำสั่ง
  docker cp agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip grafana:/

5.เอา agenty flowcharting ของ skyfrank มาลงใน plugin และไปเเก้ใข docker compose ใน หัวข้อของ grafana เป็นเเบบนี้

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
      ## i add this to anable flowcharting
      - GF_SECURITY_ANGULAR_SUPPORT_ENABLED=true
      - GF_FEATURE_TOGGLES_ANGULARDEPRECATIONUI=false
    restart: unless-stopped
    links:
       - prometheus
    ports:
      - '8085:3000'

6. พอลงเสร็จก็ docker compose restart
7. และทุกครั้งที่จะใช้ เราจะ ssh เข้า server ของเราและทำการ docker compose stop grafana
   และใช้คำสั่ง docker compose up grafana

   
