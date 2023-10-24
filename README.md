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

## Job 03 (penser à faire la doc)
Les différents types de serveurs web:
- Apache2
- NGINX
- IIS
...

## Job 04
Mise en place d'un serveur DNS:
domaine principal dnsproject.prepa.com

## Job 05
- Comment obtenir un nom de domaine public?

- Quelles sont les spécificités que l'on peut avoir sur certaines
extensions de nom de domaine?

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

## Pour aller plus loin...
Génération d'un certificat auto-signé

## Pour aller encore plus loin
Instalation d'un serveur DHCPd
