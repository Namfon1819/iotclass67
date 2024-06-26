# Install Server and Docker


## How to install Server
1. กด F12 เพื่อ  Boot mode is set to: UEFI: Secure Boot : ON
    เลือก  UEFI: SanDisk
2. จากนั้นหน้าต่างถัดไปเลือก try on Install Ubuntu Server
3. หน้าที่ให้เลือกภาษา เราเลือกภาษา English
4. ในหน้าถัดไปเราเลือก continues และในหน้าต่อไปเลือก Done ได้เลย และหน้าถัดไปก็กด Done ได้เลยเหมือนกัน
5. ในหน้า choose the base for the installation .
    เลือก Ubuntu Server (minimized)
6. ในหน้า enp เราควรถ่าย ip ของเราไว้(มันจะขึ้นถ้าเราเชื่อมเน็ตได้แล้ว)
7. ใน 2 หน้าถัดไปกด Done ได้เลย
8. ในหน้า Configure a guide storage layout 
    เลือก Custom storage layout
9. ในหน้า storage configuration 
    ให้ลบ available devices ออกให้หมด  และกดเลือก free space เพื่อ add File system summary ไป 2 ตัว คือ
    1> swap 1.000G
    2> / ext4 
10. ในหน้าต่อไปก็เป็นการตั้ง username ต่างๆและ passwords
11. ในหน้าถัดไป หน้า upgrad ให้กด Continues ได้เลย
12. ในหน้า choose to install the openSSH
    เลือก Install OpenSSH server
13. ในหน้า Featured Server Snaps ไม่ต้องเลือกอะไรเลย กด Done ได้เลย
14. จากนั้นรอมัน run เสร็จรอให้ขึ้น reboot อย่างเดียวและเลือก reboot ได้เลย

## How to install Docker
1. Set up the Docker apt repository.
    # Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

    # Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

2. Install the Docker packages.
    #To install the latest version, run:
    "sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin"

    #Lets run hello-world image to verify the installation:
    "sudo docker run hello-world"

    #Add the docker group (it might be already exist):
    "sudo groupadd docker"

    #Add the connected user “$USER” to the docker group:
    "sudo gpasswd -a $USER docker"

    #Either do a newgrp docker or log out/in to activate the changes to groups:
    "newgrp docker"

    #Run docker run hello-world command to test it:
    "docker run hello-world"

