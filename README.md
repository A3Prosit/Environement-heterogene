


# Prosit 4 - Environnement hétérogène

Team :

Animateur : Flo

Secrétaire : Fantou

Gestionnaire : Étienne

Scribe : Nico

## Mots clés :

Groupe policy object

- Open LDAP

- Architecture distribué

- OS différents

- Linux

- Réplication

- AD

- Robustesse de l’architecture

- Serveur web

- Annuaire

## Contexte :

Quoi ?

- Mettre en place un LDAP sur différents OS (Linux et Windows)

- Faire cohabiter 2 OS différents

- Augmenter la robustesse de l’architecture

Comment ?

- Avec OpenLDAP

- Avec réplication

Pourquoi ?

- Pour qu’il y ai toujours au moins un serveur opérationnel

## Contraintes :

Aucune

## Problématique :

Comment faire cohabiter 2 OS ?

Comment assurer la redondance d’un annuaire LDAP sur 2 OS différents ?

Comment faire fonctionner AD sur Linux ?

## Généralisation :

- Administration

- Réplication

- Cohabitation

## Hypothèses :

- Il existe une application permettant de faire fonctionner AD sur Linux

- Samba permet de partage de fichiers entre plusieurs machines

- Le SMB permet de transférer des fichiers entre des postes Windows

- Samba est un module Linux

- Samba fonctionne avec n’importe quel LDAP

- Samba est utilisable sur Windows et Linux

## Plan d’action :

Études

### Samba

Samba est une suite logiciel open source créée en 1992, il s'agit d'une re-implementation du protocole SMB 
Samba propose des service des fichiers et des services I/O  pour différents clients Windows et peut s'intégrer à un domaines Windows Server en tant que contrôleur de domaine ou membre du domaine. Depuis la v4 il supporte Active Directory et les domaines Windwos NT

Samba tourne sur la plupart des systèmes Unix, OpenVMS et Unix-like y compris macOS (mac OS X 10.2+) et macOS Server etest inclu dans GNU.

SMB Server Message Block est le protocole utilisé par le système de fichier de Windows



pb de sécu:

- sur certaines versions de samba 3.6.3- les utilisateurs pouvaient gagner accès au root à partir d'une co anonyme en exploitant une erreur RPC
- le 12 avril 2016 Badlock a été douvert (permet les man in the middle)
- le 24 mai 2017 SambaCry/EternalRed a été découver permettant l'exécution de code à distance

**Fonctionalités:**
Samba permet le partage de fichiers et d'I/O entre pc Windows et Unix, il s'agit d'une implémentation de nombreux services et protocoles:
- NETBIOS sur TCP/IP
- SMB (aka CIFS)
- DCE/RPC plus précisément MSRPC
- un serveur WINS aka NetBIOS Name Server (NBNS)
- la suite de protocole NT Domain
- la BDD SAM (Security Account Managment)
- Local Security Manager (LSA)
- NT-style printing service(SPOOLSS)
- NTLM
- Active Directory Logon utilisant un version modifiée de Kerberos et LDAP
- DFS Server
NBT et WINS sont dépréciés sur Windows


**Joindre un DC Samba et un DC existant**

préparation du client (host):

serv DS local : par défaut le premier DC d'une forêt fait tourner le DNS des zones AD  mais il conseillé d'avoir ne redondance avec plusieurs DC servant de DNS. Donc si on veut que le nouveau DC propose le service DNS il faut config BIND9 avant de démarrer Samba

kerberos: 
config /etc/krb5.conf
```
[libdefaults]
    dns_lookup_realm = false
    dns_lookup_kdc = true
    default_realm = SAMDOM.EXAMPLE.COM
  ```
on peut utiliser kinit pour demander un ticket pour l'administrateur de domaine:

```
kinit administrator
```

et on peut lister les tickets kerberos avec ```klist```


Pour join l'AD en tant que DC
```
samba-tool domain join samdom.example.com DC -U"SAMDOM\administrator" --dns-backend=SAMBA_INTERNAL
```
on peut ajouter --site=SITE pour directement joindre en tant que DC d'un site spécifique et --option="interface=lo eth0" --optin="bind interfaces only=yes"

LA réplication de Sysvol n'est pas supportée par Samba, il faut utiliser un outil externe, tous les DC doivent avoir the même ID mappings pour les utilisateurs et les groupes

par défaut un DC Samba stock les ID utilisateurs et les ID des groupes dans des attributs "xidNumber" dans idmap.ldb". A cause du fonctionnement d'idmap.ldb on ne peut garantire que chaque DC utilise le même ID pour un utilisateur ou un groupe donné, pour s'en assurer on doit donc :
- créer un hot-backup de /usr/local/samba/private/idmap.ldb du DC existant ```# tdbbackup -s .bak /usr/local/samba/private/idmap.ldb``` créé `/usr/local/samba/private/idmap.ldb.bak`
- bouger le backup dans /usr/local/samba/private/ du nouveau DC et enlever le .bak à la fin
- reset l'ACL du dossier Sysvol sur le nouveau DC ```samba-tool ntacl sysvolreset```
 
**réplication Sysvol**

créé /etc/xinetd.d/rsync

```
service rsync
{
   disable         = no
   only_from       = 10.99.0.0/28     # Restrict to your DC address(es) or ranges, to prevent other hosts retrieving the content, too.
   socket_type     = stream
   wait            = no
   user            = root
   server          = /usr/bin/rsync
   server_args     = --daemon
   log_on_failure += USERID
}
``` 
créé /etc/rsyncd.conf
```
[SysVol]
path = /usr/local/samba/var/locks/sysvol/
comment = Samba Sysvol Share
uid = root
gid = root
read only = yes
auth users = sysvol-replication
secrets file = /usr/local/samba/etc/rsyncd.secret
```

/usr/local/samba/etc/rsyncd.secret

```
sysvol-replication:pa$$w0rd
```

redémarrer xinted

créer /usr/local/samba/etc/rsync-sysvol.secret etmettre le mdp d'avant

puis ``` rsync --dry-run -XAavz --delete-after --password-file=/usr/local/samba/etc/rsync-sysvol.secret rsync://sysvol-replication@{IP-of-you-PDC}/SysVol/ /path/to/your/sysvol/folder/```

### Réplication sur AD




### OpenLDAP

logiciel open source qui est une implémentation de LDAP


### Maitres d’opérations sur AD

restauration AD: AD Magic Restore permet de restaurer rapidement et simplement les objets Active Directory de manière complète ou partielle. On peut le trouver sur http://admagicrestore.codeplex.com/

L'avantage par rapport à la corbeille AD : la fonctionnalité de comparaison et restauration granulaire

Il s'agit d'un script Powershell disposant d'une interface graphique

il s'apuye sur:
- DSAMAIN pour l'instanciation de ntds.dit
-   Les clichés instantanés de volume (volume shadow copy) ou sauvegarde locale réalisé avec Windows Server Backup (le remplaçant de NTBackup)
-   Module PowerShell Active Directory
-   Web Services Active Directory ou service passerelle de gestion Active Directory
-   Esentutil, diskpart…

pré requis:

l'outils doit être lancé sur un serveur Windows Server 2008 ou + avec la fonctionnalité AD-Domain-Services et le web service Active Directory sur au moins un des contrôleurs de domaine.
On peut alors instancier la BDD depuis un NTDS.DIT récupéré depuis un des contrôleurs de domaine pour récupérer les objets

Pour utiliser les snapshots et backups il faut utiliser l'outil depuis le contrôleur de domaine.

 On ne peut pas récupérer les objets purgés de la base pour changer la durée de rétention d'info on modifie l'attribut tombstonelifetime (par défaut 60 ou 180j selon l'environement)


Réalisation

- 2 corbeilles
