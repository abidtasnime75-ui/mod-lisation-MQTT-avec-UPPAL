# Modélisation formelle de MQTT v3.1.1 avec UPPAAL

Modèle à automates temporisés du protocole MQTT v3.1.1 (Publisher, Broker,
Subscriber), vérifié par model checking (CTL), avec deux attaques réelles
et leurs contre-mesures.

![UPPAAL](https://img.shields.io/badge/UPPAAL-<version>-blue)
![MQTT](https://img.shields.io/badge/MQTT-v3.1.1-green)
![License](https://img.shields.io/badge/license-<type>-lightgrey)

## Résultats en bref

| Scénario | Fichier | Verdict |
|---|---|---|
| Fonctionnement normal | `modèle nominal.xml` | 12/12 propriétés satisfaites |
| Attaque 1 : sniffing (sans TLS) | `modèle avec attaque1.xml` | Interception **possible** → corrigée par TLS |
| Attaque 2 : CVE-2017-7654 | `modèle avec attaque2.xml` | Bypass d'authentification **possible** |
| Attaque 2 + contre-mesure | `modèle avec attaque2 avec sol.xml` | Attaque **non atteignable** |

## Architecture du modèle

```
 ┌───────────┐  CONNECT/PUBLISH   ┌────────┐   PUBLISH    ┌────────────┐
 │ Publisher │ ─────────────────► │ Broker │ ───────────► │ Subscriber │
 └───────────┘ ◄───────────────── └────────┘ ◄─────────── └────────────┘
      CONNACK/PUBACK                 ▲   SUBSCRIBE/SUBACK
                                     │
                              ┌──────┴──────┐
                              │  Attaquant  │  (modèles attaque1 / attaque2)
                              └─────────────┘
```

- 12 types de paquets modélisés comme canaux de synchronisation
- Timers : timeout de connexion et Keep Alive (PINGREQ/PINGRESP)
- QoS 0, 1 et 2 : trois chemins distincts après PUBLISH

## Contenu du dépôt

| Fichier | Rôle |
|---|---|
| `modèle nominal.xml` | Protocole sans attaquant |
| `modèle avec attaque1.xml` | + attaquant passif (pas de TLS) |
| `modèle avec attaque2.xml` | + exploitation de CVE-2017-7654 |
| `modèle avec attaque2 avec sol.xml` | Attaque 2 avec authentification stricte et limite de connexions |

## Utilisation

1. Installer [UPPAAL](https://uppaal.org) (version testée : `<x.y.z>`)
2. Ouvrir un fichier `.xml` : *File → Open*
3. Onglet **Verifier** → sélectionner une requête → **Check**
4. Vert = propriété satisfaite, rouge = violée

## Propriétés vérifiées (modèle nominal)

| # | Type | Requête CTL | Signification | Résultat |
|---|---|---|---|---|
| P1 | Deadlock | `A[] not deadlock` | Le système ne se bloque jamais | ✅ |
| P2 | Reachability | `E<> <...>` | Connexion atteignable | ✅ |
| P3 | Safety | `A[] <...>` | Pas de PUBLISH sans authentification | ✅ |
| … | … | … | … | … |

## Attaques et contre-mesures

### Attaque 1 : interception passive (pas de TLS)
- **Principe :** un attaquant lit et altère les messages en clair
- **Propriété violée :** `<requête>`
- **Contre-mesure :** TLS → la propriété devient non atteignable ✅

### Attaque 2 : CVE-2017-7654 (Mosquitto)
- **Principe :** connexion et authentification sans mot de passe valide, d'où un déni de service
- **Propriété violée :** `<requête>`
- **Contre-mesure :** vérification stricte du mot de passe + limite de connexions ✅

## Références
- [Spécification MQTT v3.1.1 (OASIS)](https://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)
- [CVE-2017-7654](https://nvd.nist.gov/vuln/detail/CVE-2017-7654)

## Auteurs
 Tasnim ABID /Wala Ouaili
