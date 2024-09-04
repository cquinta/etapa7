# Redes Docker do Tipo Overlay 

Funcionam apenas no modo Swarm

## Criando uma rede do tipo Overlay


```bash
$ docker network create -d overlay -o encrypted uber-net

```

```bash
[node1] (local) root@192.168.0.8 /var/log
$ docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
5005fa751bee   bridge            bridge    local
c43ba8d6df37   docker_gwbridge   bridge    local
3ae43dec21f3   host              host      local
p4dodr0rryob   ingress           overlay   swarm
5758387451e5   localnet          bridge    local
ec683a473fb9   macvlan100        macvlan   local
b89ee50f8f2a   none              null      local
c2r9594qrdbu   uber-net          overlay   swarm
```

```bash
$ docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
5c69bf480f6c   bridge            bridge    local
b8e599a7804a   docker_gwbridge   bridge    local
e76c5721dae1   host              host      local
p4dodr0rryob   ingress           overlay   swarm
a6b0b9b7d7d4   none              null      local

```

O docker só estenderá a nova rede quando houver algum serviço utilizando .

## Criar um Container na rede

Por padrão só é possível criar container na rede através do swarm. Caso seja necessário que containers "standalone" utilizem a rede ela deve ser criada com a opção ```--attachable``` 

```bash
$ docker service create --name test --with-registry-auth \
   --network uber-net \
   --replicas 2 \
   ubuntu sleep infinity
```

```bash

$ docker service ps test
ID             NAME      IMAGE           NODE      DESIRED STATE   CURRENT STATE            ERROR     PORTS
wg26rapoudmn   test.1    ubuntu:latest   node1     Running         Running 47 seconds ago
lx8jrurqjopk   test.2    ubuntu:latest   node2     Running         Running 47 seconds ago


[node2] (local) root@192.168.0.7 ~
$ docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
5c69bf480f6c   bridge            bridge    local
b8e599a7804a   docker_gwbridge   bridge    local
e76c5721dae1   host              host      local
p4dodr0rryob   ingress           overlay   swarm
a6b0b9b7d7d4   none              null      local
c2r9594qrdbu   uber-net          overlay   swarm



[node1] (local) root@192.168.0.8 /var/log
$ docker network inspect uber-net
[
    
        "Containers": {
            "f42aed7dc7eb237ebe765c4473d91b51f4d2ba0853b3609fa8ccab8881637abf": {
                "Name": "test.1.wg26rapoudmnzpg1z15l33v4a",
                "EndpointID": "73556ec59b00b9c83919a5b2e306b5a9f4f0db16d334cff82dd75a20d55acf46",
                "MacAddress": "02:42:0a:00:01:47",
                "IPv4Address": "10.0.1.71/24",
                "IPv6Address": ""
            },
            
        }
        ]



[node2] (local) root@192.168.0.7 ~
$ docker network inspect uber-net
[
    
        "Containers": {
            "74870196a120f8d6c8328c9d61400ad624b69fd612ec565df68d57a30558616a": {
                "Name": "test.2.lx8jrurqjopkqyortx40frqmx",
                "EndpointID": "587009942ef89c54671fbecb451fdf0ca1a8e9427387b203c1e04ee71b8c26b2",
                "MacAddress": "02:42:0a:00:01:48",
                "IPv4Address": "10.0.1.72/24",
                "IPv6Address": ""
            },
    
]


$ docker exec -it f42aed7dc7e bash
root@f42aed7dc7eb:/# ping
bash: ping: command not found
root@f42aed7dc7eb:/# apt-get update && apt-get install iputils-ping -y

root@f42aed7dc7eb:/# ping test.2.lx8jrurqjopkqyortx40frqmx
PING test.2.lx8jrurqjopkqyortx40frqmx (10.0.1.72) 56(84) bytes of data.
64 bytes from test.2.lx8jrurqjopkqyortx40frqmx.uber-net (10.0.1.72): icmp_seq=1 ttl=64 time=0.564 ms
64 bytes from test.2.lx8jrurqjopkqyortx40frqmx.uber-net (10.0.1.72): icmp_seq=2 ttl=64 time=0.402 ms
64 bytes from test.2.lx8jrurqjopkqyortx40frqmx.uber-net (10.0.1.72): icmp_seq=3 ttl=64 time=0.371 ms


root@f42aed7dc7eb:/# traceroute 10.0.1.72
traceroute to 10.0.1.72 (10.0.1.72), 30 hops max, 60 byte packets
 1  test.2.lx8jrurqjopkqyortx40frqmx.uber-net (10.0.1.72)  1.939 ms  1.537 ms  1.424 ms
```



