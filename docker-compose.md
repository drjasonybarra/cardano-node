Docker Compose File

```
networks:
  monitoring:
    driver: bridge

services:
  cardano-node
    image: inputoutput/cardano-node
    ports:
      - 6001:6001
    volumes:
      - cnode-ipc:/ipc
      - cnode-data:/data
    environment:
      - NETWORK=mainnet
      - CARDANO_NODE_SOCKET_PATH=/ipc/node.socket
    command: 
      - '--topology /ipc/mainnet-topology.json'
      - '--config /ipc/mainnnet-config.json'
    restart: unless-stopped
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - cnode-prom:/prometheus
     command:
      - '--config.file=/prometheus/prometheus.yml'
    ports:
      - 9090:9090
    networks:
      - monitoring


  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    restart: unless-stopped
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - 3000:3000
    restart: unless-stopped
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=celery
      - GF_USERS_ALLOW_SIGN_UP=false
    depends_on:
      - prometheus
    networks:
      - monitoring

volumes:
  cnode-ipc:
    external: true
  cnode-data:
    external: true
  cnode-prom:
    external: true
```
