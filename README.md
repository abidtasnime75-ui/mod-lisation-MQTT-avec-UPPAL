# Modélisation du protocole MQTT v3.1.1 avec UPPAAL

Ce dépôt contient la modélisation formelle du protocole MQTT v3.1.1 (outil UPPAAL), incluant un modèle nominal et deux attaques avec leurs contre-mesures.

## 2. Référence officielle de la spécification MQTT v3.1.1

La modélisation se base sur la spécification officielle OASIS, *MQTT Version 3.1.1* (29 octobre 2014), disponible ici : https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html

## 3. Extraction des comportements pertinents du protocole

### 3.1 Types de messages MQTT
Les principaux paquets de contrôle du protocole (CONNECT, CONNACK, SUBSCRIBE, SUBACK, PUBLISH, PUBACK, PUBREC, PUBREL, PUBCOMP, PINGREQ, PINGRESP, DISCONNECT) ont été identifiés et traduits en canaux de synchronisation UPPAAL.

### 3.2 États et transitions des entités
Chaque entité (Publisher, Broker, Subscriber) a été modélisée par un automate avec ses propres états (connexion, authentification, publication/réception, déconnexion) et les transitions correspondant aux échanges de paquets MQTT.

### 3.3 Contraintes temporelles et timers
Des horloges ont été ajoutées pour représenter le timeout de connexion et le mécanisme de Keep Alive (PINGREQ/PINGRESP) prévus par la norme.

### 3.4 Niveaux de qualité de service (QoS 0, 1, 2)
Les trois niveaux de QoS ont été modélisés comme trois chemins distincts après publication : sans accusé de réception (QoS 0), avec un accusé simple (QoS 1), et avec un handshake en plusieurs étapes garantissant une livraison unique (QoS 2).

## 4. Description générale du modèle nominal UPPAAL

### 4.1 Template Publisher — états, canaux et variables
Automate représentant le client qui publie les données : connexion, authentification, puis publication selon le QoS choisi.

### 4.2 Template Broker — états, canaux et logique de contrôle
Automate central qui gère l'authentification des deux clients et relaie les messages du Publisher vers le Subscriber.

### 4.3 Template Subscriber — états, canaux et variables
Automate représentant le client abonné : connexion, authentification, abonnement au topic, réception des messages.

## 5. Vérifications générales — Propriétés et requêtes CTL

### 5.1 Absence de deadlock
Vérifie que le système ne se bloque jamais complètement.

### 5.2 Propriétés de reachability
Vérifient que les comportements attendus (connexion, publication, réception, QoS 1/2, déconnexion) sont bien atteignables.

### 5.3 Propriétés de sécurité (safety)
Vérifient qu'aucune publication ni abonnement n'a lieu sans authentification préalable.

### 5.4 Tableau récapitulatif des requêtes CTL
L'ensemble des propriétés vérifiées (P1 à P12) est regroupé et validé dans le modèle nominal.

## 6. Vulnérabilités CVE et modélisation des attaques

### 6.1 Interception passive (Sniffing) — absence de chiffrement TLS
Modélise un attaquant capable d'intercepter et d'altérer les messages en l'absence de chiffrement TLS.

### 6.2 Bypass d'authentification et déni de service (CVE-2017-7654)
Modélise la vulnérabilité réelle CVE-2017-7654 du broker Mosquitto, permettant à un attaquant de se connecter et de s'authentifier sans mot de passe valide.

## 7. Contre-mesures et validation formelle

### 7.1 Activation du chiffrement TLS — Attaque 1
L'activation du chiffrement TLS est vérifiée formellement : les propriétés d'interception et d'altération des messages deviennent non atteignables.

### 7.2 Authentification stricte et limitation de connexions — CVE-2017-7654
L'ajout d'une vérification stricte du mot de passe et d'une limite de connexions est vérifié formellement : l'attaquant ne peut plus se connecter ni s'authentifier.
