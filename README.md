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

- Samba

- Réplication sur AD

- OpenLDAP

- Maitres d’opérations sur AD

Réalisation

- 2 corbeilles



## Samba :
- Logiciel gratuit open Source - 1992
	- Membre de la Software Freedom Conservancy : Organisation caritative, qui aide à promouvoir, développer et défendre les projets open source).
- Tourne sur l'OS Unix (Linux,MAC OS X...)
- Utilise des protocoles SBM/CIFS (SMB de SaMBa) de Microsoft Windows
	- Fournit fichiers & services impressions dans un réseau informatique
	- Facilite l’interopérabilité entre système hétérogènes
	- Peut se substituer d'être un serveur windows (contrôleur de domaine)
		- V3 ⇒ S'intégrer à un domaine de l'AD en tant que contrôleur de domaine principal
- Livré dans presque toutes les distributions de Linux


**Principe de réplication sous SAMBA** : 
- Ne fournit pas la réplication du SysVol
	- rsync est la solution pour garder une synchronisation
		- Outil unidirectionnel ⇒ Un seul DC envois le Sysvol vers les autres DC (overwriting when sync)
- Samba gère les GPO / ACL

**Différence entre hôte et NETBIOS** (Samba peut faire les deux)
- Nom d'hôte : Nom symbolique d'un ordi pour le protocole IP
	- télécharger : *ftp*
- Nom NetBios : nom symbolique d'un ordi pour le protocole NetBIOS (Réseau Microsoft)
	- Ajouter une ressource réseau (windows) : *net use*

## Active Directory - Compléments

**Rappel des 5 rôles FSMO :**
- Maître de schéma (Master) : Contrôle les modifications apportées au schéma de données de l'AD
- Maître attribution de noms de domaines (Master) : Add/Delete noms de domaines pour unicité
- Emulateur de PCD : Changements de pawd, horloge...
- Maître RID : Fournit les tranches d'identifiants uniques aux autres contrôleurs de domaine
- Maître d'infrastructure : Synchronise les changements inter-domaines

--------------------
- **Nouveau rôle** : RODC (Read Only Domain Controller) depuis Windows 2008
	- Possibilité de contrôleur de domaine en lecture seule (BDD non modifiable)
	- Réplication unidirectionnelle depuis un DC normal
	- Résous les pb de sécurité où l'intégrité physique ne peut être assurée
	- Composantes :
		- BDD
		- Réplication impossible depuis ce contrôleur
		- Mise en cache des identifiants (Pas de pwd répliqués en local)
		- Un compte unique qui sera Admin du RODC
		- DNS en lecture seule 


----------

**Rappels :**
- Catalogue global = Annuaire étendue de l'AD :
	- Un réplica partiel de tous les attributs de la forêt
	- Les infos des objets de la forêt*
	- Se situe sur le premier contrôleur de domaine
	- Recommandé de faire deux contrôleur de domaine en tant que catalogue global
	
**Rappel n°2**

![](https://www.it-connect.fr/wp-content-itc/uploads/2015/06/cours-active-directory-9.png)

- Arbre = Domaine (+ sous-domaines)
- Arbres reliés entre eux = Forêt ⇒ Partagent catalogue global
- Arbre fonctionne de manière indépendante

## Workshop 


- Paquets à utiliser :
	- Python (pour samba-tool)
	- Perl
	- acl
	- xattr
	- python-crypto (pour samba-tool)

- dans /home
	- git https://github.com/samba-team/samba samba

- /!\ Samba AD DC and --enable-selftest requires lmdb 0.9.16 or later
	- ./configure --without-ldb-lmdb
