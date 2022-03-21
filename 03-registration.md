Copy stake.cert and payment.addr to BP /ipc/txs

```
docker exec -it cardano-block1 /bin/bash

cardano-cli query protocol-parameters \
    --mainnet \
    --out-file /ipc/txs/params.json

cardano-cli query utxo \
    --address $(cat /ipc/txs/payment.addr) \
    --mainnet > /ipc/txs/fullUtxo.out

exit
```

```
CID=$(docker run -d -v ~/cnode/ipc:/ipc busybox true)
docker cp $CID:/ipc/txs/fullUtxo.out .
docker cp $CID:/ipc/txs/params.json .

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

currentSlot=$($CLI query tip --mainnet | jq -r '.slot')
echo Current Slot: $currentSlot

$CLI transaction build-raw \
    ${tx_in} \
    --tx-out $(cat ~/cnode/ipc/txs/payment.addr)+0 \
    --invalid-hereafter $(( ${currentSlot} + 10000)) \
    --fee 0 \
    --out-file /ipc/txs/tx.draft \
    --certificate-file /ipc/txs/stake.cert

fee=$($CLI transaction calculate-min-fee \
    --tx-body-file /ipc/txs/tx.draft \
    --tx-in-count ${txcnt} \
    --tx-out-count 1 \
    --mainnet \
    --witness-count 2 \
    --byron-witness-count 0 \
    --protocol-params-file /ipc/txs/params.json | awk '{ print $1 }') 

echo fee: $fee

stakeAddressDeposit=$(cat params.json | jq -r '.stakeAddressDeposit')
echo stakeAddressDeposit : $stakeAddressDeposit

txOut=$((${total_balance}-${stakeAddressDeposit}-${fee}))
echo Change Output: ${txOut}

$CLI transaction build-raw \
    ${tx_in} \
    --tx-out $(cat ~/cnode/ipc/txs/payment.addr)+${txOut} \
    --invalid-hereafter $(( ${currentSlot} + 10000)) \
    --fee ${fee} \
    --certificate-file /ipc/txs/stake.cert \
    --out-file /ipc/txs/tx.raw

docker cp $CID:/ipc/txs/tx.raw .
```



Copy tx.raw to USB drive to be signed

On **air-gapped machine**

```
cardano-cli transaction sign \
    --tx-body-file tx.raw \
    --signing-key-file payment.skey \
    --signing-key-file stake.skey \
    --mainnet \
    --out-file tx.signed
```

Copy tx.signed to USB

On BP machine

```
docker cp tx.signed $CID:/ipc/txs/

$CLI transaction submit \
    --tx-file /ipc/txs/tx.signed \
    --mainnet
```

```
$CLI query utxo \
--address $(cat ~/cnode/ipc/txs/payment.addr) \
--mainnet

```

