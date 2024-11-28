# Creating 3d photogrammetry using a Drone

## SOFTWARE INSTALLATION

### OpenDroneMap
https://www.opendronemap.org/webodm/

### Installation server side Component
The server needs to have an NVIDIA GPU
#### Install docker on Fedora 40 and 41:
https://docs.docker.com/engine/install/fedora/

##### Uninstall old versions
```commandline
sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```
##### Set up the repository
```commandline
sudo dnf -y install dnf-plugins-core
sudo dnf-3 config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```
##### Install Docker Engine
```commandline
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
##### Start Docker Engine.
```commandline
sudo systemctl enable --now docker
```

#### Use a different disk for container storage
```commandline
sudo mkfs.ext4 /dev/nvme1n1
sudo mkdir /mnt/nvme1
sudo mount /dev/nvme1n1 /mnt/nvme1r
echo "/dev/nvme1n1 /mnt/nvme1 ext4 defaults 0 0" | sudo tee -a /etc/fstab
```

```bash
sudo systemctl stop docker
```

```bash
sudo mkdir -p /mnt/nvme1/docker
```

```commandline
sudo vi /etc/docker/daemon.json
put this:
{
  "data-root": "/mnt/nvme1/docker/docker-data"
}
```

ensuring that all the users in the group docker can access docker..

```commandline
sudo mkdir -p /mnt/nvme1/docker/docker-data
sudo chown -R root:docker /mnt/nvme1/docker/docker-data
sudo chmod -R 770 /mnt/nvme1/docker/docker-data
sudo find /mnt/nvme1/docker/docker-data -type d -exec chmod 770 {} \;

```

```commandline
sudo systemctl stop docker
sudo systemctl start docker
sudo systemctl enable docker
```

Verify docker configuration
```commandline
sudo docker info | grep "Docker Root Dir"
```

```
 Docker Root Dir: /mnt/docker/docker-data
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```

### Manage Docker as a non-root user
#### Create the docker group.
```
sudo groupadd docker
```
#### Add your user to the docker group.
```
sudo usermod -aG docker $USER
```
#### Verify that you can run docker commands without sudo.


#### Add current user to docker group 
```commandline
sudo usermod -aG docker $USER
sudo systemctl restart docker
```

## Getting started with NodeODM

#### Creating a folder to store the images for NodeODM
```commandline
sudo mkdir -p /mnt/nvme1/docker-data/nodeodm-data
sudo chmod -R 770 /mnt/nvme1/docker-data/nodeodm-data
sudo chown -R root:docker /mnt/nvme1/docker-data/nodeodm-data
```
#### install nvidia container toolkit

```commandline
sudo dnf install -y nvidia-container-toolkit
sudo systemctl restart docker
```

#### Configure Docker for GPU Support:
```commandline
sudo tee /etc/docker/daemon.json <<EOF
{
  "data-root": "/mnt/nvme1/docker/docker-data",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
EOF
```
check if NVIDIA GPU is working within the docker:
```commandline
docker run --rm --gpus all  nvidia/cuda:11.8.0-runtime-ubuntu22.04 nvidia-smi
```

#### prepare the NodeODM    
```commandline
sudo docker run -d --restart unless-stopped --gpus '"device=0"' -p 3000:3000 -v /mnt/nvme1/docker-data/nodeodm-data:/var/www/data --user root opendronemap/nodeodm:gpu --max-concurrency 20

or to use all GPus
sudo docker run -d --restart unless-stopped --gpus all -p 3000:3000 -v /mnt/nvme1/docker-data/nodeodm-data:/var/www/data --user root opendronemap/nodeodm:gpu --max-concurrency 10


```
####

#### check the image
```
(base) cristianku@localhost:~$ docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS                          PORTS     NAMES
81ab73f336e7   opendronemap/nodeodm:gpu   "/usr/bin/node /var/…"   About a minute ago   Restarting (1) 23 seconds ago             gracious_bardeen
```

### WEBODM directly installed in the server

### WEBODM ( web interface on the Desktop Computer)
#### Install docker desktop for your OS, in my case MacOs
https://docs.docker.com/desktop/setup/install/mac-install/

#### You can define a specific folder for the export
![images/dockerdesktop1.png](images/dockerdesktop1.png)


```commandline
git clone https://github.com/OpenDroneMap/WebODM --config core.autocrlf=input --depth 1
cd WebODM
./webodm.sh start
```

#### Now you can access your WebODM:
```commandline
http://localhost:8000

```
![images/webodm_web.png](images/webodm_web.png)

#### Now we add the remote processing node

![images/processingnode.png](images/processingnode.png)