####début de brouillon######


* [Présentation](#présentation)
* [Installation de base du serveur](#installation-de-base-du-serveur)
* [Interface web de gestion](#interface-web-de-gestion)

## Présentation

Proxmox est un puissant système de virtualisation basé sur Debian et qui utilise KVM. Il permet à la façon de virtualbox de créer des  snapshots de machine, des sauvegardes complètes et beaucoup d'autres fonctionnalités. 
Le serveur et les vms peuvent se gérer à partir d'une interface web, et une gestion possible des comptes utilisateurs va permettre de laisser à des tiers un accès avec plus ou moins de droits sur une machine (comme le serveur bcdi par exemple).  

L'installation pourra se faire avec les paquets proxmox, mais il sera bien plus pratique d'utiliser l'iso toute faite sur le site officiel de proxmox:
https://www.proxmox.com/en/downloads

## Installation de base du serveur

L'installation est très simple: très peu de choses sont demandées. On regrêtera cependant le peu de choix concernant le partionnement du serveur.

Dans l'installation présentée, il y a trois disques dur:

sda 500 Go, sdb 100 Go et sdc 500 Go

![01](images/01.png)

On choisit d'installer proxmox sur l premier disque sda.  Les autres disques serviront à stocker des sauvegardes de machines (les snapshots sont placés dans le même espace de stockage que les machines), des fichiers iso de livecd pour les machines,etc...

Pour un serveur comme le se3, il sera clairement conseillé de mettre plusieurs disques identiques et d'utiliser un système zfs avec du cache ( aller voir dans `options`). Les machines virtuelles pourront elles être en autres formats (xfs,ext4,ntfs...)
![01bis](images/01bis.png)

On choisit la langue, ainsi que la ville et le type de clavier.
![02](images/02.png)

**Choix du mdp root**
![03](images/03.png)

On entrera une **adresse mail valide** pour que le serveur puisse envoyer des alertes à l'administrateur.

**choix du réseau**

![04](images/04.png)

Entrer l'ip du serveur, pour le dns, on pourra choisir l'ip du `Amon`, oun dns externe comme celui de google.
L'installation des paquets est automatique.

Le serveur redémarre , et indique comment acceder à l'interface de gestion. On peut acceder evidemment au serveur en ssh (connexion directe par le compte root possible).
![06](images/06.png)

Il faudra se connecter en root sur le serveur et configurer le proxy en ligne de commande

```
export http_proxy="http://172.20.0.1:3128"
export https_proxy="http://172.20.0.1:3128"
export ftp_proxy="http://172.20.0.1:3128"
```
## Interface web de gestion

Pour gérer le serveur et les machines virtuelles,il suffit d'utiliser un navigateur et de se rendre à l'adresse indiquée. 


**ATTENTION**, il faudra peut-être désactiver le proxy du navigateur pour accéder à l'interface.

On indique le mode `pam authentification`, puis le login "root", et mdp.

![07](images/07.png)

Par la suite, il sera possible de créer des comptes utilisateurs proxmox pour permettre à d'autres personnes de démarrer/éteindre/gérer des vms.

Un message d'erreur indique que le serveur n'est pas enregistré, ce qui est normal. Ignorer donc cet avertissement.


On arrive sur l'interface. 
On observe la présence d'un datacenter, qui pourra contenir plusieurs serveurs proxmox (noeuds ou clusters).Ici il n'y en a qu'un (nom netbios pve).
Le disque de départ est donc partionné en deux parties distinctes:
*local(pve)* qui prend environ 1/5 du disque sda .
*local-lvm  (pve)*, l'espace restant 

De base, le systeme est installé sur la partie "local (pve)", les vms et snapshots seront écrites sur cet espace dans /var/lib/vz  si on ne modifie rien. 
Le reste du disque est organisé en lvm et pourra contenir les sauvegardes et images iso de livecd.
![08](images/08.png)

Il suffit de double cliquer sur l'espace de stockage pour changer le type de données qu'on va y mettre.
![09](images/09.png)
**Quelques points à modifier pour la gestion du serveur**

*modification des sources
Il est possible de récupérer les mises à jour de proxmox sans payer de cotisation en utilisant le dépot non-entreprise. La méthode est décrite ici.

https://pve.proxmox.com/wiki/Package_Repositories

*Configuration de l'envoi de mail par le serveur
Pour que le serveur puisse envoyer les alertes par mail, on va installer le paquet en lgne de commande
```
apt-get update
apt-get install ssmtp

```
On modifie le fichier de configuration
```
nano /etc/ssmtp/ssmtp.conf
```

Il suffira ensuite de copier le contenu du fichier de conf /etc/ssmtp/ssmtp.conf du se3 sur celui du serveur proxmox. 5voici à quoi doit ressembler le contenu du fichier.

```
# Genere par l'interface de Se3
root=marc.bansse@ac-versailles.fr
mailhub=smtp.nerim.net
rewriteDomain=lyc-prevert-longjumeau.ac-versailles.fr
hostname=lyc-prevert-longjumeau.ac-versailles.fr
```


**Ajout d'un disque dur interne pour stocker les vm et/ou  les sauvegardes**

Dans l'exemple suivant, on choisit de mettre ses vm et les sauvegardes dans deux disques qui seront montés dans /vm et dans /sauvegardes.

Il faut évidemment créer ces répertoire sur le serveur avec la commande *mkdir*.

Chaque disque doit posséder une partition valide et doit êre formaté. 

On repère l'UUID du disque en faisant:
```
blkid
```


Ensuite, il suffira de modifier le fichier fstab pour que le disque soit automatiquement monté au démarrage.
![10](images/10.png)

On peut faire datacenter>stockage>ajouter> répertoire. On indique le répertoire choisi comme point de montage.
![11](images/11.png)

On choisi également le type de contenu que l'on souhaite y mettre:

*image iso --> livecd ou autre image de cd*

*conteneur et images disques --> vm et snapshots*

*fichier sauvegarde vzdump --> export de machine pour sauvegarde complète.*



**Ajout de livecd iso pour booter sur clonezilla une vm (ou autre livecd)**
On va sur le serveur (et non plus sur datacenter), on se place sur l'espace choisi pour mettre les iso ex (local-sav), puis résumé>upload, on ajoute le fichier iso qui a été téléchargé sur un poste quelconque.
![13](images/13.png)

**Utilisation su serveur**
Au lieu d'utiliser un logiciel comme putty, il suffira de se connecter à l'interface et de cliquer sur shell pour travailler sur le terminal /tty1


**création de VM**
A venir

**restauration d'un serveur existant sur une vm proxmox**
a venir

**snapshots**
**savegarde complète d'une machine**
**restauration d'une machine**



