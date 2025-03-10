# Pre-requisites

## Pre-requisites Installation on Ubuntu:

1.  **Update Package List** Ensure your package list is up-to-date.

    ```bash
    sudo apt-get update
    ```
2.  **Install Git** Git is required for version control and to clone the necessary repositories.

    ```bash
    sudo apt-get install -y git
    ```
3.  **Install cURL:** cURL is necessary for downloading files and interacting with web services.

    ```bash
    sudo apt-get install -y curl
    ```
4.  **Install Docker** Docker is needed to run the Hyperledger Fabric network.

    ```bash
    sudo apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
    sudo apt-get update
    sudo apt-get install -y docker-ce
    ```

    After installation, verify that Docker is installed successfully:

    ```bash
    docker --version
    ```

    To manage Docker as a non-root user:

    ```bash
    sudo usermod -aG docker $USER
    ```

    Log out and log back in so that your group membership is re-evaluated.
5.  **Install Docker Compose** Docker Compose is used to define and run multi-container Docker applications.

    ```bash
    sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")')/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    ```



Verify the installation:

```bash
docker-compose --version
```

6. **Install Go (Golang)** Go is required for writing and building the Hyperledger Fabric smart contracts.

```bash
wget https://golang.org/dl/go1.20.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz
```

Add Go to your system PATH by adding the following lines to your `~/.profile`:

```bash
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
source ~/.profile
```

Verify the installation:

```bash
go version
```



## Download Hyperledger Fabric:

Let's Create a new folder on the Desktop:

```bash
mkdir hyperledger
cd hyperledger
```

Download samples, binaries and docker images:

```
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.4.9 1.5.5
```

The command above downloads and executes a bash script that will download and extract all of the platform-specific binaries you will need to set up your network and place them into the cloned repo you created above. It retrieves the following platform-specific binaries:

> * `configtxgen`,
> * `configtxlator`,
> * `cryptogen`,
> * `discover`,
> * `idemixgen`
> * `orderer`,
> * `peer`,
> * `fabric-ca-client`,
> * `fabric-ca-server`

and places them in the `bin` sub-directory of the current working directory.

### Note: Every Question will have Fabric Samples folders as support codes for candidates.&#x20;



## Git Repo for Question and Solution:

{% embed url="https://github.com/DayalMukati/lxp-transactionlogging-hlf" %}

### It should be cloned inside the hyperledger folder.&#x20;

### Please confirm if fabric-samples and lxp-transactionlogging-hlf folder are available inside the hyperledger folder.&#x20;

### Now copy all the files of the bin folder of fabric-samples to the bin folder of lxp-transactionlogging-hlf.&#x20;

### Script for Pre-req Installation:&#x20;

```sh
#!/bin/bash

# Update Package List
echo "Updating package list..."
sudo apt-get update

# Install Git
echo "Installing Git..."
sudo apt-get install -y git

# Install cURL
echo "Installing cURL..."
sudo apt-get install -y curl

# Install Docker
echo "Installing Docker..."
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce

# Start and enable Docker service
echo "Starting and enabling Docker service..."
sudo systemctl start docker
sudo systemctl enable docker

# Create Docker group and add user to the Docker group
echo "Adding $USER to the docker group..."
sudo groupadd docker
sudo usermod -aG docker $USER

# Set ACL permissions on Docker socket
echo "Installing setfacl and setting permissions on Docker socket..."
sudo apt-get install -y acl
sudo setfacl --modify user:$USER:rw /var/run/docker.sock

# Verify Docker Installation
echo "Verifying Docker installation..."
docker --version

# Check 'docker ps'
echo "Running 'docker ps' to check Docker is running correctly..."
docker ps

# Install Docker Compose
echo "Installing Docker Compose..."
sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -oP '"tag_name": "\K(.*)(?=")')/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify Docker Compose Installation
echo "Verifying Docker Compose installation..."
docker-compose --version

# Install Go (Golang)
echo "Installing Go (Golang)..."
wget https://golang.org/dl/go1.20.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz

# Add Go to PATH
echo "Adding Go to the system PATH..."
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.profile
source ~/.profile

# Verify Go Installation
echo "Verifying Go installation..."
go version

# Create a new folder on Desktop for Hyperledger
echo "Creating hyperledger folder on Desktop..."
mkdir -p ~/Desktop/hyperledger
cd ~/Desktop/hyperledger

# Download Hyperledger Fabric samples, binaries, and docker images
echo "Downloading Hyperledger Fabric samples, binaries, and docker images..."
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.4.9 1.5.5

echo "All pre-requisites and Hyperledger setup have been completed successfully."

```
