### On BP

Be sure to have ```stake.vkey``` and ```cold.vkey``` on BP machine

```
cat > poolMetaData.json << EOF
{
"name": "<POOL NAME>",
"description": "My pool description",
"ticker": "<POOL TICKER>",
"homepage": "https:<HOMEPAGE>"
}
EOF
```

Save poolMetaData.json to GIT Pool project 
Use RAW then https://git.io/ to shorten URL.

Download
```
wget -O poolMetaData.json <your git.io link>
```

 ```
$CLI stake-pool metadata-hash --pool-metadata-file poolMetaData.json > poolMetaDataHash.txt
```
Copy poolMetaDataHash.txt to USB

```minPoolCost=$(cat params.json | jq -r .minPoolCost)
echo minPoolCost: ${minPoolCost}
```
Find deposit needed (i.e. 500 ADA)

```
stakePoolDeposit=$(cat $NODE_HOME/params.json | jq -r '.stakePoolDeposit')
echo stakePoolDeposit: $stakePoolDeposit
```

Make sure to have stakePoolDeposit + pool pledge in payment address 
 

 ```
$CLI stake-pool registration-certificate \
    --cold-verification-key-file cold.vkey \
    --vrf-verification-key-file vrf.vkey \
    --pool-pledge 200000000 > \
    --pool-cost 340000000 \
    --pool-margin 0.02 \
    --pool-reward-account-verification-key-file stake.vkey \
    --pool-owner-stake-verification-key-file stake.vkey \
    --mainnet \
    --single-host-pool-relay <RELAY NODE DNS #1> \
    --pool-relay-port <RELAY NODE PORT #1> \
    --pool-relay-ipv4 <RELAY NODE PUBLIC IP #2> \
    --pool-relay-port <RELAY NODE PORT #2> \
    --metadata-url <METADATA URL> \
    --metadata-hash <POOL METADATA HASH> \
    --out-file pool-registration.cert

 ```
 
### ON CM

```
./cardano-cli stake-address delegation-certificate \
    --stake-verification-key-file stake.vkey \
    --cold-verification-key-file $HOME/cold-keys/node.vkey \
    --out-file deleg.cert

 ```
cp deleg.cert to USB
 
### ON BP

```
#---#
docker exec -it cardano-block1 /bin/bash

cardano-cli query utxo \
    --address $(cat /ipc/txs/payment.addr) \
    --mainnet > /ipc/txs/fullUtxo.out

exit
#---#

docker cp $CID:/ipc/txs/fullUtxo.out .

currentSlot=$($CLI query tip --mainnet | jq -r '.slot')
echo Current Slot: $currentSlot

tail -n +3 fullUtxo.out | sort -k3 -nr > balance.out
cat balance.out

tx_in=""
total_balance=0
while read -r utxo; do
    in_addr=$(awk '{ print $1 }' <<< "${utxo}")
    idx=$(awk '{ print $2 }' <<< "${utxo}")
    utxo_balance=$(awk '{ print $3 }' <<< "${utxo}")
    total_balance=$((${total_balance}+${utxo_balance}))
    echo TxHash: ${in_addr}#${idx}
    echo ADA: ${utxo_balance}
    tx_in="${tx_in} --tx-in ${in_addr}#${idx}"
done < balance.out
txcnt=$(cat balance.out | wc -l)
echo Total ADA balance: ${total_balance}
echo Number of UTXOs: ${txcnt}


$CLI transaction build-raw \
    ${tx_in} \
    --tx-out $(cat payment.addr)+$(( ${total_balance} - ${stakePoolDeposit}))  \
    --invalid-hereafter $(( ${currentSlot} + 10000)) \
    --fee 0 \
    --certificate-file pool.cert \
    --certificate-file deleg.cert \
    --out-file tx.tmp

```
