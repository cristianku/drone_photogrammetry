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
change the group
```yaml
sudo chown -R root:docker /mnt/nvme1/docker
```
automatically inherit the group ownership of the parent directory.
```yaml
sudo chmod g+s /mnt/nvme1/docker
```
access rights
```yaml
sudo chmod -R 770 /mnt/nvme1/docker
```

add the docker-data folder to the docker configuration , so that it will use the nvme 
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

```

```commandline
sudo systemctl start docker
sudo systemctl enable docker
```

Verify docker configuration
```commandline
sudo docker info | grep "Docker Root Dir"
```
```
 Docker Root Dir: /mnt/docker/docker-data
```

### Manage Docker as a non-root user
#### Create the docker group.
#### Add your user to the docker group.
```
sudo usermod -aG docker $USER
sudo systemctl restart docker
```

## Getting started with NodeODM

#### Creating a folder to store the images for NodeODM
```commandline
sudo mkdir -p /mnt/nvme1/docker/docker-app-data
sudo chmod -R 770 /mnt/nvme1/docker/docker-app-data
sudo chown -R root:docker /mnt/nvme1/docker/docker-app-data
```

```
sudo mkdir -p /mnt/nvme1/docker/docker-app-data/nodeodm-data
sudo chmod -R 770 /mnt/nvme1/docker/docker-app-data
sudo chown -R root:docker /mnt/nvme1/docker/docker-app-data
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
sudo docker run -d --restart unless-stopped --gpus '"device=1"' -p 3000:3000 -v /mnt/nvme1/docker/docker-app-data/nodeodm-data:/var/www/data --user root opendronemap/nodeodm:gpu --max-concurrency 20

or to use all GPus
sudo docker run -d --restart unless-stopped --gpus all -p 3000:3000 -v /mnt/nvme1/docker/docker-app-data/nodeodm-data:/var/www/data --user root opendronemap/nodeodm:gpu --max-concurrency 10
```

or create a script:
```yaml
vi create_nodeodm.sh
```
add:
```yaml
#!/bin/bash
docker stop $(docker ps -a -q --filter "ancestor=opendronemap/nodeodm:gpu")
docker rm $(docker ps -a -q --filter "ancestor=opendronemap/nodeodm:gpu")
sudo rm -rf /mnt/nvme1/docker/docker-app-data/nodeodm-data/*

sudo docker run -d --restart unless-stopped --gpus '"device=1"' -p 3000:3000 -v /mnt/nvme1/docker/docker-app-data/nodeodm-data:/var/www/data --user root opendronemap/nodeodm:gpu --max-concurrency 20
```


####

#### check the image
```
(base) cristianku@localhost:~$ docker ps
CONTAINER ID   IMAGE                      COMMAND                  CREATED              STATUS                          PORTS     NAMES
81ab73f336e7   opendronemap/nodeodm:gpu   "/usr/bin/node /var/…"   About a minute ago   Restarting (1) 23 seconds ago             gracious_bardeen
```

### WEBODM directly installed in the server
[webOdm.md](webOdm.md)

