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
PS C:\Users\Utilisateur> cat ~/.ssh/id_rsa_reseau2.pub | ssh manon@vpn2 "sudo nano >> ~/etc/.ssh/authorized_keys"
```
➜ **Assurez-vous d'avoir une connexion sans mot de passe à la machine**

> Vraiment, faitez-le sinon vous bloquerez votre propre accès à l'étape suivante.

### B. SSH Server Hardening

Je vais pas ré-écrire la roue, y'a 10000 articles pour faire ça sur le web. Je vous link [**un fichier de conf**](https://gist.github.com/cig0/d769b26c5f8a79fbd2ff0e635ebe0846) qui contient les clauses importantes à changer dans votre fichier de conf.

> Vous pouvez google "ssh server hardening" et/ou me demander pour + de clarté.

# II. Serveur VPN

Il existe deux solutions de référence dans le monde open-source/Linux pour mettre en place un serveur VPN : OpenVPN et WireGuard. Je vous laisse faire vos propres recherches si vous voulez une idée de la diff

Là non plus je vais pas ré-écrire la roue, je vous renvoie vers [l'excellent **guide de Digital Ocean** pour mettre en place Wireguard sur une machine Rocky 8.](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-rocky-linux-8)

Si vous suivez ce guide, vous pouvez sautez la partie concernant l'IPv6, pour mieux maîtriser ce que vous faites.