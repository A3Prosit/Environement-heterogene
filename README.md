


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
