# Serveur VPN


# I. Setup machine distante

Assurez-vous d'être connecté SSH à la machine distante pour la suite.

```
PS C:\Users\Utilisateur> ssh manon@vpn2
manon@vpn2's password:
Last login: Mon Nov  7 14:31:23 2022 from 192.168.1.1
[manon@vpn2 ~]$
```
## 1. Utilisateurs

➜ **Création d'utilisateur**

```
[root@vpn2 ~]# usermod -aG wheel manon
```
utilisateur manon ayant la possibilité d'utiliser sudo pour accéder aux droits root

## 2. Serveur SSH

### A. Connexion par clé

➜ **Génération d'une paire de clé** SUR LE CLIENT

```
PS C:\Users\Utilisateur> ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\Utilisateur/.ssh/id_rsa): C:\Users\Utilisateur/.ssh/id_rsa_reseau2
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\Utilisateur/.ssh/id_rsa_reseau2.
Your public key has been saved in C:\Users\Utilisateur/.ssh/id_rsa_reseau2.pub.
The key fingerprint is:
SHA256:tLMnMjDI+WEz5V/IzvqFA2XSUe1eMS9IdRpihBjBZW4 utilisateur@PC-Manon
The key's randomart image is:
+---[RSA 4096]----+
|       .+*+++.o .|
|       .o+..o.o+ |
|      o = Eo ..+ |
| . o o * +  o o .|
|  + B o S .. . . |
|   o * = =  .    |
|    . o O o      |
|       + =       |
|      ...        |
+----[SHA256]-----+
```
```
PS C:\> ls ~/.ssh


    Répertoire : C:\Users\Utilisateur\.ssh


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        07/11/2022     13:36           3389 id_rsa_reseau2
-a----        07/11/2022     13:36            747 id_rsa_reseau2.pub

```
➜ on copie notre clé publique pour qu'elle soit présente dans le fichier ~/etc/.ssh/authorized_keys de l'utilisateur sur le serveur distant.
```
PS C:\Users\Utilisateur> ssh-copy-id manon@vpn2
PS C:\Users\Utilisateur> ssh -i C:\Users\Utilisateur\.ssh\id_rsa_reseau2 manon@vpn2
```
➜ **Assurez-vous d'avoir une connexion sans mot de passe à la machine**

```
PS C:\Users\Utilisateur> ssh manon@vpn2
Last login: Tue Nov  8 10:49:03 2022 from 192.168.1.1
[manon@vpn2 ~]$
```

### B. SSH Server Hardening


```
[manon@vpn2 ssh]$ cat sshd_config

#################
#               #
#   Hardening   #
#               #
#################

ChallengeResponseAuthentication no
### Temporarily enabled to check IPs of possibly attackers
PasswordAuthentication yes
###

IgnoreRhosts yes
MaxAuthTries 3
Ciphers aes128-ctr,aes192-ctr,aes256-ctr
#MACs hmac-sha2-512-etm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
#KexAlgorithms=curve25519-sha256@libssh.org,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group-exchange-sha256,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1
RekeyLimit 256M
#ServerKeyBits 2048 # Deprecated
ClientAliveCountMax 2
LogLevel VERBOSE
MaxAuthTries 2
MaxSessions 3
PermitRootLogin no
UseDNS no
UsePrivilegeSeparation SANDBOX

Compression delayed
X11Forwarding no
AllowTcpForwarding no
GatewayPorts no
PermitTunnel no
TCPKeepAlive yes

#RSAAuthentication no # Deprecated
PermitEmptyPasswords no
GSSAPIAuthentication no
#PasswordAuthentication no
PasswordAuthentication no
KerberosAuthentication no
HostbasedAuthentication no
ChallengeResponseAuthentication no
```

# II. Serveur VPN (Wireguard)

- on installe deux référentiels de logiciels supplémentaires à l'index de packages du serveur : **epel**, et **elrepo**
```
[manon@vpn2 ~]$ sudo dnf install elrepo-release epel-release -y
```
- on installe  Wireguard 
```
[manon@vpn2 ~]$ sudo dnf install kmod-wireguard wireguard-tools -y
```
- on génère une clé privé sur le serveur et on lui supprime toutes les autorisations pour que seul root puisse y accéder
```
[manon@vpn2 ~]$ wg genkey | sudo tee /etc/wireguard/private.key
[sudo] password for manon:
AOr2v73F9p5DcbXXgPYRVDa6B5BojyOzF/zxVchY7FM=
```
- on génère une clé publique sur le serveur correspondant à la clé privé 
```
[manon@vpn2 ~]$ sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
[sudo] password for manon:
+aWHFsKyySgp+lfMq2ofNVVvwAMXOy8vQ8YpAMuqDQw=
```
- Choix de la plage IPv4 : \
10.8.0.0/24

- Choix de la plage IPv6 :
    - on collecte un horodatage 
     ```
    [manon@vpn2 etc]$ date +%s%N
    1667905833986197739
    ```
    - on récupère notre machine-id :
    ```
    [manon@vpn2 etc]$ cat /var/lib/dbus/machine-id
    c1c6b326f7834aca950baf5211ca7609
    ```
    - on combine l'horodatage avec machine-id et on hacher la valeur à l'aide de l'algorithme SHA-1
    ```
    [manon@vpn2 etc]$ printf 1667905833986197739c1c6b326f7834aca950baf5211ca7609 | sha1sum
    1f14da58ed5e4e803da8896f6c8f46e4747ee5e3  -
    ```
    - on récupère les 5 derniers octets :
    ```
    [manon@vpn2 etc]$ printf 1f14da58ed5e4e803da8896f6c8f46e4747ee5e3 | cut -c 31-
    e4747ee5e3
    ```
    - on obtient la plage : \
    fde4 : 747e : e5e3 : : /64

- on créer un fichier de configuration :
```
[manon@vpn2 etc]$ sudo vi /etc/wireguard/wg0.conf
```
```
[manon@vpn2 etc]$ sudo cat /etc/wireguard/wg0.conf
[Interface]
PrivateKey = AOr2v73F9p5DcbXXgPYRVDa6B5BojyOzF/zxVchY7FM=
Address = 10.8.0.1/24, fde4:747e:e5e3::1/64
ListenPort = 51820
SaveConfig = true
```
- on configure le transfert IP :
```
[manon@vpn2 ~]$ cat /etc/sysctl.conf
# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/.
#
# Vendors settings live in /usr/lib/sysctl.d/.
# To override a whole file, create a new file with the same in
# /etc/sysctl.d/ and put new settings there. To override
# only specific settings, add a file with a lexically later
# name in /etc/sysctl.d/ and put new settings there.
#
# For more information, see sysctl.conf(5) and sysctl.d(5).
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```
- on charge les nouvelles valeurs pour notre session de terminal actuelle :
```
[manon@vpn2 ~]$ sudo sysctl -p
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
```
- on configure le firewall :
    - autorise l'accès  au service Wireguard sur le port UDP 51820
    ```
    [manon@vpn2 ~]$ sudo firewall-cmd --zone=public --add-port=51820/udp    --permanent
    success
    ```
    - on ajoute l'interface wg0 pour permettre au trafic sur l'interface VPN d'atteindre d'autres interfaces sur le serveur WireGuard.
    ```
    [manon@vpn2 ~]$ sudo firewall-cmd --zone=internal --add-interface=wg0 --permanent
    success
    ```
    ```
    [manon@vpn2 ~]$ sudo firewall-cmd --reload
    success
    ```

```
[manon@vpn2 ~]$ sudo firewall-cmd --zone=public --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 51820/udp
  protocols:
  forward: no
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
        rule family="ipv6" source address="fde4:747e:e5e3::/64" masquerade
        rule family="ipv4" source address="10.8.0.0/24" masquerade

```
```
[manon@vpn2 ~]$ sudo firewall-cmd --zone=internal --list-interfaces
wg0
```
- on configure Wireguard pour qu'il démarre au démarrage : 
on active le service wg-quick pour l'interface wg0 en l'ajoutant à systemctl:
```
[manon@vpn2 ~]$ sudo systemctl enable wg-quick@wg0.service
Created symlink /etc/systemd/system/multi-user.target.wants/wg-quick@wg0.service → /usr/lib/systemd/system/wg-quick@.service.
```
- on démarre le service :
```
[manon@vpn2 ~]$ sudo systemctl start wg-quick@wg0.service
```
```
[manon@vpn2 ~]$ sudo systemctl status wg-quick@wg0.service
● wg-quick@wg0.service - WireGuard via wg-quick(8) for wg0
   Loaded: loaded (/usr/lib/systemd/system/wg-quick@.service; enabled; vendor preset: disabled)
   Active: active (exited) since Tue 2022-11-08 13:38:37 CET; 2min 7s ago
     Docs: man:wg-quick(8)
           man:wg(8)
           https://www.wireguard.com/
           https://www.wireguard.com/quickstart/
           https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8
           https://git.zx2c4.com/wireguard-tools/about/src/man/wg.8
  Process: 11281 ExecStart=/usr/bin/wg-quick up wg0 (code=exited, status=0/SUCCESS)
 Main PID: 11281 (code=exited, status=0/SUCCESS)

Nov 08 13:38:37 vpn2.lab.ingesup systemd[1]: Starting WireGuard via wg-quick(8) for wg0...
Nov 08 13:38:37 vpn2.lab.ingesup wg-quick[11281]: [#] ip link add wg0 type wireguard
Nov 08 13:38:37 vpn2.lab.ingesup wg-quick[11281]: [#] wg setconf wg0 /dev/fd/63
Nov 08 13:38:37 vpn2.lab.ingesup wg-quick[11281]: [#] ip -4 address add 10.8.0.1/24 dev wg0
Nov 08 13:38:37 vpn2.lab.ingesup wg-quick[11281]: [#] ip -6 address add fde4:747e:e5e3::1/64 dev wg0
Nov 08 13:38:37 vpn2.lab.ingesup wg-quick[11281]: [#] ip link set mtu 1420 up dev wg0
Nov 08 13:38:37 vpn2.lab.ingesup systemd[1]: Started WireGuard via wg-quick(8) for wg0.
```
- on installe Wireguard sur le client 
- Wireguard génère automatiquement une paire de clés 
- on créer un fichier de configuration 
```
[Interface]
PrivateKey = QPZe0DOp/vIXHIekaJmGq05m43WpqzndXKm12k9fA0U=
Address = 10.8.0.2/24, fde4:747e:e5e3::2/64

[Peer]
PublicKey = +aWHFsKyySgp+lfMq2ofNVVvwAMXOy8vQ8YpAMuqDQw=
AllowedIPs = 10.8.0.0/24, fde4:747e:e5e3::/64
Endpoint = 192.168.1.3:51820
```
- on configure une route pour acheminer tout le trafic par le tunnel vpn :

    - on cherche la passerelle par defaut utilisé par le système :
    ```
    PS C:\Users\Utilisateur> route print -4
    IPv4 Table de routage
    Itinéraires actifs :
    Destination réseau    Masque réseau  Adr. passerelle   Adr. interface Métrique
          0.0.0.0          0.0.0.0     10.33.19.254      10.33.16.34     35
    ```
    - on modifie le fichier de configuration du client :
    ```
    [Interface]
    PrivateKey = QPZe0DOp/vIXHIekaJmGq05m43WpqzndXKm12k9fA0U=
    Address = 10.8.0.2/24, fde4:747e:e5e3::2/64
    PostUp = ip route add table 200 default via 10.8.0.1/24
    PreDown = ip route delete table 200 default via 10.8.0.1/24

    [Peer]
    PublicKey = +aWHFsKyySgp+lfMq2ofNVVvwAMXOy8vQ8YpAMuqDQw=
    AllowedIPs = 10.8.0.0/24, fde4:747e:e5e3::/64
    Endpoint = 192.168.1.3:51820

    ```
- on configure les resolvers DNS :
    
    - on modifie le fichier de configuration : 
    ```
    [Interface]
    PrivateKey = QPZe0DOp/vIXHIekaJmGq05m43WpqzndXKm12k9fA0U=
    Address = 10.8.0.2/24, fde4:747e:e5e3::2/64
    DNS = 1.1.1.1, 8.8.8.8
    PostUp = ip route add table 200 default via 10.8.0.1/24
    PreDown = ip route delete table 200 default via 10.8.0.1/24

    [Peer]
    PublicKey = +aWHFsKyySgp+lfMq2ofNVVvwAMXOy8vQ8YpAMuqDQw=
    AllowedIPs = 10.8.0.0/24, fde4:747e:e5e3::/64
    Endpoint = 192.168.1.3:51820
    ```
- on ajoute la clé publique du client sur le serveur :
    ```
    [manon@vpn2 ~]$ sudo wg set wg0 peer FfjcS+RQSK+KObq8LYJy525hiFHioAiY6cfAM9xqpEQ= allowed-ips "10.8.0.2,fde4:747e:e5e3::2"
    ```
    ```
        [manon@vpn2 ~]$ sudo wg
    [sudo] password for manon:
    interface: wg0
      public key: +aWHFsKyySgp+lfMq2ofNVVvwAMXOy8vQ8YpAMuqDQw=
      private key: (hidden)
      listening port: 51820

    peer: FfjcS+RQSK+KObq8LYJy525hiFHioAiY6cfAM9xqpEQ=
      endpoint: 192.168.1.1:59747
      allowed ips: 10.8.0.2/32, fde4:747e:e5e3::2/128
      latest handshake: 8 minutes, 51 seconds ago
      transfer: 1.27 KiB received, 124 B sent
    ```
- on démarre le tunnel 
