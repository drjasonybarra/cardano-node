Using Docker

```

docker volume create cnode-data
docker volume create cnode-ipc
docker volume create cnode-monitor

docker network create monitoring

docker run -d \
--name cardano-node \
--network monitoring \
-e CARDANO_NODE_SOCKET_PATH=/ipc/node.socket \
-e CARDANO_DATABASE_PATH=/data \
-e CARDANO_PORT=6001 \
-v cnode-ipc:/ipc \
-v cnode-data:/data \
-p 6001:6001 \
inputoutput/cardano-node run --socket-path /ipc/node.socket



docker run -d \
--cpus=2.5 \
--name cardano-node2 \
--network monitoring \
--restart=unless-stopped \
-e CARDANO_NODE_SOCKET_PATH=/ipc/node.socket \
-e CARDANO_DATABASE_PATH=/data \
-e CARDANO_TOPOLOGY=/ipc/config/mainnet-topology.json \
-e CARDANO_PORT=6001 \
-v cnode-ipc:/ipc \
-v cnode-data:/data \
-p 6001:6001 \
inputoutput/cardano-node run --socket-path /ipc/node.socket

```
