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
docker run -v ~/cnode/ipc:/ipc -it --rm  busybox

pushd /ipc/txs/
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

echo $currentSlot  > currentSlot.txt 

exit
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
$CLI transaction submit \
    --tx-file tx.signed \
    --mainnet
```
