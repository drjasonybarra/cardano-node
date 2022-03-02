For docker-machine on cloud

```
docker-machine create cnode --driver kamatera \
--kamatera-api-client-id {APIClientId} \
--kamatera-api-secret {APIsecretId}
```

```
eval "$(docker-machine env cnode)"
```
