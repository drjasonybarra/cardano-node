Be sure to have ```stake.vkey``` and ```cold.vkey``` on BP machine


Save poolMetaData.json to GIT Pool project 
Use RAW then https://git.io/ to shorten URL
Download
wget -O poolMetaData.json <your git.io link>

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
 
On BP

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
    --pool-relay-ipv4 <RELAY NODE PUBLIC IP> \
    --pool-relay-port <RELAY NODE PORT> \
    --metadata-url <METADATA URL> \
    --metadata-hash <POOL METADATA HASH> \
    --out-file pool-registration.cert

 ```
 
