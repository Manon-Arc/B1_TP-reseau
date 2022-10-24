# TP3 : On va router des trucs

## I. ARP

### 1. Echange ARP
| Machine  | `10.3.1.0/24` |
|----------|---------------|
| `john`   | `10.3.1.11`   |
| `marcel` | `10.3.1.12`   |

```
[john@localhost network-scripts]$ cat ifcfg-enp0s8
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.11
NETMASK=255.255.255.0
```
```
[marcel@localhost network-scripts]$ cat ifcfg-enp0s8
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.12
NETMASK=255.255.255.0
```

🌞**Générer des requêtes ARP**

- effectuer un `ping` d'une machine à l'autre
```
[manon@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.555 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.646 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.749 ms
64 bytes from 10.3.1.12: icmp_seq=4 ttl=64 time=0.736 ms
^C
--- 10.3.1.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3107ms
rtt min/avg/max/mdev = 0.555/0.671/0.749/0.078 ms
```
```
[manon@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.473 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.665 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.744 ms
^C
--- 10.3.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2048ms
rtt min/avg/max/mdev = 0.473/0.627/0.744/0.113 ms
[manon@localhost ~]$
```
- observer les tables ARP des deux machines  

```
[john@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 DELAY
10.3.1.12 dev enp0s8 lladdr 08:00:27:b1:56:39 STALE
```
```
[marcel@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 DELAY
10.3.1.11 dev enp0s8 lladdr 08:00:27:06:d1:52 STALE
```

- repérer l'adresse MAC de `john` dans la table ARP de `marcel` et vice-versa
```
[john@localhost ~]$ ip neigh show 10.3.1.12
10.3.1.12 dev enp0s8 lladdr 08:00:27:b1:56:39 STALE
```
```
[marcel@localhost ~]$ ip neigh show 10.3.1.11
10.3.1.11 dev enp0s8 lladdr 08:00:27:06:d1:52 STALE
```
- prouvez que l'info est correcte (que l'adresse MAC que vous voyez dans la table est bien celle de la machine correspondante)
  - une commande pour voir la MAC de `marcel` dans la table ARP de `john`
 ```
[john@localhost ~]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 REACHABLE
10.3.1.12 dev enp0s8 lladdr 08:00:27:b1:56:39 STALE
```
  - et une commande pour afficher la MAC de `marcel`, depuis `marcel`
```
[marcel@localhost ~]$ ip a
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b1:56:39 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb1:5639/64 scope link
       valid_lft forever preferred_lft forever
```

### 2. Analyse de trames

🌞**Analyse de trames**

🦈 **Capture réseau `tp3_arp.pcapng`** qui contient un ARP request et un ARP reply

![tp3_arp.pca](tp3_arp.pcapng)

## II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |

```
[john@localhost network-scripts]$ cat ifcfg-enp0s8
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.11
NETMASK=255.255.255.0
```
```
[marcel@localhost network-scripts]$ cat ifcfg-enp0s8
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.2.12
NETMASK=255.255.255.0
```
```
[routeur@localhost network-scripts]$ cat ifcfg-enp0s8
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.254
NETMASK=255.255.255.0
```
```
[routeur@localhost network-scripts]$ cat ifcfg-enp0s9
DEVICE=enp0s9

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.2.254
NETMASK2=255.255.255.0
```
### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

```
[routeur@localhost ~]$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s8 enp0s9

[routeur@localhost ~]$ sudo firewall-cmd --add-masquerade --zone=public
success

[routeur@localhost ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- il faut ajouter une seule route des deux côtés
```
[john@localhost network-scripts]$ cat route-enp0s8
sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s8
```
```
[marcel@localhost network-scripts]$ cat route-enp0s8
sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s8
```
- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre
```
[marcel@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=0.901 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=0.910 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=1.29 ms
^C
--- 10.3.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.901/1.034/1.292/0.182 ms
```
```
[john@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.449 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.897 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.21 ms
c64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.21 ms
^C
--- 10.3.2.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3035ms
rtt min/avg/max/mdev = 0.449/0.941/1.213/0.311 ms
```

![THE SIZE](./pics/thesize.png)

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

- videz les tables ARP des trois noeuds

```
[routeur@localhost network-scripts]$ sudo ip neigh flush all
[routeur@localhost network-scripts]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 REACHABLE
```
```
[john@localhost network-scripts]$ sudo ip neigh flush all
[john@localhost network-scripts]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 REACHABLE
```
```
[marcel@localhost network-scripts]$ sudo ip neigh flush all
[marcel@localhost network-scripts]$ ip neigh show
10.3.2.1 dev enp0s8 lladdr 0a:00:27:00:00:08 REACHABLE
```
- effectuez un `ping` de `john` vers `marcel`
  - **le `tcpdump` doit être lancé sur la machine `john`**

```
[john@localhost ~]$ sudo tcpdump -i enp0s8 -c 10 -w tp3_routage_john.pcap not port 22
[sudo] password for john:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```
```
[john@localhost network-scripts]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=2.21 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.98 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.78 ms
^C
--- 10.3.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.783/1.992/2.213/0.175 ms
[john@localhost network-scripts]$`
```
- regardez les tables ARP des trois noeuds
```
[routeur@localhost network-scripts]$ ip neigh show
10.3.1.11 dev enp0s8 lladdr 08:00:27:06:d1:52 STALE
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 DELAY
10.3.2.12 dev enp0s9 lladdr 08:00:27:b1:56:39 STALE
```
```
[john@localhost network-scripts]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 DELAY
10.3.1.254 dev enp0s8 lladdr 08:00:27:c7:87:80 STALE
```
```
[marcel@localhost network-scripts]$ ip neigh show
10.3.2.1 dev enp0s8 lladdr 0a:00:27:00:00:08 DELAY
10.3.2.254 dev enp0s8 lladdr 08:00:27:95:bb:1b STALE
```

- essayez de déduire un les échanges ARP qui ont eu lieu
  - en regardant la capture et/ou les tables ARP de tout le monde

![tp3_routage_john.pcapng](tp3_routage_john.pcapng)

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `john` `08:00:27:06:d1:52` | x              | `Broadcast` `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         |`routeur (enp0s8)` `08:00:27:c7:87:80` | x              |`john` `08:00:27:06:d1:52`  |
| 3     | ICMP        | `john` `10.3.2.11`| `john` `08:00:27:06:d1:52` | `marcel` `10.3.2.12`| `routeur (enp0s8)` `08:00:27:c7:87:80`                            |
| 4     | ICMP        | `marcel` `10.3.2.12` | `routeur (enp0s8)` `08:00:27:c7:87:80`|`john` `10.3.2.11`| `john` `08:00:27:06:d1:52`|
| 5     | ICMP        | `john` `10.3.2.11`| `john` `08:00:27:06:d1:52` | `marcel` `10.3.2.12`| `routeur (enp0s8)` `08:00:27:c7:87:80`                            |
| 6     | ICMP        | `marcel` `10.3.2.12` | `routeur (enp0s8)` `08:00:27:c7:87:80`|`john` `10.3.2.11`| `john` `08:00:27:06:d1:52`|

- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`

```
[routeur@localhost network-scripts]$ sudo ip neigh flush all
[routeur@localhost network-scripts]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 REACHABLE
```
```
[john@localhost network-scripts]$ sudo ip neigh flush all
[john@localhost network-scripts]$ ip neigh show
10.3.1.1 dev enp0s8 lladdr 0a:00:27:00:00:03 REACHABLE
```
```
[marcel@localhost network-scripts]$ sudo ip neigh flush all
[marcel@localhost network-scripts]$ ip neigh show
10.3.2.1 dev enp0s8 lladdr 0a:00:27:00:00:08 REACHABLE
```
```
[marcel@localhost ~]$ sudo tcpdump -i enp0s8 -c 10 -w tp3_routage_marcel.pcap not port 22
[sudo] password for marcel:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```

écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange

![tp3_routage_marcel.pcapng](tp3_routage_marcel.pcapng)

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `routeur (enp0s9)` `08:00:27:95:bb:1b`| x|  `Broadcast` `FF:FF:FF:FF:FF`|
| 2     | Réponse ARP | x         | `marcel` `08:00:27:b1:56:39` |x          | `routeur (enp0s9)` `08:00:27:95:bb:1b`
| 3     | ICMP        |`routeur (enp0s9)` `10.3.2.254` | `routeur (enp0s9)` `08:00:27:95:bb:1b`| `marcel` `10.3.2.12`  | `marcel` `08:00:27:b1:56:39`                          |
| 4     | ICMP        |`marcel` `10.3.2.12`| `marcel` `08:00:27:b1:56:39`  | `routeur (enp0s9)` `10.3.2.254`           | `routeur (enp0s9)` `08:00:27:95:bb:1b` |
| 5     | ICMP        |`routeur (enp0s9)` `10.3.2.254` | `routeur (enp0s9)` `08:00:27:95:bb:1b`| `marcel` `10.3.2.12`  | `marcel` `08:00:27:b1:56:39`|
| 6     | ICMP        |`marcel` `10.3.2.12`| `marcel` `08:00:27:b1:56:39`  | `routeur (enp0s9)` `10.3.2.254`           | `routeur (enp0s9)` `08:00:27:95:bb:1b|

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

- ajoutez une carte NAT en 3ème inteface sur le `router` pour qu'il ait un accès internet
```
[routeur@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:c7:87:80 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.254/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fec7:8780/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:95:bb:1b brd ff:ff:ff:ff:ff:ff
    inet 10.3.2.254/24 brd 10.3.2.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe95:bb1b/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:17:fc:06 brd ff:ff:ff:ff:ff:ff
    inet 10.0.5.15/24 brd 10.0.5.255 scope global dynamic noprefixroute enp0s10
       valid_lft 85954sec preferred_lft 85954sec
    inet6 fe80::1c25:6979:933d:60a1/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
- ajoutez une route par défaut à `john` et `marcel`
```
[john@localhost network-scripts]$ cat route-enp0s8
10.3.2.0/24 via 10.3.1.254 dev enp0s8
10.0.5.15/24 via 10.3.1.254 dev enp0s8
```
```
[marcel@localhost network-scripts]$ cat route-enp0s8
10.3.1.0/24 via 10.3.2.254 dev enp0s8
10.0.5.15/24 via 10.3.2.254 dev enp0s8
```
  - vérifiez que vous avez accès internet avec un `ping`

```
[marcel@localhost network-scripts]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=13.1 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=12.3 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 12.275/12.664/13.053/0.389 ms
```
```
[john@localhost network-scripts]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=12.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=12.6 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 12.638/12.734/12.830/0.096 ms
```

- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser
```
[john@localhost network-scripts]$ cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```

```
[marcel@localhost etc]$ cat resolv.conf
# Generated by NetworkManager
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
```
[john@localhost network-scripts]$ dig gitlab.com

; <<>> DiG 9.16.23-RH <<>> gitlab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24870
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;gitlab.com.                    IN      A

;; ANSWER SECTION:
gitlab.com.             182     IN      A       172.65.251.78

;; Query time: 28 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Oct 24 09:38:52 CEST 2022
;; MSG SIZE  rcvd: 55
```
```
[marcel@localhost ~]$ dig gitlab.com

; <<>> DiG 9.16.23-RH <<>> gitlab.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 13164
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;gitlab.com.                    IN      A

;; ANSWER SECTION:
gitlab.com.             300     IN      A       172.65.251.78

;; Query time: 36 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Oct 24 09:37:17 CEST 2022
;; MSG SIZE  rcvd: 55
```
  - puis avec un `ping` vers un nom de domaine
```
[john@localhost network-scripts]$ ping ynov.com
PING ynov.com (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=1 ttl=53 time=26.7 ms
^C64 bytes from 172.67.74.226: icmp_seq=2 ttl=53 time=27.7 ms

--- ynov.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 26.655/27.188/27.722/0.533 ms
```
```
[marcel@localhost ~]$ ping ynov.com
PING ynov.com (104.26.11.233) 56(84) bytes of data.
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=1 ttl=53 time=27.8 ms
64 bytes from 104.26.11.233 (104.26.11.233): icmp_seq=2 ttl=53 time=27.4 ms
^C
--- ynov.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 27.448/27.622/27.796/0.174 ms
```
🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
```
[john@localhost network-scripts]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=25.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=26.6 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 25.508/26.059/26.610/0.551 ms
```
- capturez le ping depuis `john` avec `tcpdump`
```
[john@localhost ~]$ sudo tcpdump -i enp0s8 -c 10 -w tp3_routage_internet.pcap not port 22
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), snapshot length 262144 bytes
```
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

![tp3_routage_internet.pcapng](tp3_routage_internet.pcapng)

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |     |
|-------|------------|--------------------|-------------------------|----------------|-----------------|-----|
| 1     | ping       | `john` `10.3.1.11` | `john` ` 08:00:27:06:d1:52`| `8.8.8.8`   | `routeur (enp0s8)` `08:00:27:c7:87:80`|     |
| 2     | pong       | `8.8.8.8`          | `routeur (enp0s8)` `08:00:27:c7:87:80`   | `john` `10.3.1.11` | `john` ` 08:00:27:06:d1:52` | ... |


## III. DHCP

On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests.

| Machine  | `10.3.1.0/24`              | `10.3.2.0/24` |
|----------|----------------------------|---------------|
| `router` | `10.3.1.254`               | `10.3.2.254`  |
| `john`   | `10.3.1.11`                | no            |
| `bob`    | oui mais pas d'IP statique | no            |
| `marcel` | no                         | `10.3.2.12`   |

```schema
   john               router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └─┬─┘    └─────┘    └───┘    └─────┘
   dhcp        │
  ┌─────┐      │
  │     │      │
  │     ├──────┘
  └─────┘
```

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `john`
```
[john@localhost network-scripts]$ sudo dnf install dhcp-server -y
```
```
[john@localhost ~]$ sudo cat /etc/dhcp/dhcpd.conf
default-lease-time 900;
max-lease-time 10800;

authoritative;

subnet 10.3.1.0 netmask 255.255.255.0 {
range 10.3.1.2 10.3.1.253;
}
```
```
[john@localhost ~]$ systemctl status dhcpd.service
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2022-10-24 11:33:30 CEST; 9min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 10932 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 2684)
     Memory: 4.6M
        CPU: 8ms
     CGroup: /system.slice/dhcpd.service
             └─10932 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Oct 24 11:33:58 localhost.localdomain dhcpd[10932]: DHCPDISCOVER from 08:00:27:cb:15:23 via enp0s8
Oct 24 11:33:58 localhost.localdomain dhcpd[10932]: ICMP Echo reply while lease 10.3.1.1 valid.
Oct 24 11:33:58 localhost.localdomain dhcpd[10932]: Abandoning IP address 10.3.1.1: pinged before offer
Oct 24 11:35:02 localhost.localdomain dhcpd[10932]: DHCPDISCOVER from 08:00:27:cb:15:23 via enp0s8
Oct 24 11:35:03 localhost.localdomain dhcpd[10932]: DHCPOFFER on 10.3.1.2 to 08:00:27:cb:15:23 via enp0s8
Oct 24 11:35:03 localhost.localdomain dhcpd[10932]: DHCPREQUEST for 10.3.1.2 (10.3.1.11) from 08:00:27:cb:15:23 via enp0s8
Oct 24 11:35:03 localhost.localdomain dhcpd[10932]: DHCPACK on 10.3.1.2 to 08:00:27:cb:15:23 via enp0s8
Oct 24 11:35:34 localhost.localdomain dhcpd[10932]: reuse_lease: lease age 31 (secs) under 25% threshold, reply with unaltered, existing lease for 10.3.1.2
Oct 24 11:35:34 localhost.localdomain dhcpd[10932]: DHCPREQUEST for 10.3.1.2 from 08:00:27:cb:15:23 via enp0s8
Oct 24 11:35:34 localhost.localdomain dhcpd[10932]: DHCPACK on 10.3.1.2 to 08:00:27:cb:15:23 via enp0s8
```
- créer une machine `bob`
```
[bob@localhost ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
DEVICE=enp0s8

BOOTPROTO=dhcp
ONBOOT=yes
```
```
[bob@localhost ~]$ sudo nmcli con up "System enp0s8"
[sudo] password for bob:
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/2)
```
- faites lui récupérer une IP en DHCP à l'aide de votre serveur
```
[bob@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:cb:15:23 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 831sec preferred_lft 831sec
    inet6 fe80::a00:27ff:fecb:1523/64 scope link
       valid_lft forever preferred_lft forever
```

🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
  - une route par défaut
  - un serveur DNS à utiliser
```
[john@localhost ~]$ sudo cat /etc/dhcp/dhcpd.conf
default-lease-time 900;
max-lease-time 10800;

authoritative;

subnet 10.3.1.0 netmask 255.255.255.0 {
range 10.3.1.2 10.3.1.253;
option routers 10.3.1.254;
option subnet-mask 255.255.255.0;
option domain-name-servers 1.1.1.1;
}
```
- récupérez de nouveau une IP en DHCP sur `bob`
```
[bob@localhost ~]$ sudo nmcli con down "System enp0s8"
```
```
[bob@localhost ~]$ sudo nmcli con up "System enp0s8"
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
```

pour tester :
  - `bob` doit avoir une IP
    - vérifier avec une commande qu'il a récupéré son IP
```
[bob@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:cb:15:23 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 732sec preferred_lft 732sec
    inet6 fe80::a00:27ff:fecb:1523/64 scope link
       valid_lft forever preferred_lft forever
```
  - vérifier qu'il peut `ping` sa passerelle
```
  [bob@localhost ~]$ ping 10.3.1.254
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.354 ms
64 bytes from 10.3.1.254: icmp_seq=2 ttl=64 time=0.617 ms
^C
--- 10.3.1.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1009ms
rtt min/avg/max/mdev = 0.354/0.485/0.617/0.131 ms
```
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande
```
[bob@localhost ~]$ ip route show
default via 10.3.1.254 dev enp0s8 proto dhcp src 10.3.1.2 metric 100
10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.2 metric 100
```
- vérifier que la route fonctionne avec un `ping` vers une IP
```
[bob@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.405 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.664 ms
^C
--- 10.3.1.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1015ms
rtt min/avg/max/mdev = 0.405/0.534/0.664/0.129 ms
```
- il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne
```
[bob@localhost ~]$ dig youtube.com

; <<>> DiG 9.16.23-RH <<>> youtube.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11637
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;youtube.com.                   IN      A

;; ANSWER SECTION:
youtube.com.            78      IN      A       142.250.179.110

;; Query time: 26 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mon Oct 24 12:44:49 CEST 2022
;; MSG SIZE  rcvd: 56
```
  - vérifier un `ping` vers un nom de domaine
```
[bob@localhost ~]$ ping ynov.com
PING ynov.com (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=1 ttl=53 time=24.9 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=2 ttl=53 time=31.8 ms
^C
--- ynov.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 24.943/28.377/31.812/3.434 ms
```

### 2. Analyse de trames

🌞**Analyse de trames**

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP
- demander une nouvelle IP afin de générer un échange DHCP
- exportez le fichier `.pcapng`

🦈 **Capture réseau `tp3_dhcp.pcapng`**
