# Develop in a cloud sandbox
# IBM Blockchain Platform

Follow the instructions on [ibm-blockchain.github.io](https://ibm-blockchain.github.io) to launch a basic IBM Blockchain network on the IBM Container Service's free plan. At the end of the install, you will be able to obtain a public URL to access your instance of Composer Playground (the UI for creating and deploying Business Networks to Fabric).

There is a single script install available as well as full documentation on individual commands for custom installations.

You will need to clone this repository to get the relevant bash scripts and YAML files.

Setting up Blockchain network and installig chaincode:
1.	Set the Persistent Volume and Persistent claim:
                ./create/create_storage.sh
                
Persistent volume: shared-pvc

                kubectl create –f ./kube-configs/storage-composer-pvc.yaml
Persistent volume: composer-pvc

                kubectl create –f ./kube-configs/storage-composer-pvc.yaml
                
2.	Create blockchain deployment:
                ./create/create_blockchain.sh
This script does the following:
  a.	Generate the crypto-config files
  b.	Generate the genesis block
  
3.	Create channel:
        PEER_MSPID="Org1MSP" CHANNEL_NAME="channel1" create/create_channel.sh
        
4.	Join channel from Org1:
        CHANNEL_NAME="channel1" PEER_MSPID="Org1MSP" PEER_ADDRESS="blockchain-org1peer1:30110" MSP_CONFIGPATH="/shared/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" create/join_channel.sh
        
5.	Join channel from Org2:
        CHANNEL_NAME="channel1" PEER_MSPID="Org2MSP" PEER_ADDRESS="blockchain-org2peer1:30210" MSP_CONFIGPATH="/shared/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp" create/join_channel.sh
        
6.	Create a fabric cli pod which can be used to install / instantiate the chaincode:

        kubectl create -f ../kube-configs/chaincode_install.yaml
        
        Copy the go chaincode to the blockchain cli
        
        kubectl cp src_dir container_name:/opt/gopath/src/
        
        Example:
        kubectl cp ../../postal chaincodeinstall:/opt/gopath/src/
        
7.	Install & Instantiate chaincode:

        Get into the blockchain cli container
        
        kubectl exec –it blockchaincli bash
        
        Copy the go chaincode to the chaincode install cli
        
        Install on Org1Peer1:
        
        export CORE_PEER_ADDRESS=blockchain-org1peer1:30110
        export CORE_PEER_LOCALMSPID=Org1MSP
        export CORE_PEER_MSPCONFIGPATH=/shared/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp  
 
        peer chaincode install -n postal-scm -v 1.0 -p postal/
	      peer chaincode instantiate -o blockchain-orderer:31010 -C channel1 -n postal-scm -v 1.0 -c '{"Args":["init"]}' -P     "OR('Org1MSP.member','Org2MSP.member')"


# Privacy Notice
For serviceability needs regarding the number of network activity, IBM has added a mechanism in the ordering service to collect a "pulse" from the networks. The UUID of a network is collected periodically and sent to a monitoring service, there is no blockchain or transaction information or data gathered or accessed. The only purpose is to provide information on activity passing through the ordering service.

The UUID is generated by the network randomly when the orderer comes up and is not attached to any further network information. The UUID is re-generated and old UUID lost whenever the ordering service is restarted.
