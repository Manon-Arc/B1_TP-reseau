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
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
  - puis avec un `ping` vers un nom de domaine

🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
- capturez le ping depuis `john` avec `tcpdump`
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |     |
|-------|------------|--------------------|-------------------------|----------------|-----------------|-----|
| 1     | ping       | `marcel` `10.3.1.12` | `marcel` `AA:BB:CC:DD:EE` | `8.8.8.8`      | ?               |     |
| 2     | pong       | ...                | ...                     | ...            | ...             | ... |

🦈 **Capture réseau `tp3_routage_internet.pcapng`**

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
- créer une machine `bob`
- faites lui récupérer une IP en DHCP à l'aide de votre serveur

> Il est possible d'utilise la commande `dhclient` pour forcer à la main, depuis la ligne de commande, la demande d'une IP en DHCP, ou renouveler complètement l'échange DHCP (voir `dhclient -h` puis call me et/ou Google si besoin d'aide).

🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
  - une route par défaut
  - un serveur DNS à utiliser
- récupérez de nouveau une IP en DHCP sur `bob` pour tester :
  - `bob` doit avoir une IP
    - vérifier avec une commande qu'il a récupéré son IP
    - vérifier qu'il peut `ping` sa passerelle
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande
    - vérifier que la route fonctionne avec un `ping` vers une IP
  - il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne
    - vérifier un `ping` vers un nom de domaine

### 2. Analyse de trames

🌞**Analyse de trames**

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP
- demander une nouvelle IP afin de générer un échange DHCP
- exportez le fichier `.pcapng`

🦈 **Capture réseau `tp3_dhcp.pcapng`**