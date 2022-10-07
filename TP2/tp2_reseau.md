# TP2 : Ethernet, IP, et ARP

# I. Setup IP

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

  - les deux IPs choisies, en précisant le masque \
10.10.10.1 et 10.10.10.2 
- masque : \
255.255.252.0
```
-[ipv4 : 10.10.10.2/22] - 0

[CIDR]
Host address		- 10.10.10.2
Host address (decimal)	- 168430082
Host address (hex)	- A0A0A02
Network address		- 10.10.8.0
Network mask		- 255.255.252.0
Network mask (bits)	- 22
Network mask (hex)	- FFFFFC00
Broadcast address	- 10.10.11.255
Cisco wildcard		- 0.0.3.255
Addresses in network	- 1024
Network range		- 10.10.8.0 - 10.10.11.255
Usable range		- 10.10.8.1 - 10.10.11.254
```
```
 PS C:\Windows\system32> New-NetIPAddress -InterfaceIndex 22 -IPAddress 10.10.10.2 -PrefixLength 22
```

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

```
PS C:\Windows\system32> ping 10.10.10.1

Envoi d’une requête 'Ping'  10.10.10.1 avec 32 octets de données :
Réponse de 10.10.10.1 : octets=32 temps=1 ms TTL=64
Réponse de 10.10.10.1 : octets=32 temps<1ms TTL=64
Réponse de 10.10.10.1 : octets=32 temps<1ms TTL=64
Réponse de 10.10.10.1 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 10.10.10.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 1ms, Moyenne = 0ms
```

🌞 **Wireshark it**

- **déterminez, grâce à Wireshark, quel type de paquet ICMP est envoyé par `ping`**
  - pour le ping que vous envoyez \
Type: 8 (Echo (ping) request)
  - et le pong que vous recevez en retour \
Type: 0 (Echo (ping) reply)


# II. ARP my bro

🌞 **Check the ARP table**
```
PS C:\Windows\system32> arp -a

Interface : 10.10.10.2 --- 0x16
  Adresse Internet      Adresse physique      Type
  10.10.10.1            88-a4-c2-ac-a8-2b     dynamique
  10.10.11.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```
```
PS C:\Windows\system32> ipconfig
   Passerelle par défaut. . . . . . . . . : 10.33.19.254
```
```
Interface : 10.33.16.34 --- 0x6
  Adresse Internet      Adresse physique      Type
  10.33.18.221          78-4f-43-87-f5-11     dynamique
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  10.33.19.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
  ```
🌞 **Manipuler la table ARP**

- utilisez une commande pour vider votre table ARP
```
PS C:\Windows\system32> arp -d
```
- prouvez que ça fonctionne en l'affichant et en constatant les changements
```
Interface : 10.10.10.2 --- 0x16
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
```
- ré-effectuez des pings, et constatez la ré-apparition des données dans la table ARP

```
PS C:\Windows\system32> ping 10.10.10.1
```
```
Interface : 10.10.10.2 --- 0x16
  Adresse Internet      Adresse physique      Type
  10.10.10.1            88-a4-c2-ac-a8-2b     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

🌞 **Wireshark it**

  - déterminez, pour les deux trames, les adresses source et destination

  trame 1 : \
  src : 48:9e:bd:4e:44:f4 \
  dst : Broadcast (ff:ff:ff:ff:ff:ff) \

  trame 2 : \
  src : 88:a4:c2:ac:a8:2b \
  dst : 48:9e:bd:4e:44:f4 \

  - déterminez à quoi correspond chacune de ces adresses :

48:9e:bd:4e:44:f4 : pc1 qui cherche à qui appartient l'ip 10.10.10.1 \
88:a4:c2:ac:a8:2b : pc2 qui repond car son ip est 10.10.10.1


# II.5 Interlude hackerzz

**Chose promise chose due, on va voir les bases de l'usurpation d'identité en réseau : on va parler d'*ARP poisoning*.**

> On peut aussi trouver *ARP cache poisoning* ou encore *ARP spoofing*, ça désigne la même chose.

Le principe est simple : on va "empoisonner" la table ARP de quelqu'un d'autre.  
Plus concrètement, on va essayer d'introduire des fausses informations dans la table ARP de quelqu'un d'autre.

Entre introduire des fausses infos et usurper l'identité de quelqu'un il n'y a qu'un pas hihi.

---

➜ **Le principe de l'attaque**

- on admet Alice, Bob et Eve, tous dans un LAN, chacun leur PC
- leur configuration IP est ok, tout va bien dans le meilleur des mondes
- **Eve 'lé pa jonti** *(ou juste un agent de la CIA)* : elle aimerait s'immiscer dans les conversations de Alice et Bob
  - pour ce faire, Eve va empoisonner la table ARP de Bob, pour se faire passer pour Alice
  - elle va aussi empoisonner la table ARP d'Alice, pour se faire passer pour Bob
  - ainsi, tous les messages que s'envoient Alice et Bob seront en réalité envoyés à Eve

➜ **La place de ARP dans tout ça**

- ARP est un principe de question -> réponse (broadcast -> *reply*)
- IL SE TROUVE qu'on peut envoyer des *reply* à quelqu'un qui n'a rien demandé :)
- il faut donc simplement envoyer :
  - une trame ARP reply à Alice qui dit "l'IP de Bob se trouve à la MAC de Eve" (IP B -> MAC E)
  - une trame ARP reply à Bob qui dit "l'IP de Alice se trouve à la MAC de Eve" (IP A -> MAC E)
- ha ouais, et pour être sûr que ça reste en place, il faut SPAM sa mum, genre 1 reply chacun toutes les secondes ou truc du genre
  - bah ui ! Sinon on risque que la table ARP d'Alice ou Bob se vide naturellement, et que l'échange ARP normal survienne
  - aussi, c'est un truc possible, mais pas normal dans cette utilisation là, donc des fois bon, ça chie, DONC ON SPAM

![Am I ?](./pics/arp_snif.jpg)

---

➜ J'peux vous aider à le mettre en place, mais **vous le faites uniquement dans un cadre privé, chez vous, ou avec des VMs**

➜ **Je vous conseille 3 machines Linux**, Alice Bob et Eve. La commande `[arping](https://sandilands.info/sgordon/arp-spoofing-on-wired-lan)` pourra vous carry : elle permet d'envoyer manuellement des trames ARP avec le contenu de votre choix.

GLHF.

# III. DHCP you too my brooo

![YOU GET AN IP](./pics/dhcp.jpg)

*DHCP* pour *Dynamic Host Configuration Protocol* est notre p'tit pote qui nous file des IPs quand on arrive dans un réseau, parce que c'est chiant de le faire à la main :)

Quand on arrive dans un réseau, notre PC contacte un serveur DHCP, et récupère généralement 3 infos :

- **1.** une IP à utiliser
- **2.** l'adresse IP de la passerelle du réseau
- **3.** l'adresse d'un serveur DNS joignable depuis ce réseau

L'échange DHCP  entre un client et le serveur DHCP consiste en 4 trames : **DORA**, que je vous laisse chercher sur le web vous-mêmes : D

🌞 **Wireshark it**

- identifiez les 4 trames DHCP lors d'un échange DHCP
  - mettez en évidence les adresses source et destination de chaque trame
- identifiez dans ces 4 trames les informations **1**, **2** et **3** dont on a parlé juste au dessus

🦈 **PCAP qui contient l'échange DORA**

> **Soucis** : l'échange DHCP ne se produit qu'à la première connexion. **Pour forcer un échange DHCP**, ça dépend de votre OS. Sur **GNU/Linux**, avec `dhclient` ça se fait bien. Sur **Windows**, le plus simple reste de définir une IP statique pourrie sur la carte réseau, se déconnecter du réseau, remettre en DHCP, se reconnecter au réseau. Sur **MacOS**, je connais peu mais Internet dit qu'c'est po si compliqué, appelez moi si besoin.