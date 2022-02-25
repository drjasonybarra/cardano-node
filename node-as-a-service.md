### Node as a service

```
cat <<EOF | sudo tee -a /etc/systemd/system/cardano-node.service
[Unit]
Description=Cardano Node Relay
After=multi-user.target
[Service]
Type=simple
ExecStart=/home/cardano/bin/cardano-node run --config /home/cardano/cnode/config/mainnet-config.json --topology /home/cardano/cnode/config/mainnet-topology.json --database-path  /home/cardano/cnode/db/ --socket-path  /home/cardano/cnode/sockets/node.socket --host-addr 0.0.0.0 --port 6001    

KillSignal = SIGINT
RestartKillSignal = SIGINT
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=cardano-node
LimitNOFILE=32768


Restart=always
RestartSec=15s
WorkingDirectory=/home/cardano/cnode
User=cardano
[Install]
WantedBy=multi-user.target
EOF
```


Then

```
sudo systemctl daemon-reload
sudo systemctl enable cardano-node.service
sudo systemctl start cardano-node.service
```
