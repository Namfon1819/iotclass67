# Install Server and Docker


## How to install Server
-download ubuntu version 22.04 
-เอา flash drive มาเสียบ gateway และลง ubuntu


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
To install the latest version, run:
    "sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin"

Lets run hello-world image to verify the installation:
    "sudo docker run hello-world"

Add the docker group (it might be already exist):
    "sudo groupadd docker"

Add the connected user “$USER” to the docker group:
    "sudo gpasswd -a $USER docker"

Either do a newgrp docker or log out/in to activate the changes to groups:
    "newgrp docker"

Run docker run hello-world command to test it:
    "docker run hello-world"
    
