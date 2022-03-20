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
