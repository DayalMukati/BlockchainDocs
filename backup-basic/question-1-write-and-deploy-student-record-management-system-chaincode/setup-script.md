# Setup Script

```

#!/usr/bin/env bash

echo "🔹 Starting system setup for Hyperledger Fabric on Ubuntu..."

# Check and remove existing 'challenge' folder
if [ -d "./challenge" ]; then
    echo "🔹 Removing existing challenge folder..."
    sudo rm -rf ./challenge
fi

# Check and remove existing Docker containers
if [ "$(docker ps -aq)" ]; then
    echo "🔹 Stopping and removing existing Docker containers..."
    docker stop $(docker ps -aq)
    docker rm $(docker ps -aq)
fi

# Update system and install necessary dependencies
echo "🔹 Updating system packages and installing dependencies..."
sudo apt-get update
sudo apt-get install -y \
    curl \
    git \
    unzip \
    apt-transport-https \
    ca-certificates \
    software-properties-common \
    jq \
    make \
    gcc

# # Install Docker
echo "🔹 Installing Docker..."
sudo apt-get remove -y docker docker-engine docker.io containerd runc || true
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"

# # Install Docker Compose
echo "🔹 Installing Docker Compose..."
COMPOSE_VERSION="1.29.2"
sudo curl -L "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install Golang
echo "🔹 Installing Golang..."
GO_VERSION="1.20.5"
curl -L https://golang.org/dl/go${GO_VERSION}.linux-amd64.tar.gz -o go.tar.gz
sudo tar -xzf go.tar.gz -C /usr/local
sudo ln -s /usr/local/go/bin/* /usr/local/bin/
rm go.tar.gz
echo "✅ Golang version $(go version) installed."

# Install Hyperledger Fabric Binaries
echo "🔹 Installing Hyperledger Fabric binaries..."
HYPERLEDGER_VERSION="2.5.11"
FABRIC_BINARIES_URL="https://github.com/hyperledger/fabric/releases/download/v${HYPERLEDGER_VERSION}/hyperledger-fabric-linux-amd64-${HYPERLEDGER_VERSION}.tar.gz"
curl -L ${FABRIC_BINARIES_URL} -o fabric-binaries.tar.gz
tar -xzf fabric-binaries.tar.gz
sudo cp bin/{peer,orderer,configtxgen,configtxlator,cryptogen,osnadmin,discover,ledgerutil} /usr/local/bin/
sudo chmod +x /usr/local/bin/{peer,orderer,configtxgen,configtxlator,cryptogen,osnadmin,discover,ledgerutil}
rm -rf bin builders config fabric-binaries.tar.gz
echo "✅ Hyperledger Fabric binaries installed."

# Download and setup challenge solution
echo "🔹 Downloading solution files..."

sudo curl -sSL https://raw.githubusercontent.com/DayalMukati/student-hlf/refs/heads/main/setup.sh | bash -s
echo "✅ Solution successfully downloaded."
ls
sudo chmod -R 777 ./challenge

# Start the Hyperledger Fabric network
echo "🔹 Starting Hyperledger Fabric test network..."
cd ./challenge/test-network
./network.sh down
./network.sh up createChannel
sudo chmod -R 755 /home/ubuntu/challenge/test-network/organizations/
echo "✅ Hyperledger Fabric network successfully started."

echo "🎉 Setup complete! Your Hyperledger Fabric environment is now ready to use."
exit 0

```

