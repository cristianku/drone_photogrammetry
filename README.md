# Creating 3d photogrammetry using a Drone

## SOFTWARE INSTALLATION

### OpenDroneMap
https://www.opendronemap.org/webodm/

### Installation server side Component
The server needs to have an NVIDIA GPU
#### Install docker on Fedora:
```commandline
sudo dnf update -y
sudo dnf config-manager --add-repo=https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
#### Use a different disk for container storage
```commandline
sudo mkfs.ext4 /dev/nvme1n1
sudo mkdir /mnt/docker-data
sudo mount /dev/nvme1n1 /mnt/docker-data
echo "/dev/nvme1n1 /mnt/docker-data ext4 defaults 0 0" | sudo tee -a /etc/fstab
```

```commandline
sudo vi /etc/docker/daemon.json
put this:

{
  "data-root": "/mnt/docker-data/docker"
}
```

```commandline
sudo mkdir -p /mnt/docker-data/docker
sudo chown -R root:docker /mnt/docker-data/docker

```

```commandline
sudo systemctl start docker
sudo systemctl enable docker
```

Verify docker configuration
````commandline
sudo docker info | grep "Docker Root Dir"

 Docker Root Dir: /mnt/docker-data/docker
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled

````

#### Add current user to docker group 
```commandline
sudo usermod -aG docker $USER
sudo systemctl restart docker
```

## Gettin started with NodeODM

#### Creating a folder to store the images for NodeODM
```commandline
sudo mkdir -p /mnt/docker-data/nodeodm-data
sudo chmod -R 770 /mnt/docker-data/nodeodm-data
sudo chown -R $USER:docker /mnt/docker-data/nodeodm-data
```

check if NVIDIA GPU is working within the docker:
```commandline
docker run --rm --gpus all nvidia/cuda:11.8.0-runtime-ubuntu22.04 nvidia-smi
```

```commandline
sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

install the NVIDIA Container Toolkit
```commandline
docker run --rm --gpus all nvidia/cuda:11.8.0-runtime-ubuntu22.04 nvidia-smi

```

```commandline
docker run -d --restart unless-stopped --gpus all -p 3000:3000 -v /mnt/docker-data/nodeodm-data:/var/www/data opendronemap/nodeodm:gpu
```