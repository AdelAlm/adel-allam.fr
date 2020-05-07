---
layout: post
title: network on linux
categories: linux network
---
<!--more-->

---
## ifconfig
---

Historiquement, c'était la commande « `ifconfig` » qui était utilisée, cette commande à des inconvénients. Elle est peu précise et l’affichage des informations peut varier selon les distributions. Il est donc difficile par exemple de récupérer les IP par des scripts shells pour pouvoir les utiliser dans un programme. Cette commande est peu à peu remplacée par la commande « `ip` » (provient de __iproute2__).

---
## historic names
---

Historiquement, on utilisait `eth0`, `eth1`, …, `ethX`, ou le X est l’ordre de détection par la machine au démarrage. Cela peut poser un problème si cet ordre est amené à changer. Pour résoudre en partie ce problème Debian associe lors de la première connexions l’adresse MAC de la carte avec son ethX. Mais ce n'était pas suffisant.

---
## predictable names
---

Nom basé sur l’emplacement physique de la carte réseau dans la machine et non plus sur l’ordre de détection. Le format `ethX` est remplacé par `enpXsY` et `wlpXsY`
* `X` le numéro du port PCI de la carte réseau
* `Y` le numéro de slot

> Ex: enp0s5
> * en -> carte type Ethernet (wl : carte wifi)
> * p0 -> port PCI 0  
> * s3 -> slot 5

```bash
$ lspci # affiche ce qui est sur le bus PCI
...
00:05.0 Ethernet controller: Red Hat, Inc Virtio network device
...
```
    00:05.0
      00 -> signifie que le port PCI est le numéro 0
      05 -> signifie que le  slot PCI est le numéro 5

---
## iproute2
---

Docummentation sur [iproute2](https://baturin.org/docs/iproute2/).

```bash
# affiche les interfaces
$ ip address show
$ ip address show eth0

# configurer une interface
$ sudo ip address add 10.10.10.11/24 dev eth0
$ sudo ip address del 10.10.10.11/24 dev eth0

# activer/désactiver une interface
$ sudo ip link set dev eth0 up
$ sudo ip link set dev eth0 down

# États d’une interface
# UP : est configurée et activée (pas forcement d’adresse, car il peut exister délai avec DHCP)
# LOWER_UP : UP + branchée physiquement pour communiquer
# state UP:
# state DOWN: désactivé

# afficher table de routage
$ ip route show

# gérer les routes
$ sudo ip route add 10.10.10.0/24 via 192.168.0.1
$ sudo ip route del 10.10.10.0/24

# activer mode routeur (routage des paquets qui ne sont pas destinés à la machine)
$ sudo sysctl net.ipv4.ip_forward=1
$ # ou
$ sudo echo 1 > /proc/sys/net/ipv4/ip_forward
$ sudo sysctl net.ipv4.conf.eth0.forwarding=0 # interface spécifique
$ sudo echo "net.ipv4.ip_forward=1" > /etc/sysctl.conf # rendre persistant
```

---
## persistent routes
---

#### Debian

```bash
# ajouter directives dans la configuration de l'interface
$ cat /etc/network/interfaces
...
#auto eth0
#iface eth0 inet static
#	address 10.10.10.1
#	netmask 255.255.255.0
	post-up ip route add 10.0.2.0/24 via 192.168.10.253
	pre-down ip route del 10.0.2.0/24
...
```

#### CentOS

```bash
# il faut créer un fichier avec les routes à persister
$ cd /etc/sysconfig/netword-scripts
$ sudo touch route-eth0
$ sudo echo "192.168.10.0/24 via 10.0.2.253" >> route-eth0
```

---
## dynamic configuration
---

Lorsqu’une carte réseau ne connaît pas ses paramètres réseau, elle interroge le réseau (DHCP) pour cela. On parle de configuration « dynamic » au sens ou l’on reçoit les paramètres dynamiquement par le réseau, cela ne signifie pas forcément que l’adresse est dynamique, configurer DHCP pour cela.

#### Debian

```bash
$ cat /etc/network/interfaces

auto eth0 # on active l'interface à chaque boot
iface eth0 inet dhcp # doit utiliser DHCP pour auto-configuration

$ sudo systemctl restart networking
```

#### CentOS

```bash
$ cat /etc/sysconfig/network-scripts/ipcfg-enp0s3

BOOTPROTO="dhcp" # on va utiliser DHCP
DEFROUTE="yes" # cette route contient la route par défaut
PEERDNS="yes" # recevoir des données DNS par le serveur DHCP
PEERROUTES="yes" # précédent mais pour des informations de routage
NAME="enp0s3" # nom de l'interface
UUID="" # ID unique de l'interface
DEVICE="enp0s3" # DOIT CORRESPONDRE AU NOM DU FIHICER ET À CELUI DE L'INTERFACE
ONBOOT="yes" # demarrage au boot de la machine

$ sudo systemctl restart network
```

---
## static configuration
---

#### Debian

```bash
$ cat /etc/network/interfaces

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
	address 10.0.2.101
	netmask 255.255.255.0
	gateway 10.0.2.1

$ sudo systemctl restart networking
```

#### CentOS

```bash
$ cat /etc/sysconfig/ipcfg-enp0s3

BOOTPROTO=static # ou no
# PEERDNS et PEERROUTEs : on supprimer car plus de DHCP
IPADDR=10.0.2.100
NETMASK=255.255.255.0
GATEWAY=10.0.2.1
HOSTNAME=vm-web-centos
NM_CONTROLLED=no # dire au networkmanager qu’on prend le contrôle
ONBOOT=yes

$ sudo systemctl restart network 
```

---
## NetworkManager
---

Permet de gérer les interfaces sans être administrateur. On peut trouver de la documentation <a href="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/networking_guide/" target="_blank">ici</a>.

```bash
# status
systemctl status NetworkManager
systemctl restart NetworkManager

# network manager command line
$ nmcli

# mode graphique
$ nmtui

# afficher l'état
$ nmcli device
```

---
## debug
---

```bash
$ ping 192.168.1.12
$ traceroute 192.168.1.12
$ mtr –n 192.168.1.12 # ping en continu et en live

# Informations sur les interfaces 
$ ethtool 
$ ethtool eth0
$ mii-tools

# Analyse de paquets
$ sudo tcpdump -i eth0

# Test de débit
$ wget -O /dev/null http://ipv4.online-dc3.testdebit.info/5G.iso

$ lsof -n -i :80
$ lsof -n -i :https
```
