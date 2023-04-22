## Prerequisites

- HPE Swarm Learning is qualified with Docker 20.10.5. Make sure to have docker installed on you system
## Configure Docker to run as a non-root user.

- Create docker group
```bash
sudo groupadd docker
```

- Add your user to docker group
```bash
sudo usermod -aG docker $USER
```

- Logout and logback in so that your group membership is re-evaluated. If you’re running Linux in a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.
```bash
newgrp docker
```

- Verify that you can run `docker` commands without `sudo`
```bash
docker run hello-world
```

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

- To stop this behaviour u can run
```bash
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```

- Switch to Desktop
```bash
cd Desktop
```
- Clone the repo in the Desktop
```bash
git clone https://github.com/HewlettPackard/swarm-learning.git
```

- Set a password for su if you haven't done it before
```
sudo passwd
```

## Configuring network settings for Docker

-Create a systemd drop-in directory for the docker service:

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

- Create a file named `/etc/systemd/system/docker.service.d/http-proxy.conf` that adds `HTTP_PROXY` env variable and add the following line
`[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"` in the text editor

```bash 
gedit /etc/systemd/system/docker.service.d/http-proxy.conf
```

- Flush changes and restart docker

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

- Verify that the config has been loaded

```bash
sudo systemctl show --property=Environment docker
Environment=HTTP_PROXY=http://proxy.example.com:80 HTTPS_PROXY=https://proxy.example.com:443 NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
```

## How to configure to use IPV4

```bash
 docker run --publish "0.0.0.0:80:80" --publish "0.0.0.0:443:443" --detach nginx
```

- Installation of openssh server

```bash
sudo apt install openssh-server
```

## For VirtualBox users

- Create VM of 8 cores and 8 GB RAM, 150GB storage
- Install docker using the following command: `sudo apt install docker.io`

