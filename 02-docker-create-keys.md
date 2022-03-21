### On COLD machine ###

```
./cardano-address recovery-phrase generate --size 24 > phrase.prv
cat phrase.prv | ./cardano-address key from-recovery-phrase Shelley > root.xsk

ccencrypt phrase.prv
ccencrypt root.xsk
```



```
#Create private keys
./cardano-address key child 1852H/1815H/0H/0/0 < root.xsk > payment.xsk
./cardano-cli key convert-cardano-address-key --shelley-payment-key --signing-key-file payment.xsk --out-file payment.skey


./cardano-address key child 1852H/1815H/0H/2/0 < root.xsk > stake.xsk
./cardano-cli key convert-cardano-address-key --shelley-payment-key --signing-key-file stake.xsk --out-file stake.skey

#Public(verification) Keys
./cardano-cli key verification-key --signing-key-file payment.skey --verification-key-file payment.vkey
./cardano-cli key verification-key --signing-key-file stake.skey --verification-key-file stake.vkey
```

## Then the same

```
cardano-cli stake-address build \
    --stake-verification-key-file stake.vkey \
    --out-file stake.addr \
    --mainnet

cardano-cli address build \
    --payment-verification-key-file payment.vkey \
    --stake-verification-key-file stake.vkey \
    --out-file payment.addr \
    --mainnet
```

copy payment.addr to USB drive
