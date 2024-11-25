# Open Air Interface 5G

This document can be used to deploy an Open Air Interface (OAI) core network using Docker on a VirtualBox (7.1.4) VM running a Linux system (Ubuntu 22.04.5). Once deployed, you can test the OAI 5G core network with oai-gNB.

## Pre-requisites
| Software | Version |
| ----------- | ----------- |
| VirtualBox | 7.1.4 |
| docker engine | 27.3.1 |
| docker-compose | 1.29.2 |
| Host OS        | Windows 10 Home 64-bit (10.0, Build 19045) |
| VM OS | Ubuntu 22.04.5 LTS (Jammy Jellyfish) |
| wireshark | 4.4.0 |
### 1. Install Docker Engine

**i.** Set up Docker's `apt` repository:

```bash
# Add Docker's official GPG key:

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

**ii.** Install the specific version of Docker Engine:
```bash
# List the available versions:
$ apt-cache madison docker-ce | awk '{ print $3 }'

5:27.3.1-1~ubuntu.22.04~jammy
5:27.3.0-1~ubuntu.22.04~jammy
...
```
Select the desired version and install:
```bash
 $ VERSION_STRING=5:27.3.1-1~ubuntu.22.04~jammy

 $ sudo apt-get install docker-ce=$VERSION_STRING docker-ce-cli=$VERSION_STRING containerd.io docker-buildx-plugin docker-compose-plugin
```

**iii.** Verify successful installation by running `hello-world` image:
```bash
$ sudo docker run hello-world
```

<mark>Insert image for terminal output for hello-world container</mark>

### 2. Install docker-compose
**i.** Run:
```bash
$ sudo apt-get update
$ sudo apt-get install docker-compose
```
**ii.** Verify succesful installation:
```bash
$ docker-compose version

docker-compose version 1.29.2
```

### 3. Installing wireshark
```bash
$: sudo add-apt-repository ppa:wireshark-dev/stable
$: sudo apt update
$: sudo apt install wireshark

$: wireshark --version
Wireshark 4.4.0.
```

## Setting up the OAI 5G Core environment
### 1. Create an account on Docker Hub
Link to [Docker Hub](https://hub.docker.com/).
### 2. Pull base images
**i.** First login with your new Docker Hub details:
```bash
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username:
Password:
```

**ii.** Run:
```bash
$ docker pull ubuntu:jammy
$ docker pull mysql:8.0

$ docker pull oaisoftwarealliance/oai-amf:v2.1.0
$ docker pull oaisoftwarealliance/oai-nrf:v2.1.0
$ docker pull oaisoftwarealliance/oai-upf:v2.1.0
$ docker pull oaisoftwarealliance/oai-smf:v2.1.0
$ docker pull oaisoftwarealliance/oai-udr:v2.1.0
$ docker pull oaisoftwarealliance/oai-udm:v2.1.0
$ docker pull oaisoftwarealliance/oai-ausf:v2.1.0
$ docker pull oaisoftwarealliance/oai-upf-vpp:v2.1.0
$ docker pull oaisoftwarealliance/oai-nssf:v2.1.0
$ docker pull oaisoftwarealliance/oai-pcf:v2.1.0
$ docker pull oaisoftwarealliance/oai-lmf:v2.1.0
# Utility image to generate traffic
$ docker pull oaisoftwarealliance/trf-gen-cn5g:latest

```

**iii.** You may logout at this stage:
```bash
$ docker logout
```

**iv.** Synchronise the components:
```bash
# Clone directly on the latest release tag
$ git clone --branch v2.1.0 https://gitlab.eurecom.fr/oai/cn5g/oai-cn5g-fed.git
$ cd oai-cn5g-fed
# If you forgot to clone directly to the latest release tag
$ git checkout -f v2.1.0

# Synchronize all git submodules
$ ./scripts/syncComponents.sh
---------------------------------------------------------
OAI-NRF     component branch : master
OAI-AMF     component branch : master
OAI-SMF     component branch : master
OAI-UPF     component branch : master
OAI-AUSF    component branch : master
OAI-UDM     component branch : master
OAI-UDR     component branch : master
OAI-UPF-VPP component branch : master
OAI-NSSF    component branch : master
OAI-NEF     component branch : master
OAI-PCF     component branch : master
OAI-LMF     component branch : master
---------------------------------------------------------
git submodule deinit --all --force
git submodule init
git submodule update
```
### 3. Network Configuration
**i.** Run:
```bash
$ sudo sysctl net.ipv4.conf.all.forwarding=1
$ sudo iptables -P FORWARD ACCEPT
```

**ii.** The `demo-oai` bridge connecting the network functions is built automatically using docker-compose.

If you wish to capture initial packets, you will have to configure the bridge manually (not covered in this document).

The `default` version is in the `docker-compose-basic-nerf.yaml` file:

```yaml
...
networks:
    # public_net:
    #     external:
    #         name: demo-oai-public-net
      public_net:
          driver: bridge
          name: demo-oai-public-net
          ipam:
              config:
                  - subnet: 192.168.70.128/26
          driver_opts:
              com.docker.network.bridge.name: "demo-oai"
...
```

**iii.** Add ip route from gNB-host to reach the OAI-5G-Core <mark>(not working yet)</mark>

## Deploying the OAI 5G Core
### 1. Running the core
**i.** In the `~/oai-cn5g-fed/docker-compose` directory, run:
```bash
$ python3 core-network.py --type start-basic
```

**ii.** Verify network function containers are running:

<mark>Insert image for terminal output for successful OAI 5G Core deployment</mark>

### 2. Undeploying the OAI 5G Core
**i.** In the `~/oai-cn5g-fed/docker-compose` directory, run:
```bash
$ python3 core-network.py --type stop-basic
```