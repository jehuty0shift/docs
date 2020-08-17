---
title: Ajout de bloc IP
slug: ajout-de-bloc-ip
excerpt: Commander un bloc IP sur votre Private Cloud
legacy_guide_number: '7766457'
section: Fonctionnalités OVH
order: 01
---

**Dernière mise à jour le 05/08/2020**

## Objectif

Un bloc IP peut vous servir à rendre vos services accessibles sur Internet. 

**Ce guide explique comment commander, ajouter et migrer un bloc IP associé à votre Hosted Private Cloud.**

## Prérequis

* Être connecté à votre [espace client OVHcloud](https://www.ovh.com/auth/?action=gotomanager){.external}.
* Posséder une [infrastructure Hosted Private Cloud](https://www.ovhcloud.com/fr/enterprise/products/hosted-private-cloud/){.external} sur votre compte OVHcloud.

## En pratique

### Commander un bloc IP

Pour commander un bloc IP supplémentaire pour votre **Private Cloud** , dirigez-vous sur votre espace client OVHcloud. Dans la section `Serveur`, cliquez sur la rubrique `IP` dans la colonne de gauche puis cliquez sur `Commander des IP additionnelles`{.action}. Sélectionnez ensuite votre **Private Cloud** dans le menu déroulant avant de passer à l'étape suivante.


Plusieurs champs seront à remplir pour la création de votre bloc IP

- Taille du Bloc IP (de /28 à /24)

> [!primary]
>
> Pour rappel, voici un tableau récapitulant le nombres d'IPs présentes dans un bloc, et le nombres d'IP utilisables.
> 

|Taille du bloc|IP dans le bloc|IP utilisables chez OVHcloud|
|:---:|:---:|:---:|
|28|16|11|
|27|32|27|
|26|64|59|
|25|128|123|
|24|256|251|

> [!primary]
>
> N'hésitez pas à consulter notre guide sur le [plugin OVHcloud Network](https://docs.ovh.com/fr/private-cloud/plugin-ovh-network/){.external-link} afin de savoir quelles sont les IPs réservées de votre bloc ainsi que leur utilisation.
>

- Pays du bloc IP, important dans certains cas pour le référencement de vos services (un site à affluence française aura un meilleur référencement en France si l'IP est française également)
- Nom du réseau (Information visible dans le whois du bloc ip).
- Nombre de clients estimés (Combient de clients finaux seront hébérgés sur ces IPs).
- Description du réseau (Information visible dans le whois du bloc ip).
- Usage (Information sur l'utilisation (Web, SSL, Cloud...)).

> [!success]
>
> Les frais d'activation d'un bloc sont de 2€ HT/IP. Ainsi, pour un bloc en `/28` comprenant 16 IPs, vous aurez un bon de commande de 32€HT à payer avant livraison.
>  
> Les frais de renouvellement des IPs sont gratuits.
>

Après avoir confirmé la dernière étape, vous obtenez le bon de commande de votre bloc IP. Si celui-ci est conforme à votre souhait, il vous suffit de le payer avec les moyens de paiement proposés en bas de page afin que celui-ci soit livré.

### Migrer un bloc IP entre deux Hosted Private Cloud

La migration d'un bloc d'IP nécessite de déplacer manuellement les blocs via l'APIv6 OVHcloud.

Utilisez l'appel API suivant :

> [!api]
>
> @api {POST} /ip/{ip}/move
> 

Les champs doivent être complétés ainsi :

- ip : bloc IP avec le /mask
- nexthop « newPrimaryIp » (sensible a la casse)
- to : Hosted Private Cloud de destination sous la forme pcc-XXX-XXX-XXX-XXX

![champ nexthop](images/move-api.png){.thumbnail}


Le résultat sera sous cette forme :

![champ nexthop](images/api-result.png){.thumbnail}

Utilisez ensuite cet appel API pour déplacer l'IP dans le parking des IPS :

> [!api]
>
> @api {POST} /ip/{ip}/park
> 

> [!warning]
>
> Cet appel coupe le réseau sur les VMs qui utilisent les IPs en question.
>

Vous pourrez suivre le déplacement du bloc IP depuis votre [espace client OVHcloud](https://www.ovh.com/auth/?action=gotomanager){.external} dans la partie `Server`{.action} puis `Private Cloud`{.action}. Cliquez sur votre service Hosted Private Cloud puis sur l'onglet `Operations`{.action}.

La référence de l'opération est « removeIpRipeBlock ».

![operations manager](images/operations.png){.thumbnail}

## Aller plus loin

Échangez avec notre communauté d'utilisateurs sur <https://community.ovh.com>.
