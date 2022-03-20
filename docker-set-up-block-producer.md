### Making the KES keys

Create directories
```
docker run -v ~/cnode/ipc:/ipc -it --rm  busybox
mkdir /ipc/keys
mkdir /ipc/txs
exit
```
Create CLI shortcut
```
export CLI='docker run -it --entrypoint cardano-cli -e CARDANO_NODE_SOCKET_PATH=/ipc/node.socket -v <FULLPATH>/cnode/ipc:/ipc inputoutput/cardano-node'
```

Now make the keys
```
$CLI node key-gen-KES \
    --verification-key-file /ipc/keys/kes.vkey \
    --signing-key-file /ipc/keys/kes.skey
```

Determine start KES period

```
slotsPerKESPeriod=$(cat ~/ipc/config/mainnet-shelley-genesis.json | jq -r '.slotsPerKESPeriod')
echo slotsPerKESPeriod: ${slotsPerKESPeriod}

slotNo=$($CLI query tip --mainnet | jq -r '.slot')
echo slotNo: ${slotNo}

kesPeriod=$((${slotNo} / ${slotsPerKESPeriod})) 
echo kesPeriod: ${kesPeriod}
startKesPeriod=${kesPeriod}
echo startKesPeriod: ${startKesPeriod}
```

Save the kes.vkey and startKesPeriod to USB

### Make cold keys
**MUST BE DONE ON AIR-GAPPED MACHINE**

```
mkdir $HOME/cold-keys
pushd $HOME/cold-keys
```
Copy kes.vkey from USB to $HOME/cold-keys

Generate node/cold key

```
cardano-cli node key-gen \
    --cold-verification-key-file node.vkey \
    --cold-signing-key-file $HOME/cold-keys/node.skey \
    --operational-certificate-issue-counter node.counter
```

```
cardano-cli node issue-op-cert \
    --kes-verification-key-file kes.vkey \
    --cold-signing-key-file $HOME/cold-keys/node.skey \
    --operational-certificate-issue-counter $HOME/cold-keys/node.counter \
    --kes-period <startKesPeriod> \
    --out-file node.cert
```
Copy node.cert to USB

### Make VRF key

Back to block producer machine

```
$CLI node key-gen-VRF \
    --verification-key-file /ipc/keys/vrf.vkey \
    --signing-key-file /ipc/keys/vrf.skey
```

### Restart Block Producer Node

Add the following
```
-e CARDANO_BLOCK_PRODUCER=true
-e CARDANO_SHELLEY_KES_KEY=/ipc/keys/kes.skey
-e CARDANO_SHELLEY_VRF_KEY=/ipc/keys/vrf.skey
-e CARDANO_SHELLEY_OPERATIONAL_CERTIFICATE=/ipc/keys/node.cert

```

