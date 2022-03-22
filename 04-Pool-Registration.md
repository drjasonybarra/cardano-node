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

Save poolMetaData.json to Github; Obtain RAW link and then
use a URL shortener

Download
```
wget -O poolMetaData.json <SHORT URL>
```

 ```
$CLI stake-pool metadata-hash --pool-metadata-file poolMetaData.json > poolMetaDataHash.txt
```

Verify pool cost
```
minPoolCost=$(cat params.json | jq -r .minPoolCost)
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
    --cold-verification-key-file /ipc/keys/node.vkey \
    --vrf-verification-key-file /ipc/keys/vrf.vkey \
    --pool-pledge 100000000 \
    --pool-cost 340000000 \
    --pool-margin 0.02 \
    --pool-reward-account-verification-key-file /ipc/txs/stake.vkey \
    --pool-owner-stake-verification-key-file /ipc/txs/stake.vkey \
    --mainnet \
    --single-host-pool-relay "<RELAY NODE DNS #1>" \
    --pool-relay-port <RELAY NODE PORT #1> \
    --pool-relay-ipv4 "<RELAY NODE PUBLIC IP #2>" \
    --pool-relay-port <RELAY NODE PORT #2> \
    --metadata-url <METADATA URL> \
    --metadata-hash <METADATA HASH> \
    --out-file /ipc/txs/pool-registration.cert


 ```

```
$CLI stake-address delegation-certificate \
    --stake-verification-key-file /ipc/keys/stake.vkey \
    --cold-verification-key-file /ipc/keys/node.vkey \
    --out-file /ipc/txs/deleg.cert

 ```

 
### ON BP

```
#---#
docker exec -it cardano-block1 /bin/bash

cardano-cli query utxo \
    --address $(cat /ipc/txs/payment.addr) \
    --mainnet > /ipc/txs/fullUtxo.out

exit
#---#

cd $HOME
CID=$(docker run -d -v ~/cnode/ipc:/ipc busybox true)
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
    --tx-out $(cat ~/cnode/ipc/txs/payment.addr)+$(( ${total_balance} - ${stakePoolDeposit}))  \
    --invalid-hereafter $(( ${currentSlot} + 10000)) \
    --fee 0 \
    --certificate-file /ipc/txs/pool-registration.cert \
    --certificate-file /ipc/txs/deleg.cert \
    --out-file /ipc/txs/tx.tmp


```

Find fee
```
fee=$($CLI transaction calculate-min-fee \
    --tx-body-file /ipc/txs/tx.tmp \
    --tx-in-count ${txcnt} \
    --tx-out-count 1 \
    --mainnet \
    --witness-count 3 \
    --byron-witness-count 0 \
    --protocol-params-file /ipc/txs/params.json | awk '{ print $1 }')

echo fee: ${fee}
txOut=$((${total_balance}-${stakePoolDeposit}-${fee}))
echo txOut: ${txOut}


$CLI transaction build-raw \
    ${tx_in} \
    --tx-out $(cat ~/cnode/ipc/txs/payment.addr)+${txOut}  \
    --invalid-hereafter $(( ${currentSlot} + 10000)) \
    --fee ${fee} \
    --certificate-file /ipc/txs/pool-registration.cert \
    --certificate-file /ipc/txs/deleg.cert \
    --out-file /ipc/txs/tx.raw


docker cp $CID:/ipc/txs/tx.raw .
```
