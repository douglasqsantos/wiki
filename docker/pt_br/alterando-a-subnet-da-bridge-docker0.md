# Alterando a SubNet da Bridge docker0

Aqui vamos alterar a subnet padrão da docker0, no meu caso a subnet era a 172.17.0.0/16 porém conflitava com algumas configurações da minha rede então vamos verificar como alterar está subnet.

Vamos fazer uma copia do arquivo de configuração do serviço do Docker

```bash
cp -Rfa /lib/systemd/system/docker.service /etc/systemd/system/docker.service
```

Agora precisamos alterar a nossa subnet com a opção --bip para obter as opções diponíves de rede podemos consultar: [[https://docs.docker.com/v1.8/articles/networking/|Network configuration Summary]]

```bash
vim /etc/systemd/system/docker.service 
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
ExecStart=/usr/bin/docker daemon -H fd:// --bip=192.168.200.1/24
MountFlags=slave
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
```

Agora precisamos recarregar as configurações do systemD

```bash
systemctl daemon-reload
```

Agora podemos reiniciar o Docker

```bash
systemctl restart docker
```

Agora podemos consultar a nossa bridge docker0

```bash
ip a show docker0
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:34:f9:79:92 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.1/24 scope global docker0
       valid_lft forever preferred_lft forever
```

Vamos testar rodar um container

```bash
docker run -it ubuntu /bin/bash
```

Vamos consultar as configurações de rede do container

```bash
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
4: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:c0:a8:c8:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.2/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:c0ff:fea8:c802/64 scope link 
       valid_lft forever preferred_lft forever
```

Vamos testar pingar no Google

```bash
ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=52 time=13.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=52 time=14.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=52 time=13.7 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 13.627/13.847/14.154/0.243 ms
```

Tudo funcionando corretamente.

## Referências

- https://docs.docker.com/v1.8/articles/networking/
- https://support.zenoss.com/hc/en-us/articles/203582809-How-to-Change-the-Default-Docker-Subnet
