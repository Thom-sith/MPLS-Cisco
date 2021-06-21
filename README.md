# Projet édutiant : INTERCONNEXION ET ARCHITECTURE MPLS

## Contenu du dépôt

Ce repos concentre la totalité des configurations des équipements qui m'ont permis de réaliser mon projet étudiant : Interconnexion et architecture MPLS.
Le but de ce projet étudiant a été de découvrir les bases de la technologie MPLS, s'interroger sur les problématiques de l'interconnexion, et finalement mettre en place une infrastructure en appliquant ces apprentissages.

## Schéma de l'infrastructure

L’infrastructure a été entièrement réalisée sur une machine virtuelle [Eve-ng](https://www.eve-ng.net/) mise à disposition par mon établissement. En voici une capture d'écran : 

**Objectif du projet :** 

- Réaliser l'interconnexion de sites clients (routeurs CE) au travers d'un réseau opérateur via MPLS
- Mettre en place une topologie Hub & Spoke (lien wiki), avec un routeur client central qui va être au centre des interconnexions (on peut imaginer un équipement de sécurité qui a la charge de sécuriser les échanges d'informations)
- Permettra la connexion à internet de tous les sites clients

## Partie 1 - Le backbone opérateur

Le réseau d'opérateur, ici au centre, est composé de routeur Cisco. Ils forment une réseau IP. Les routes sont apprises via un protocole de routage interne (OSPF) (lien wiki) et j'utilise LDP pour pouvoir mettre en place mon réseau MPLS (lien wiki)

## Partie 2 - Connexion CE-PE

Pour relier les routeurs des sites clients (CE) avec le routeur d'entrée du réseau MPLS, les deux routeurs sont reliés par un réseau d'interconnexion (protocole RIP ou OSPF) qui permet aux routeurs PE d'appendre les routes directement connectées des routeurs CE (cela émule les réseaux privés des clients). Une interface sur le routeur PE correspond à un client et donc une VRF. (lien vers le wiki)

## Partie 3 - Connexion PE-PE

Les informations doivent être ensuite envoyées de routeur PE à routeur PE, plus exactement de VRF correspondant au client à VRF correspondant au même client. Pour cela des sessions BGP doivent être correctement mise en place entre PE, et les routes apprises grâce au réseau d'interconnexion redistribuées dans BGP.

## Partie 4 - Hub & Spoke

Le routeur CE1 sera le routeur central client. Il doit être annoncé comme passerelle par défaut à tous les routeurs CE. De plus, ce routeur doit connaitre l'intégralité des réseaux de chaque sites clients, afin de router les informations entre sites. Pour cela, le CE possède autant d'interfaces que de sites clients, et le PE auquel il est directement connecté possède autant de VRF que de sites clients. Les VRF contiennent les informations d'un site client, puis ces informations sont échangées avec le CE via un réseau d'interconnexion (OSPF car la gestion des zones permet de mieux contrôler le contenu des tables de routage dans cette zone où il y a de nombreux réseau d'interconnexion). 

## Partie 5 - Connexion internet

La connexion internet s'effectue via un routeur nommé "internet", et un PE nommé "IGW" (pour internet Gateway). Sur le routeur "internet" un service DHCP délivre une adresse à l'interface qui détient la VRF nommée "internet" sur IGW, ainsi qu'une route par défaut. 

Sur CE1 est effectué du NAT dynamique pour tous les paquets à destination de la passerelle par défaut de CE1.  Cette translation d'adresse va permettre d'utiliser un pool d'adresses IP publique pour accéder à internet. (lien wiki). Une nouveau réseau d'interconnexion entre CE1 et PE1 est créé pour faire transiter les informations entre CE1 et une VRF nommée "internet". Cette VRF est liée via un session BGP à l'autre VRF "internet" sur IGW.

Une dernière translation dynamique au niveau du routeur IGW permet aux paquets arrivants de PE1 d'accéder à internet.

## Conclusion

Ce projet m'a permis de mobiliser de nombreuses bases en réseau : routages, supports de communications, ainsi que les technologies associées. En plus de cela, j'ai pu découvrir la richesse d'utiliser un émulateur de réseau virtuel afin de réellement apprendre par l'exemple, et l'erreur.

De nombreuses notions sont encore à éclaircir, traffic engineering, QoS... pour un prochain commit.
