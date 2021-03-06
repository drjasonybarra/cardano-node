Using Docker

```

docker volume create cnode-data
docker volume create cnode-ipc
docker volume create cnode-monitor

docker network create monitoring

docker run -d \
--cpus=2.5 \
--name cardano-node2 \
--network monitoring \
--restart=unless-stopped \
-e CARDANO_NODE_SOCKET_PATH=/ipc/node.socket \
-e CARDANO_DATABASE_PATH=/data \
-e CARDANO_PORT=6001 \
-v cnode-ipc:/ipc \
-v cnode-data:/data \
-p 6001:6001 \
inputoutput/cardano-node run --socket-path /ipc/node.socket

```
Other computer

Create directories ~/cnode/data and ~/cnode/ipc

```
docker volume create cnode-monitor
docker network create monitoring

docker run -d \
--cpus=2.5 \
--name cardano-block \
--network monitoring \
--restart=unless-stopped \
-e CARDANO_NODE_SOCKET_PATH=/ipc/node.socket \
-e CARDANO_DATABASE_PATH=/data \
-e CARDANO_TOPOLOGY=/ipc/config/mainnet-topology.json \
-e CARDANO_PORT=6002 \
-v ~/cnode/data:/data \
-v ~/cnode/ipc:/ipc \
-p 6002:6002 \
inputoutput/cardano-node run --socket-path /ipc/node.socket
```



To get into a running container
```
docker exec -it cardano-node /bin/bash
cardano-cli query tip --mainnet
```
To see how much disk space Docker is taking up
```
docker system df
docker system df -v
```
To see stats
```
docker stats --no-stream
```

To see last 10 lines of the log
```
docker logs <container> --tail 10
```

To see first 10 lines
```
docker logs <container> |& head -n10
```

Copy Files from Docker volume to host

```
CID=$(docker run -d -v <docker_volume>:/mnt_point busybox true)
docker cp $CID:/mnt_point ./
```
