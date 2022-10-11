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


🦈 **PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP**

![trameICMP](pingicmp.pcapng)


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

🦈 **PCAP qui contient les trames ARP**

![trameARP](tramearp.pcapng)


# III. DHCP you too my brooo

DHCP pour Dynamic Host Configuration Protocol est notre p'tit pote qui nous file des IPs quand on arrive dans un réseau, parce que c'est chiant de le faire à la main :)
Quand on arrive dans un réseau, notre PC contacte un serveur DHCP, et récupère généralement 3 infos :


1. une IP à utiliser

2. l'adresse IP de la passerelle du réseau

3. l'adresse d'un serveur DNS joignable depuis ce réseau

L'échange DHCP  entre un client et le serveur DHCP consiste en 4 trames : DORA, que je vous laisse chercher sur le web vous-mêmes : D

🌞 **Wireshark it**

- identifiez les 4 trames DHCP lors d'un échange DHCP
  - mettez en évidence les adresses source et destination de chaque trame

*trame 1 :* \
src : 0.0.0.0 | dst : 255.255.255.255 \
*trame 2 :* \
src : 10.33.19.254 | dst : 10.33.16.34 \
*trame 3 :* \
src : 0.0.0.0 | dst : 255.255.255.255 \
*trame 4 :* \
src : 10.33.19.254 | dst : 10.33.16.34 


- identifiez dans ces 4 trames les informations **1**, **2** et **3** dont on a parlé juste au dessus

**1.** une IP à utiliser : \
10.33.16.34 

**2**. l'adresse IP de la passerelle du réseau : \
10.33.19.254

**3.** l'adresse d'un serveur DNS joignable depuis ce réseau : \
8.8.8.8


🦈 **PCAP qui contient l'échange DORA**

![echangedora](echangedora.pcapng)