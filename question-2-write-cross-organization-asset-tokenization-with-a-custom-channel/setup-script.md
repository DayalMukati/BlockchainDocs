# Setup Script

```bash
#!/usr/bin/env bash
: 'This implements the hook of additional steps of "provision" action. It executes right after the base provision action. It can be used to provision additional packages, generate random data, etc'
BLACKBOX_PROVISION_WITH_OPTS() {
  echo "Installing system dependencies..."
  apt-get update
  apt-get install -y \
      curl \
      git \
      unzip \
      apt-transport-https \
      ca-certificates \
      software-properties-common \
      jq \
      make \
      gcc

  echo "Installing Docker..."
  apt-get remove -y docker docker-engine docker.io containerd runc || true
  apt-get update
  apt-get install -y docker.io
  systemctl enable --now docker
  usermod -aG docker "$USER"

  echo "Installing Docker Compose..."
  COMPOSE_VERSION="1.29.2"
  curl -L "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  chmod +x /usr/local/bin/docker-compose

  echo "Installing Golang..."
  GO_VERSION="1.20.5"
  curl -L https://golang.org/dl/go${GO_VERSION}.linux-amd64.tar.gz -o go.tar.gz
  tar -xzf go.tar.gz
  mv go /usr/local/
  cp /usr/local/go/bin/* /usr/local/bin/
  chmod +x /usr/local/bin/*
  rm go.tar.gz
  rm go${GO_VERSION}.linux-amd64.tar.gz
  echo "Golang version $(go version) installed."

  echo "Installing specific Hyperledger Fabric binaries..."

  HYPERLEDGER_VERSION="2.5.11"
  FABRIC_BINARIES_URL="https://github.com/hyperledger/fabric/releases/download/v${HYPERLEDGER_VERSION}/hyperledger-fabric-linux-amd64-${HYPERLEDGER_VERSION}.tar.gz"
  curl -L ${FABRIC_BINARIES_URL} -o fabric-binaries.tar.gz
  tar -xzf fabric-binaries.tar.gz
  cp bin/{peer,orderer,configtxgen,configtxlator,cryptogen,osnadmin,discover,ledgerutil} /usr/local/bin/
  chmod +x /usr/local/bin/{peer,orderer,configtxgen,configtxlator,cryptogen,osnadmin,discover,ledgerutil}
  rm -rf bin builders config fabric-binaries.tar.gz
  echo "Selected Hyperledger Fabric binaries installed."
  
  echo "Downloading Solution..."
  curl -sSL https://raw.githubusercontent.com/DayalMukati/hr-asset-hlf/refs/heads/main/setup.sh | bash -s
  chmod -R 777 ../challenge
  echo "Solution successfully downloaded."
  
  echo "Starting the network"
  cd ../challenge/test-network
  ./network.sh up
  sudo chmod -R 755 /home/ubuntu/challenge/test-network/organizations/
  echo "Network Successfully Started."
}

: 'Initialize the framework with the module. It launches the corresponding action'
. <(cat /blackbox/blackbox 2>/dev/null || wget -qO- --no-cache "https://raw.githubusercontent.com/ProblemSetters/devops-blackbox/2204/blackbox" || printf "echo 'error: *** blackbox is not available'; exit 1") docker-stdl setup "challenge"

exit 0
```

