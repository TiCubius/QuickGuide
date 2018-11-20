
# VirtualBox - Mise en place d'une carte virtuelle "Réseau Privé Hôte"

Ce guide vous permettera de mettre en place une Machine Virtuelle (**VM**) possédant 2 cartes réseaux : une carte réseau `NAT` qui vous autorisera la VM à accéder à Internet, et une carte `Réseau Privé Hôte` afin de pouvoir communiquer avec le PC Hôte ou d'autres VM possédant la même carte réseau.

- [VirtualBox - Mise en place d'une carte virtuelle "Réseau Privé Hôte"](#virtualbox---mise-en-place-dune-carte-virtuelle-%22r%C3%A9seau-priv%C3%A9-h%C3%B4te%22)
- [ETAPE 01 - Configuration de VirtualBox](#etape-01---configuration-de-virtualbox)
    - [ETAPE 01A - Création de la carte réseau](#etape-01a---cr%C3%A9ation-de-la-carte-r%C3%A9seau)
    - [ETAPE 01B - Configuration de la carte réseau](#etape-01b---configuration-de-la-carte-r%C3%A9seau)
    - [ETAPE 01C - Configuration du serveur DHCP](#etape-01c---configuration-du-serveur-dhcp)
        - [CHOIX 01 - Utiliser le serveur DHCP](#choix-01---utiliser-le-serveur-dhcp)
        - [CHOIX 02 - Ne pas utiliser le serveur DHCP](#choix-02---ne-pas-utiliser-le-serveur-dhcp)
- [ETAPE 02 - Configurer vos VMs](#etape-02---configurer-vos-vms)
    - [ETAPE 02A - Réseau NAT](#etape-02a---r%C3%A9seau-nat)
    - [ETAPE 02B - Réseau Privé Hôte](#etape-02b---r%C3%A9seau-priv%C3%A9-h%C3%B4te)
- [ETAPE 03 - Configuration des OS](#etape-03---configuration-des-os)
    - [ETAPE 03A - Configuration de DEBIAN 9.6](#etape-03a---configuration-de-debian-9.6)

# ETAPE 01 - Configuration de VirtualBox

Avant de configurer nos VMs, nous devons tout d'abord configurer VirtualBox afin de créer et paramétrer une carte réseau virtuelle. Cette configuration peut se faire via l'interface de VirtualBox.

Pour cela, rendez-vous dans `Fichier`, puis `Host Network Manager`, ou utilisez le raccourcis clavier <kbd>Ctrl</kbd> + <kbd>W</kbd>.

## ETAPE 01A - Création de la carte réseau

> NOTE: Si vous êtes sur Windows, il est possible qu'une carte réseau ait déjà été créée lors de l'installation de VirtualBox. Vous pouvez donc passer à l'étape 01B.

- Cliquez sur le bouton `Créer`.

Cette action peut prendre quelques instants et peut demander des autorisations supplémentaires. Sous Windows, il est possible que des messages d'avertissement d'installation d'un pilote s'affiche.

## ETAPE 01B - Configuration de la carte réseau

- Sélectionner la carte réseau que vous souhaitez modifier
- Cliquez sur le bouton `Properties`

Dans l'onglet `Adapter`:
- Sélectionner `Configure Adapter Manually`
- Remplissez les champs avec les valeurs suivantes

| Champ                          | Valeur            |
| ------------------------------ | ----------------- |
| Adresse IPv4                   | 192.168.64.1      |
| Masque réseau IPv4             | 255.255.255.0     |
| Adresse IPv6                   | Valeur par défaut |
| Longueur du masque réseau IPv6 | Valeur par défaut |

## ETAPE 01C - Configuration du serveur DHCP

Lorsque vous utilisez cette nouvelle carte réseau sur vos VM, vous pouvez demander à votre OS d'obtenir automatiquement une adresse IPv4 en utilisant le serveur DHCP fourni par VirtualBox. Afin de maintenir un contrôle plus important sur vos machine, il est également possible de désactiver ce service et de configurer manuellement vos OS afin d'obtenir une adresse IPv4 spécifique.

Attention toutefois, une adresse IPv4 ne doit être alloué et utilisé qu'à une seule machine afin d'éviter tout problème.

> Ce guide montre les 2 possibilités. Vous devez suivre au choix l'une des 2 étapes suivantes:

### CHOIX 01 - Utiliser le serveur DHCP

- Sélectionner la carte réseau que vous souhaitez modifier
- Cliquez sur le bouton `Properties`

Dans l'onglet `Serveur DHCP`:
- Cochez la case `Activer le serveur`
- Remplissez les champs avec les valeurs suivantes

| Champ                          | Valeur         |
| ------------------------------ | -------------- |
| Adresse du serveur             | 192.168.64.255 |
| Masque serveur                 | 255.255.255.0  |
| Limite inférieur des adresses  | 192.168.64.1   |
| Limite supérieure des adresses | 192.168.64.254 |

Vos VM obtiendront automatiquement des adresses IPv4 situées entre `192.168.64.1` et `192.168.64.254` sur la carte `Réseau Privé Hôte`.

### CHOIX 02 - Ne pas utiliser le serveur DHCP

- Sélectionner la carte réseau que vous souhaitez modifier
- Cliquez sur le bouton `Properties`

Dans l'onglet `Serveur DHCP`:
- Décochez la case `Activer le serveur`

Vos VM n'obtiendront pas d'adresse IPv4 de manière automatique sur la carte `Réseau Privé Hôte`, vous devrez les configurer manuellement. Attention cependant à ne pas attribuer une adresse IPv4 déjà utilisée par une autre VM.

# ETAPE 02 - Configurer vos VMs

Chacune de nos VM possèderont 2 cartes réseaux:
- Une carte `Réseau NAT` permettant l'accès à Internet
- Une carte `Réseau Privé Hôte` permettant la communication VM et Machine Hôte

Pour configurer cela:
- Sélectionner la VM que vous souhaitez modifier
- Cliquez sur le bouton `Configuration`
- Accédez à l'onglet `Réseau`

## ETAPE 02A - Réseau NAT

Une fois dans l'onglet `Réseau`:
- Sélectionner l'onglet `Carte 1`
- Cochez la case `Activer la carte réseau`
- Sélectionner le mode d'accès réseau `NAT`
- Cliquez sur `Avancé`
- Laissez le Type de carte par défaut
- Laissez le Mode Promiscuité par défaut
- Laisser l'adresse MAC par défaut
- Cochez la case `Câble braché`
- Supprimer toutes les Redirections de port

La carte `Réseau NAT` est maintenant configurée

## ETAPE 02B - Réseau Privé Hôte

Une fois dans l'onglet `Réseau`:
- Sélectionner l'onglet `Carte 2`
- Cochez la case `Activer la carte réseau`
- Sélectionner le mode d'accès réseau `Réseau Privé Hôte`
- Sélectionner le nom de la carte réseau créée à l'étape #01A
- Cliquez sur `Avancé`
- Laissez le Type de carte par défaut
- Laissez le Mode Promiscuité par défaut
- Laisser l'adresse MAC par défaut
- Cochez la case `Câble braché`

La carte `Réseau Privé Hôte` est maintenant configurée

#  ETAPE 03 - Configuration des OS

Maintenant que la carte réseau virtuelle et vos VMs sont correctement configurée, il est nécessaire de configurer vos OS. Cette configuration est différente en fonction de votre OS et de sa version.

Ainsi, un Debian 9.5 Server demandera une édition d'un fichier de configuration en ligne de commande, là ou un Windows 10 Professionnel nécessitera l'utilisation de l'interface graphique.

Référez-vous aux manuels correspondant à votre OS et à sa version afin de configurer correctement les nouvelles cartes réseau.

## ETAPE 03A - Configuration de DEBIAN 9.6


**ATTENTION: Il est possible que le nom de votre interface soit différente que celle indiquée dans ce guide, que l'interface soit déjà configurée ou que vous possédiez plus ou moins d'interface. Vérifiez bien que les modifications sont conforme avec ce que vous rapporte `ip -c a` !**

**Il est recommandé de copier le fichier `/etc/network/interfaces` dans un autre repertoire afin de conserver une sauvegarde fonctionelle avant toute modification!**

> NOTE: Cette méthode fonctionne probablement sur la plupart des systèmes GNU/Linux et Unix. Il est cependant préférable de vérifier la configuration par rapport aux manuels de votre OS.

- Lancez votre VM
- Connectez-vous avec l'utilisateur `root`
- Vérifier la présence de toutes vos cartes réseaux à l'aide de la commande `ip -c a`

Si votre VM ne possède 2 cartes réseaux : une `NAT` et une `Réseau Privé Hôte`, la commande devrait retourner un résultat similaire à celui-ci:

```bash
root@devbox:~# ip -c a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:86:7f:20 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe86:7f20/64 scope link
       valid_lft forever preferred_lft forever

3: enp0s8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 08:00:27:96:e6:14 brd ff:ff:ff:ff:ff:ff
```

> REMARQUE: Cette commande retourne 3 interfaces réseaux :
> - l'interface `lo` permettant de rediriger les connexions de `localhost` à `127.0.0.1`.
> - l'interface `enp0s3` qui est connectée sur l'adresse IPv4 `10.0.2.15`. Il s'agit de notre carte `Réseau NAT`.
> - l'interface `enp0s8` qui n'est pas connectée. Il s'agit de notre nouvelle carte `Réseau Privé Hôte` non configuré.

Nous allons maintenant configurer l'interface `enp0s8`:
- Ouvrez le fichier `/etc/network/interfaces`

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3

iface enp0s3 inet dhcp
```

> REMARQUE: Seule l'interface enp0s3 semble configurée : on y apprend qu'elle est `hot-plugable` : elle est lancée au démarrage, et elle est configurée en `dhcp`.

- Rajoutez la ligne suivante après `allow-hotplug enp0s3`: **`allow-hotplug enp0s8`**
- Rajoutez les lignes suivantes après `iface enp0s3 inet dhcp`:

```bash
iface enp0s8 inet static
    address 192.168.64.64
    netmask 255.255.255.0
```

> REMARQUE: L'adresse **`192.168.64.64`** sera l'adresse IPv4 de cette VM. Il est nécessaire de sélectionner une adresse IPv4 qui n'est utilisé par aucune autre VM !

Une fois les modifications effetuées, votre fichier `/etc/network/interfaces` devrait être similaire à celui-ci:

```bash
# This file describes the network interfaces available on your syst$
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
allow-hotplug enp0s8

iface enp0s3 inet dhcp
iface enp0s8 inet static
    address 192.168.64.64
    netmask 255.255.255.0
```
