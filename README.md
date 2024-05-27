# Firmware pour le Master Module V1

Bienvenue sur la page GitHub dédiée au firmware du Master Module V1. Ce firmware est essentiel pour l'intégration des fonctionnalités avancées du powertrain LMX Projects destiné aux véhicules intermédiaires. Voici un aperçu des fonctionnalités offertes par ce firmware et des capacités du Master Module V1.

## Fonctionnalités Principales

Ce firmware permet de se connecter via Bluetooth à une application mobile, offrant ainsi un contrôle complet et une surveillance en temps réel des composants du powertrain. Les fonctionnalités incluent :

- **Sélection et Contrôle des Contrôleurs** :
  - Choix du contrôleur sur architecture VESC à piloter via CAN.
  - Modes de contrôle disponibles : ERPM, courant, frein.
  - Affichage des ERPM des deux contrôleurs connectés.

- **Contrôle du Moteur** :
  - Contrôle du RPM du moteur à l'aide d'un accélérateur physique.
  - Gestion des lampes (clignotement).

## Fonctionnalités Futures
- Détection de la position de la béquille (remontée ou abaissée).
- Changement de mode sur le contrôleur via un bouton dédié.
- Calcul de la vitesse avec un deuxième système de calcul pour répondre aux exigences L6.

## Caractéristiques Techniques du Master Module V1

- **Alimentation** :
  - Tension d'entrée : 17V à 75V.
  - Sorties disponibles : 12V 4A, 5V 1A, 3,3V 0,9A.

- **Protocole de Communication** :
  - CAN (via transceiver TJA1441AT_0Z).
  - UART.
  - Bluetooth/Wifi.

- **Entrées et Sorties** :
  - Plusieurs GPIOs pour diverses fonctions (lampes, accélérateur, klaxon, etc.).
  - Pins dédiées pour UART et programmation via l'outil de téléchargement flash ESP32.

- **ESP32 Intégré** :
  - Gestion des bus CAN et des sorties lumineuses.
  - Débogage via TX/RX.

## Contact et Support

Pour toute question ou support technique, veuillez nous contacter à :

- **Téléphone** : +33 4 69 31 04 29
- **Email** : [projects@lmxbikes.com](mailto:projects@lmxbikes.com)
- **Site Web** : [www.lmxbikes.com](http://www.lmxbikes.com)
