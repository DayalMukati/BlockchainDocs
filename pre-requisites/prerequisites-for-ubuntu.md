---
description: >-
  Before you begin, you should confirm that you have installed all the
  prerequisites below on the platform where you will be running Hyperledger
  Fabric.
cover: >-
  https://images.unsplash.com/photo-1639322537228-f710d846310a?crop=entropy&cs=tinysrgb&fm=jpg&ixid=MnwxOTcwMjR8MHwxfHNlYXJjaHwyfHxibG9ja2NoYWlufGVufDB8fHx8MTY3Njg3MDU3Mw&ixlib=rb-4.0.3&q=80
coverY: 0
---

# ⚓ Prerequisites for Ubuntu

### Install Git

Download the latest version of [git](https://git-scm.com/downloads) if it is not already installed.

```sh
sudo apt-get update
```

```shell
sudo apt-get install git
```

### Install cURL

Download the latest version of the [cURL](https://curl.haxx.se/download.html) tool if it is not already installed or if you get errors running the curl commands from the documentation.

```shell
sudo apt install curl
```

### Docker and Docker Compose

You will need the following installed on the platform on which you will be operating, or developing (or for), Hyperledger Fabric:

**Set up the repository**

1.  Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:

    ```bash
    sudo apt-get update

    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```
2.  Add Docker’s official GPG key:

    ```bash
    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```
3.  Use the following command to set up the repository:

    ```bash
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

**Install Docker Engine**

1.  Update the `apt` package index:

    ```bash
    sudo apt-get update
    ```


2.  Install Docker Engine, containerd, and Docker Compose.



    To install the latest version, run:

    ```bash
     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

    ***
3. Make sure the docker daemon is running.

```shell
sudo systemctl start docker
sudo systemctl enable docker
sudo groupadd docker
sudo usermod -aG docker ${USER}
```

```bash
sudo setfacl --modify user:$USER:rw /var/run/docker.sock
```

4. Check if your Hello-World works fine:&#x20;

```shell
docker run hello-world
```

5. Check Version:

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version

```

```
docker compose version
```

### Install GoLang

We would need Go-Lang to run chaincodes on Hyperledger Fabric. Let's Install by running the below commands.

```bash
sudo snap install golang --classic
source ~/.profile
```

Let's check the go version.

```bash
go version
```

### Install NVM

NVM is a Node Version Manager tool. Using the NVM utility, you can install multiple node.js versions on a single system. You can also choose a specific Node version for applications.

```bash
sudo apt-get install build-essential libssl-dev
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.profile
```

### Install Node and NPM

You can install multiple node.js versions using nvm. And use the required version for your application from installed node.js.

```bash
nvm install v22
nvm use v22
```

Let's check Node & Npm Versions

```bash
node --version
npm --version
```

We are done installing all the prerequisites.&#x20;
