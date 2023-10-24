# DDWS
Serveur réseau

## Job 01
Je ne pense pas qu'il soit encore nécessaire de préciser comment
on intalle une machine virtuelle sur à l'aide de VirtualBox, VMware ou un
autre hyperviseur.

Configuration de la machine virtuelle:
- Mémoire: 4 GB
- Processeur: 2
- Disque: 20GB

A la fin de l'installation on verifie que le serveur ssh est opérationnel:
```bash
systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Tue 2023-10-24 17:34:23 CEST; 1min 49s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 713 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 754 (sshd)
      Tasks: 1 (limit: 4582)
     Memory: 3.4M
        CPU: 50ms
     CGroup: /system.slice/ssh.service
             └─754 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Warning: some journal files were not opened due to insufficient permissions.
```

## Job 02
Installation d'un serveur Apache2

```bash
sudo apt install apache2 -y &&
systemcl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Tue 2023-10-24 17:53:15 CEST; 25s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 3335 (apache2)
      Tasks: 55 (limit: 4582)
     Memory: 10.7M
        CPU: 104ms
     CGroup: /system.slice/apache2.service
             ├─3335 /usr/sbin/apache2 -k start
             ├─3337 /usr/sbin/apache2 -k start
             └─3338 /usr/sbin/apache2 -k start
```
![ApacheOK](./pictures/ApacheOK.jpg)

## Job 03
Les différents types de serveurs web:
- Apache2: l'un des serveurs les plus populaires et des plus utilisés sur internet. Il est utilisable sur n'importe
quel système d'exploitation et dispose d'énormément de plugins. Produit depuis 1995, son code est éprouvé et les
failles de sécurités sont très rapidement corrigés grâce à une équipe de développement très active.
- NGINX: un petit nouveau par rapport aux autre (2004). C'est un reverse-proxy natif et support des charges très importantes.
Néanmoins, sans rentrer dans des détails politiques, et aux vues du contexte actuel, je me méfie d'un projet créé et
gérée par une forte communauté russe. 
- IIS: serveur web fonctionnent uniquement sous Windows Server. Il permet de gérer une application Web avec une prise
en charge avancée des langages de programmation au travers des modules CGI. Il s'administre facilement via le gestionnaire
de serveur. Date de première sortie 1995.

Il en existe encore un bon nombre écrits dans différents langages et sous différentes licences. Je pense que je
vais m'arrêter là.

## Job 04
Mise en place d'un serveur DNS:
domaine principal dnsproject.prepa.com

- Installation des paquets
```bash
sudo apt install -y bind9 bind9utils bind9-docs dnsutils
```

- Editiondu fichier /etc/bind/named.conf.options:
```bind
acl internal-network {
    192.168.145.0;
};

logging {
    channel laplateforme_log {
        file "/var/log/named/laplateforme.log" version 3 size 250k;
        severity info;
    };
    categorie default {
        laplateforme_log;
    };
};

options {
    directory "/var/cache/bind"
    additional-from-auth no;
    additional-from-cache no;
    version "Bind Server";

    forward {
        8.8.8.8;
        1.1.1.1;
    };

    listen-on port 53 {localhost; 192.168.145.0;};
    dnssec-validation auto;
    allow-recursive { 127.0.0.1; };
    auth-ns-domain no;
    listen-on-v6 { any;};
};
```

- Edition du fichier /etc/bind/named.conf.local:
```bind
zone "dnsproject.prepa.com" {
    type master;
    file "/etc/bind/db.dnsproject.prepa.com";
    notify no;
    allow-update { none; };
    allow-transfert { 192.168.145.129; };
    also-notify { 192.168.145.129; };
};
```

- Création du fichier de zone /etc/bind/db.dnsproject.prepa.com
```bind
$TTL 86400
@   IN  SOA ns.dnsproject.prepa.com. (
            202310251   ;   serial
            3600        ;   refresh
            1800        ;   retry
            604800      ;   expire
            86400       ;   minimum
)
;
            NS  ns.dnsproject.prepa.com.    ;
;
ns          A       192.168.145.129
www         CNAME   192.168.145.129
```

- On verifie enfin que le service est bien configuré:
```bash
named-checkconf /etc/bind9.conf

named-checkzone dnsproject.prepa.com /etc/bind/???
named-checkzone 
```

- On lance enfin le service:
```bash
sudo systemctl enable --now bind9 &&
systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; preset: enabled)
     Active: active (running) since Wed 2023-10-25 00:40:20 CEST; 52min ago
       Docs: man:named(8)
   Main PID: 9308 (named)
     Status: "running"
      Tasks: 6 (limit: 4582)
     Memory: 40.0M
        CPU: 131ms
     CGroup: /system.slice/named.service
             └─9308 /usr/sbin/named -f -u bind
```

## Job 05
- Comment obtenir un nom de domaine public?
On achête un nom de domaine par l'intermédiaire d'un *bureau d'enregistrement* comme par exemple et sans faire de
publicité **GANDI**. Ce bureau effectue alors la validation de propriété pour un certain temps auprès d'un *Registrar*,
une sorte de greffier des noms de domaine. Au niveau mondial, c'est l'**ICANN** (Internet Corporation for Assigned Names and Numbers)
qui alloue l'espace d'adressage des protocoles internets. Cette société de droit californien délègue son autorité
à des registrars nationaux. En France il s'agit de l' **Afnic**.

- Quelles sont les spécificités que l'on peut avoir sur certaines
extensions de nom de domaine?
Pour certains domaine, il faut pouvoir justifier de sa nationalité, ou encore de sa résidence dans une ville particulière, ou encore
d'exercer une profession particulier. Chaque extension de nom de domaine peut avoir ses propres contraintes.

## Job 06
Montrons que l'on peut se connecter sur le serveur web avec le nom de domaine

## Job 07
Mise en place d'un pare-feu ufw sur le serveur.
Bloquage de tous les ports et services entrant ou sortant hormis:
- ssh (22/tcp),
- dns(53/udp, 53/tcp),
- http (80/tcp),
- https(443/tcp)

```bash
sudo apt install ufw -y &&
systemctl status uwf
○ ufw.service - Uncomplicated firewall
     Loaded: loaded (/lib/systemd/system/ufw.service; enabled; preset: enabled)
     Active: inactive (dead)
       Docs: man:ufw(8)
```

Bien le pare-feu n'est pas activé et cela et bien normal puisque nous ne l'avons
pas encore configuré...

```bash
sudo ufw reset &&
sudo ufw default deny outgoing &&
sudo ufw default deny incoming &&
sudo ufw allow 22/tcp &&
sudo ufw allow 53/udp &&
sudo ufw allow 53/tcp &&
sudo ufw allow 80/tcp &&
sudo ufw allow 443/tcp &&
sudo systemctl enable --now ufw &&
systemctl status enable
● ufw.service - Uncomplicated firewall
     Loaded: loaded (/lib/systemd/system/ufw.service; enabled; preset: enabled)
     Active: active (exited) since Tue 2023-10-24 19:48:20 CEST; 31s ago
       Docs: man:ufw(8)
    Process: 6850 ExecStart=/lib/ufw/ufw-init start quiet (code=exited, status=0/SUCCESS)
   Main PID: 6850 (code=exited, status=0/SUCCESS)
        CPU: 10ms
```

## Job 08
Pour les besoins de l'exercice nous allons supposer que toutes les machines sont sous Linux.
Donc nous allons mettre en place un serveur NFS.

- Installation des paquets
```
sudo apt install

```

## Pour aller plus loin...
Génération d'un certificat auto-signé

## Pour aller encore plus loin
Instalation d'un serveur DHCPd
