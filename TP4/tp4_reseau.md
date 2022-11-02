# TP4 : TCP, UDP et services réseau

# I. First steps

🌞 **Déterminez, pour 5 applications, si c'est du TCP ou de l'UDP**

🌞 **Demandez l'avis à votre OS**
- repérez, avec une commande adaptée (`netstat` ou `ss`), la connexion SSH depuis votre machine

### OperaGx (TCP):

![tp4_operagx.pcapng](tp4_operagx.pcapng)

```
PS C:\Users\Utilisateur> netstat -p TCP -b -n

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.16.34:61802     66.102.1.188:5228     ESTABLISHED
 [opera.exe]
```
 - IP et port du serveur auquel on se connecte \
 **ip** :  66.102.1.188 \
 **port** : 5228 
 - le port local que l'on ouvre pour se connecter \
 **port** : 61802

 ### FallGuys (UDP) :

![tp4_FallGuys.pcapng](tp4_FallGuye.pcapng)


```
PS C:\Users\Utilisateur> netstat -p UDP -b -n -a

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  UDP    10.33.16.34:55208     135.76.11.229:8077     ESTABLISHED
 [fall_guys_client.exe]
```
 - IP et port du serveur auquel on se connecte \
 **ip :** 135.76.11.229 \
 **port :** 8077 
 - le port local que l'on ouvre pour se connecter \
 **port :** 55208 
--> Dans ce cas, on joue le rôle du serveur et c'est le jeu qui se connecte en tant que client.

 ### Discord (TCP) :

![tp4_discord.pcapng](tp4_discord.pcapng)

```
PS C:\Users\Utilisateur> netstat -p TCP -b -n

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.16.34:61680     91.228.167.194:8883     ESTABLISHED
 [discord.exe]
```
 - IP et port du serveur auquel on se connecte \
 **ip :** 91.228.167.194 \
 **port :** 8883 
 - le port local que l'on ouvre pour se connecter \
 **port :** 61680

 ### Spotify (TDP) : Spotify utilise également de l'UDP et du QUIC

![tp4_spotify.pcapng](tp4_spotify.pcapng)

```
PS C:\Users\Utilisateur> netstat -p tcp -b -n

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.16.34:61891      217.128.86.180:2222     ESTABLISHED
 [Spotify.exe]
```

 - IP et port du serveur auquel on se connecte \
 **ip :**  217.128.86.180  \
 **port :** 2222 
 - le port local que l'on ouvre pour se connecter \
 **port :** 61891 

 ###  Firefox (TCP) :

![tp4_firefox.pcapng](tp4_firefox.pcapng)

```
PS C:\Users\Utilisateur> netstat -p TCP -b -n

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.33.16.34:55405     91.68.245.144:443     ESTABLISHED
 [firefox.exe]
```

 - IP et port du serveur auquel on se connecte \
 ip : 91.68.245.144 \
 port : 443
 - le port local que l'on ouvre pour se connecter \
 port : 55405 \


# II. Mise en place

## 1. SSH
🖥️ **Machine `node1.tp4.b1`**

🌞 **Examinez le trafic dans Wireshark**

![tp4_ssh.pcapng](tp4_ssh.pcapng)

🌞 **Demandez aux OS**

- repérez, avec une commande adaptée (`netstat` ou `ss`), la connexion SSH depuis votre machine
```
PS C:\Users\Utilisateur> netstat -p TCP -n -b

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.4.1.1:63264         10.4.1.11:22           ESTABLISHED
 [ssh.exe]
```
- Et repérez la connexion SSH depuis votre VM

```
[manon@node1 ~]$ ss
Netid              State               Recv-Q               Send-Q                                           Local Address:Port                              Peer Address:Port               Process
tcp                ESTAB               0                    0                                                    10.4.1.11:ssh                                   10.4.1.1:63264
```

## 2. Routage

🖥️ **Machine `router.tp4.b1`**

# III. DNS

## 1. Présentation

## 2. Setup

🖥️ **Machine `dns-server.tp4.b1`**

- `cat` des fichiers de conf

➜ **Fichier de conf principal**
```
[manon@dns-serveur ~]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };
        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification

           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
# référence vers notre fichier de zone
zone "tp4.b1" IN {
     type master;
     file "tp4.b1.db";
     allow-update { none; };
     allow-query {any; };
};
# référence vers notre fichier de zone inverse
zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp4.b1.rev";
     allow-update { none; };
     allow-query { any; };
};


zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```


➜ **Et pour les fichiers de zone**

```bash
# Fichier de zone pour nom -> IP

$ sudo cat /var/named/tp4.b1.db

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns-server IN A 10.4.1.201
node1      IN A 10.4.1.11
```

```bash
# Fichier de zone inverse pour IP -> nom

$ sudo cat /var/named/tp4.b1.rev

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

;Reverse lookup for Name Server
201 IN PTR dns-server.tp4.b1.
11 IN PTR node1.tp4.b1.
```
- `systemctl status named` qui prouve que le service tourne bien

```
[manon@dns-serveur etc]$ systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; vendor>
     Active: active (running) since Tue 2022-11-01 20:51:01 CET; 48min ago
   Main PID: 704 (named)
      Tasks: 5 (limit: 2684)
     Memory: 29.6M
        CPU: 216ms
     CGroup: /system.slice/named.service
             └─704 /usr/sbin/named -u named -c /etc/named.conf
```

- commande `ss` qui prouve que le service écoute bien sur un port

```
[manon@dns-serveur etc]$ ss -lnt
State             Recv-Q            Send-Q                       Local Address:Port                         Peer Address:Port            Process
LISTEN            0                 10                              10.4.1.201:53                                0.0.0.0:*
```

🌞 **Ouvrez le bon port dans le firewall**

- grâce à la commande `ss` vous devrez avoir repéré sur quel port tourne le service \
**port** : 53
- ouvrez ce port dans le firewall de la machine `dns-server.tp4.b1`
```
[manon@dns-serveur etc]$ sudo firewall-cmd --add-port=53/tcp --permanent
success

[manon@dns-serveur etc]$ sudo firewall-cmd --reload
success
```

## 3. Test

🌞 **Sur la machine `node1.tp4.b1`**

- configurez la machine pour qu'elle utilise votre serveur DNS quand elle a besoin de résoudre des noms
```
[manon@node1 network-scripts]$ cat /etc/resolv.conf
# Generated by NetworkManager
nameserver 10.4.1.201
```
- assurez vous que vous pouvez :
  - résoudre des noms comme `node1.tp4.b1` et `dns-server.tp4.b1`
```
[manon@node1 ~]$ dig node1.tp4.b1

; <<>> DiG 9.16.23-RH <<>> node1.tp4.b1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46124
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 63d7d4e4ca6b2eb701000000636230bc3684a7d6d00f5f99 (good)
;; QUESTION SECTION:
;node1.tp4.b1.                  IN      A

;; ANSWER SECTION:
node1.tp4.b1.           86400   IN      A       10.4.1.11

;; Query time: 1 msec
;; SERVER: 10.4.1.201#53(10.4.1.201)
;; WHEN: Wed Nov 02 09:56:28 CET 2022
;; MSG SIZE  rcvd: 85

```
```
[manon@node1 ~]$ dig dns-server.tp4.b1

; <<>> DiG 9.16.23-RH <<>> dns-server.tp4.b1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32167
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e7688d1b5de248c6010000006362311c24f7fa06250e0082 (good)
;; QUESTION SECTION:
;dns-server.tp4.b1.             IN      A

;; ANSWER SECTION:
dns-server.tp4.b1.      86400   IN      A       10.4.1.201

;; Query time: 1 msec
;; SERVER: 10.4.1.201#53(10.4.1.201)
;; WHEN: Wed Nov 02 09:58:04 CET 2022
;; MSG SIZE  rcvd: 90
```
- mais aussi des noms comme `www.google.com`
```
[manon@node1 ~]$ dig www.google.com

; <<>> DiG 9.16.23-RH <<>> www.google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49839
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 0e42732e2fe376a501000000636230945fd22ee8a5d5c896 (good)
;; QUESTION SECTION:
;www.google.com.                        IN      A

;; ANSWER SECTION:
www.google.com.         300     IN      A       142.250.201.164

;; Query time: 461 msec
;; SERVER: 10.4.1.201#53(10.4.1.201)
;; WHEN: Wed Nov 02 09:55:48 CET 2022
;; MSG SIZE  rcvd: 87
```

🌞 **Sur votre PC**

- utilisez une commande pour résoudre le nom `node1.tp4.b1` en utilisant `10.4.1.201` comme serveur DNS

On cherche l'index de la carte reseau "Wifi" :
```
PS C:\Users\Utilisateur> Get-DnsClientServerAddress
InterfaceAlias               Interface Address ServerAddresses
                             Index     Family
--------------               --------- ------- ---------------
Wi-Fi                                7 IPv4    {192.168.2.1}
Wi-Fi                                7 IPv6    {}
```
```
PS C:\Users\Utilisateur> netsh interface ipv4 add dnsserver "Wi-Fi" 10.4.1.201 index=7
```
```
PS C:\Users\Utilisateur> ipconfig /all
Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . : home
   Description. . . . . . . . . . . . . . : Intel(R) Wi-Fi 6 AX201 160MHz
   Adresse physique . . . . . . . . . . . : F2-9B-85-D2-57-A9
   DHCP activé. . . . . . . . . . . . . . : Oui
   Adresse IPv6. . . . . . . . . . . . . .: 2a01:cb19:8f4:6c00:d478:f3cd:3e0f:b798(préféré)
   Adresse IPv6 temporaire . . . . . . . .: 2a01:cb19:8f4:6c00:865:df38:6bf4:81b6(préféré)
   Adresse IPv6 de liaison locale. . . . .: fe80::d478:f3cd:3e0f:b798%7(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.2.17(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
   Bail obtenu. . . . . . . . . . . . . . : mercredi 2 novembre 2022 10:29:58
   Bail expirant. . . . . . . . . . . . . : jeudi 3 novembre 2022 10:48:51
   Passerelle par défaut. . . . . . . . . : fe80::8ef8:13ff:fe2f:234e%7
                                       192.168.2.1
   Serveur DHCP . . . . . . . . . . . . . : 192.168.2.1
   IAID DHCPv6 . . . . . . . . . . . : 133340037
   DUID de client DHCPv6. . . . . . . . : 00-03-00-01-F2-9B-85-D2-57-A9
   Serveurs DNS. . .  . . . . . . . . . . : 10.4.1.201
                                       2a01:cb19:8f4:6c00:8ef8:13ff:fe2f:234e
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé
   Liste de recherche de suffixes DNS propres à la connexion :
                                       home
```
```
PS C:\Users\Utilisateur> nslookup
Serveur par dÚfaut :   dns-server.tp4.b1
Address:  10.4.1.201

> node1.tp4.b1
Serveur :   dns-server.tp4.b1
Address:  10.4.1.201

Nom :    node1.tp4.b1
Address:  10.4.1.11
```


🦈 **Capture d'une requête DNS vers le nom `node1.tp4.b1` ainsi que la réponse**

![dns.node1.tp4.b1](dns.node1.tp4.b1.pcapng)
