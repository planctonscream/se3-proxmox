####début de brouillon, aucune mise en page fait######

Proxmox est un système de virtualisation basé sur Debian et qui utilise KVM. Il permet à la façon de virtualbox de créer des  snapshots de machine, des sauvegardes complètes et beaucoup d'autres fonctionnalités. 
Le serveur et les vms peuvent se gérer à partir d'une interface web, et une gestion possible des comptes utilisateurs va permettre de laisser à des tiers un accès avec plus ou moins de droits sur une machine (comme le serveur bcdi par exemple).  

L'installation pourra se faire avec les paquets proxmox, mais il sera bien plus pratique d'utiliser l'iso toute faite sur le site officiel de proxmox:
https://www.proxmox.com/en/downloads


Dans l'installation présentée, il y a trois disques dur:

sda 500 Go, sdb 100 Go et sdc 500 Go

![01](images/01.png)

On choisit d'installer proxmox sur l premier disque sda.  Les autres disques serviront à stocker des sauvegardes de machines (les snapshots sont placés dans le même esapce de stockage que les machines), des fichiers iso de livecd pour les machinest,etc...

Pour un serveur comme le se3, il sera clairement conseillé de mettre plusieurs disques identiques et d'utiliser un système zfs avec du cache. Les machines virtuelles pourront elles être en autres formats (xfs,ext4,ntfs...)

On choisit la langue, ainsi que la ville et le type de clavier.
![02](images/02.png)

**Choix du mdp root**
![03](images/03.png)

On entrera une adresse mail valide pour que le serveur puisse envoyer des alertes à l'administrateur.

**choix du réseau**

![04](images/04.png)

Entrer l'ip du serveur, pour le dns, on pourra choisir l'ip du 'Amon', oun dns externe comme celui de google.
L'installation des paquets est automatique.

Le serveur redémarre , et indique comment acceder à l'interface de gestion. On peut acceder evidemment au serveur en ssh (connexion directe par le compte root possible).
![06](images/06.png)

Il faudra se connecter en root sur le serveur et configurer le proxy en ligne de commande

```
export http_proxy="http://172.20.0.1:3128"
export https_proxy="http://172.20.0.1:3128"
export ftp_proxy="http://172.20.0.1:3128"
```

Il suffit d'utiliser un navigateur et de se rendre à l'adresse indiquée. 
**ATTENTION**, il faudra peut-être désactiver le proxy du navigateur pour accéder à l'interface.

On indique le mode pam authentification, puis le login "root", et mdp.

![07](images/07.png)

Par la suite, il sera possible de créer des comptes utilisateurs proxmox pour permettre à d'autres personnes de démarrer/éteindre/gérer des vms.

Un message d'erreur indique que le serveur n'est pas enregistré, ce qui est normal. Ignorer donc cet avertissement.


On arrive sur l'interface. 
La partie 'datecenteur' peut contenir plusieurs noeuds (serveurs proxmox). Ici il n'y en a qu'un (nom netbios pve).

De base, le systeme est installé sur la partie "local (pve)", les vms et snapshots seront écrites sur cet espace dans /var/lib/vz  si on ne modifie rien. 
Le reste du disque est organisé en lvm et pourra contenir les sauvegardes et images iso de livecd.
![08](images/08.png)

Il suffit de double cliquer sur l'espace de stockage pour changer le type de données qu'on va y mettre.
![09](images/09.png)

Il est possible de récupérer les mises à jour de proxmox sans payer de cotisation en utilisant le dépot non-entreprise. La méthode est décrite ici.

https://pve.proxmox.com/wiki/Package_Repositories

**Configuration de l'envoi de mail par le serveur**
Pour que le serveur puisse envoyer les alertes par mail, on va installer le paquet en lgne de commande
```
apt-get update
apt-get install ssmtp

```
Il suffira ensuite de copier le contenu du fichier de conf /etc/ssmtp/ssmtp.conf du se3 sur celui du serveur proxmox.

**Ajout d'un disque dur interne pour stocker les vm et/ou  les sauvegardes**

On va monter le  disque sur le serveur 
![10](images/10.png)
Il faut que le disque dispose 
On repère l'UUID du disque en faisant
```
blkid
```
Ensuite, il suffira de modifier le fichier fstab pour que le disque soit automatiquement monté au démarrage.
![fstab](images/fstab.png)
On peut faire datacenter>stockage>ajouter> répertoire
![11](images/11.png)
On choisi également le type de contenu que l'on souhaite y mettre:
image iso --> livecd
conteneur et images disques --> vm et snapshots
fichier sauvegarde vzdump --> export de machine pour sauvegarde complète.

On peut choisir de placer 


Ajout de livecd iso pour booter sur clonezilla une vm (ou autre livecd)
On va sur le serveur (et non plus sur datacenter), on se place sur l'espace choisi pour mettre les iso ex (local-sav), puis résumé>upload, on ajoute le fichier iso.




