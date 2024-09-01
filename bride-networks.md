# Redes Docker do Tipo Bridge

## Criando uma rede tipo Bridge

```bash
$ docker network create -d bridge localnet
```

Vistualizando as bridges criadas no docker host

```bash
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242190c63f0       no
br-5758387451e5         8000.024203e21a23       no
```

## Testando redes do tipo Bridge

```bash

$ docker run -d --name c1 \
  --network localnet \
  alpine sleep 1d

```
```
$ docker network inspect localnet --format '{{json .Containers}}' | jq
{
  "1ba1d38314b229fb52ed48f8c8f199b6b2260ab045c157ca43e2a6ef2bf0aff0": {
    "Name": "c1",
    "EndpointID": "fdd6e754e2462165d5f589eaf4cf2c60c00a5febbb0973da52c3f6361b13aa55",
    "MacAddress": "02:42:ac:13:00:02",
    "IPv4Address": "172.19.0.2/16",
    "IPv6Address": ""
  }
}
```
Note que agora há uma interface conectada na bridge br-575...

```bash
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.0242190c63f0       no
br-5758387451e5         8000.024203e21a23       no              vetha38fe33
```

Vamos testar a conectividade dentro da rede localnet

```bash
$ docker container run -it --name c2 --network localnet alpine sh
/ # ping c1
PING c1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.133 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.091 ms
64 bytes from 172.19.0.2: seq=2 ttl=64 time=0.104 ms
^C
--- c1 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.091/0.109/0.133 ms
```
Mapeando portas

```bash
$ docker run -d --name web \
  --network localnet \
  --publish 5005:80 \
  nginx

```

```bash
$ docker port web
80/tcp -> 0.0.0.0:5005
```
## Resolução de Nomes

Os containers se conhecem pelo nome dentro da mesma rede

