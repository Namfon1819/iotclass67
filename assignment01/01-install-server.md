# Install Server and Docker


## How to install Server

Install Ubuntu 24.04 (LTS) on server
1. Select Your Language
Choose your preferred language.

Example: [ English ]
2. Installer Update Available
Skip the update for now.

Select: [ Continue without updating ]
3. Keyboard Configuration
If the default keyboard layout is correct, proceed.

Select: [ Done ]
4. Choose the Type of Installation
Choose the minimized Ubuntu Server installation.

Option: (X) Ubuntu Server (minimized)
Select: [ Done ]
5. Network Configuration
Accept the default network configuration.

Select: [ Done ]
6. Proxy Configuration
If you don't use a proxy, leave this blank.

Select: [ Done ]
7. Ubuntu Archive Mirror Configuration
Accept the default mirror settings.

Select: [ Done ]
8. Guided Storage Configuration
Choose to create a custom storage layout.

Option: (X) Custom storage layout
Select: [ Done ]
9. Storage Configuration
Configure your partitions as follows:

Mount Boot Partition:

Use free space to create a new GPT partition.
Size: 2G
Format: [ ext4 ]
Mount: [ /boot ]
Select: [ Create ]
Mount Swap Partition:

Use free space to create a new GPT partition.
Size: 2G
Format: [ swap ]
Select: [ Create ]
Mount Root Partition:

Use the remaining free space to create a new GPT partition.
Size: MAX
Format: [ ext4 ]
Mount: [ / ]
Select: [ Create ]
File System Summary:
MOUNT POINT	SIZE	TYPE	DEVICE TYPE
/	MAX	ext4	partition of localdis
/boot	2.0 G	ext4	partition of localdis
/SWAP	2.0 G	swap	partition of localdis
Select: [ Done ]
Confirm the destructive action.
Select: [ Continue ]
10. Profile Configuration
Fill in the following fields to set up your server profile:

Your name: Enter your full name.
Your server name: Choose a name for your server.
Pick a username: Choose a username for your account.
Choose a password: Create a secure password.
Confirm your password: Re-enter your password to confirm.
Select: [ Done ]
11. Upgrad to Ubunto Pro
If prompted, select the option for upgrading to Ubuntu Pro.

Option: (X) Custom storage layout
Select: [ Continue ]
12. SSH Configuration
Install the OpenSSH server to enable remote access.

Option: (X) Install OpenSSH server
Select: [ Done ]
13. Featured server snaps
Skip the installation of featured snaps.

Select: [ Continue ]
14. Installation Complete!
Once the installation is finished, your server is ready to use.

Select: [ Reboot Now ]
(SET IP)

1. install nano
2. เข้า nano  ใช้คำสั่ง sudo nano /etc/netplan/00-installer-config.yaml
3. เข้าไป setup ip 
4. จากนั้นไปพิมพ์ sudo netplan apply
5. ssh เข้า ubuntu ในเครื่องเรา
``` cpp
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
```
