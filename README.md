# Guide to Setting Up the Hyperledger Test Network in Ubuntu 20.04

## Install the prerequisites

### Install Docker and Docker Compose

#### Install Docker
[Reference](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04)
```
sudo apt update
```
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```
```
apt-cache policy docker-ce
```
```
sudo apt install docker-ce
```

To run docker commands without using sudo
```
sudo usermod -aG docker ${USER}
```

Log out of the server and then log back in
```
exit
```
```
ssh ubuntu@<private_ip> -i hyperledger.pem
```

Test by checking the docker version  
```
docker -v
```

#### Install Docker Compose
[Reference](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-compose-on-ubuntu-20-04)
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
```
docker-compose --version
```

### Install Node v12.16.1
[Reference](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
source ~/.bashrc
nvm install v12.16.1
```
```
node -v
```

### Install Go
[Reference](https://www.digitalocean.com/community/tutorials/how-to-install-go-on-ubuntu-20-04)
```
cd ~
curl -OL https://golang.org/dl/go1.16.7.linux-amd64.tar.gz
sha256sum go1.16.7.linux-amd64.tar.gz
sudo tar -C /usr/local -xvf go1.16.7.linux-amd64.tar.gz
sudo nano ~/.profile
```

Add the following at the end of the file
```
export PATH=$PATH:/usr/local/go/bin
```

After you’ve added this information to your profile, save and close the file. If you used nano, do so by pressing CTRL+X, then Y, and then ENTER.
```
source ~/.profile
go version
```

### Install the jq command
```
sudo apt install jq
```

## Setting Up the Test Network

### Install Fabric and Fabric Samples
```
curl -sSLO https://raw.githubusercontent.com/hyperledger/fabric/main/scripts/install-fabric.sh && chmod +x install-fabric.sh
./install-fabric.sh
```

### Bring up the test network
```
cd fabric-samples/test-network
./network.sh up
```

### Enable peer CLI
```
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
```

### Set the environment variables to operate the peer CLI as Org1
```
# Environment variables for Org1
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

### Creating a channel
```
./network.sh createChannel -c mychannel
```

### Starting a chaincode on a channel
```
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript -ccl javascript
```

### Check installed chaincodes
```
peer lifecycle chaincode queryinstalled
```

### Initialize the ledger, add sample assets
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
```

### Query the ledger using CLI
```
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

### Invoke a chaincode transaction to update asset6 owner
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```

## Clearing your dev environment (in case you want to start from scratch)

### Bring the network down
```
./network.sh down
```

### Clear fabric-samples
```
cd ~
rm -rf *
```

### Prune all Docker images
```
docker system prune -a
```

### You can start restart at Step "Setting Up the Test Network"
