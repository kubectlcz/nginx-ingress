# HAproxy jako loadbalancer

## Nastaveni load balanceru
Prihlaste se k load balanceru
```
ssh root@172.16.16.100
```
##### Instalace HAProxy
```
apt update && apt install -y haproxy
```
##### Konfigurace haproxy
Nasledujici snippet pridejde do **/etc/haproxy/haproxy.cfg** viz **[konfigurace](/haproxy/haproxy.cfg)`**
```
frontend http-frontend
    bind *:80
    mode http
    default_backend http-backend

backend http-backend
    mode http
    balance roundrobin
    server worker-node-1 172.16.16.201:80 check fall 3 rise 2
    server worker-node-2 172.16.16.202:80 check fall 3 rise 2
```
##### Restart haproxy service
```
systemctl restart haproxy
```
V tuto chvili je nainstalovany a nakonfigurovany load balancer. Jedna se o jednoduchou konfiguraci kdy je potreba mit nadefinovany __frontend__ a __backend__

## frontend
__http-frontend__ - je nazev frontendu a je jedno co tam date.

__bind__ - rikame cokoliv prijde na load balancer na port 80

__mode__ - jaky protokol se pouzije

__default_backend__ - rikame kdo bude pozadavky ktere prijde load balancovat. Nazev muzi odpovidat nazvu __backend_backend http-backend__ a __default_backend http-backend__

```
frontend http-frontend
    bind *:80
    mode http
    default_backend http-backend
```

## backend
__http-backend__ - je nazev backendu ktery musi byt stejny jaky je uveden ve __frontendu__ a to konkretne __default_backend__.

__balance__ - rika se jaky typ balanci requestu ma load balancer pouzit v tomto pripade to je typ __roundrobin__ coz je prijde jeden request tak jde na server1 prijde druhy jde na server2 a dalsi request, ktery prijde pujde na server1 a atd.

__server__ oznacuje na jaky server ma frontend balancovat. Zde muze byt kolik jen serveru chcete v mem pripade to jsou dva. __worker-node-1__ __worker-node-2__ je jen pojmenovani serveru a pak informaci o tom na jake ip adresu se ma balancovat request

```
backend http-backend
    mode http
    balance roundrobin
    server worker-node-1 172.16.16.201:80 check fall 3 rise 2
    server worker-node-2 172.16.16.202:80 check fall 3 rise 2

```

Muzete se ze load balanceru odhlasit v tuto chvili je LB nastaven.