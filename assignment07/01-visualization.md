# Data Visualization.

1. ก่อนที่เราจะทำการเพิ่ม Data Visualization หรือการเพิ่ม dashboard นั้นอย่างแรกเราต้องทำการลง plugin ที่เราจะใช้ก่อน
2. เข้าไปใน link github ที่อยู่ จะเจอ link ของ sky frank แล้เราทำการ fork มา

   ![image](https://github.com/user-attachments/assets/bb619ad2-82de-4e14-9cfe-426a6c084de5)

4. จากนั้นทำการโหลดไฟล์ที่ชื่อว่า agenty-.........

   ![image (1)](https://github.com/user-attachments/assets/9f8b1f51-776d-406b-b5b1-7e02d18a12b0)

4.แล้วเอาไฟล์ zip นั้นมาแตกไฟล์ด้วยคำสั่ง
  docker cp agenty-flowcharting-panel-1.0.0e.231214594-SNAPSHOT.zip grafana:/

5.เอา agenty flowcharting ของ skyfrank มาลงใน plugin และไปเเก้ใข docker compose ใน หัวข้อของ grafana เป็นเเบบนี้

   ![image](https://github.com/user-attachments/assets/f66b87e9-1364-409a-8417-95a759f7be67)

6. พอลงเสร็จก็ docker compose restart
7. และทุกครั้งที่จะใช้ เราจะ ssh เข้า server ของเราและทำการ docker compose stop grafana
   และใช้คำสั่ง docker compose up grafana

   
