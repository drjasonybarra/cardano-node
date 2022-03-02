Using Docker

```

docker volume create cnode-data
docker volume create cnode-ipc
docker volume create prom

docker network create monitoring

docker run -d \
--network monitoring \
-e CARDANO_NODE_SOCKET_PATH=/ipc/node.socket \
-e CARDANO_DATABASE_PATH=/data \
-e CARDANO_PORT=6001 \
-v cnode-ipc:/ipc \
-v cnode-data:/data \
-p 6001:6001 \
inputoutput/cardano-node run

```
