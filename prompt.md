# Projet : Serveur OTA Sécurisé pour IoT

## Contexte du projet

Je veux construire un **serveur OTA (Over-The-Air) sécurisé** open-source pour la mise à jour de parcs d'appareils IoT. Le projet doit être **self-hostable**, **conteneurisé** (Docker), et déployable sur **Proxmox**. L'objectif final est que le client lourd tourne sur un **modem Sequans 4G/5G**, mais il doit aussi fonctionner sur un **Raspberry Pi** ou un **smartphone Android/Linux** dans le même réseau que les appareils IoT.

---

## Architecture globale

```
[Serveur OTA] ←──── WireGuard VPN ────→ [Gateway Client]
                                              │
                                     ┌────────┴────────┐
                                     ▼                  ▼
                               [IoT Device A]    [IoT Device B]
                              (firmware propre)  (firmware propre)
```

**Flux de mise à jour :**

1. Le Gateway Client (RPi / téléphone / Sequans) établit un tunnel **WireGuard** vers le serveur OTA.
2. Le Client interroge le serveur pour les mises à jour disponibles pour ses appareils IoT enregistrés.
3. Le Serveur envoie le firmware signé de manière sécurisée (TLS + signature cryptographique).
4. Le Client applique la mise à jour via un mécanisme **Dual Bank / A/B Partitioning** :
      - La partition A reste active (production).
      - La mise à jour est flashée sur la partition B et testée.
      - **Rollback automatique** si les tests échouent.
      - **Rollback manuel** possible si la mise à jour est fonctionnelle mais indésirable.
5. Chaque appareil IoT a son propre firmware indépendant.

---

## Stack technique

| Composant             | Technologie                                                    |
| --------------------- | -------------------------------------------------------------- |
| Serveur OTA (backend) | **Rust** (Axum ou Actix-web)                                   |
| Frontend dashboard    | **React** (responsive, mobile-first)                           |
| Base de données       | PostgreSQL (via SQLx ou Diesel)                                |
| VPN                   | **WireGuard**                                                  |
| Transport sécurisé    | **TLS 1.3** (rustls)                                           |
| Signature firmware    | À définir (TUF ou système custom PKI)                          |
| Conteneurisation      | **Docker + Docker Compose**                                    |
| Hébergement cible     | **Proxmox** (VM ou LXC)                                        |
| CLI                   | Binaire Rust exposant les mêmes fonctions que le dashboard web |

---

## Sécurité — PKI à construire from scratch

- Mettre en place une **PKI complète** (Certificate Authority interne) :
     - CA racine (Root CA, offline de préférence)
     - CA intermédiaire pour signer les certificats serveur et device
     - Chaque appareil IoT reçoit un **certificat X.509** unique (identité cryptographique)
     - Chaque gateway client reçoit également un certificat signé par la CA
- **Signature des firmwares** : chaque package OTA est signé avant envoi (à décider entre TUF — The Update Framework — ou signature ECDSA/Ed25519 custom avec manifest JSON)
- Authentification mutuelle TLS (mTLS) entre client et serveur
- Les mises à jour ne transitent **que** via le tunnel WireGuard + TLS

---

## Fonctionnalités du serveur

- Gestion du parc d'appareils IoT (enregistrement, suppression, groupes)
- Gestion des versions de firmware par appareil ou groupe d'appareils
- Upload sécurisé des packages firmware (avec vérification de signature)
- Historique des mises à jour par appareil
- Statut en temps réel de chaque appareil (connecté, en cours de MAJ, à jour, en erreur)
- Gestion des rollbacks (déclenchement automatique ou manuel)
- API REST complète (pour le dashboard et le CLI)

---

## Dashboard Web (React)

- **Responsive** (fonctionne sur mobile et desktop)
- Deux rôles utilisateur :
     - **Admin** : configuration globale, gestion PKI, upload firmware, gestion utilisateurs, paramètres WireGuard
     - **User** : consultation du parc, déclenchement de mises à jour autorisées, visualisation des statuts, rollbacks manuels
- Pages principales :
     - Vue d'ensemble du parc (fleet dashboard)
     - Détail d'un appareil (statut, historique, firmware actuel/disponible)
     - Gestion des firmwares (upload, versioning, signature)
     - Logs et alertes
     - Gestion des utilisateurs et droits (Admin only)
     - Configuration WireGuard et PKI (Admin only)

---

## CLI (Interface ligne de commande)

Le binaire CLI doit exposer **exactement les mêmes fonctions** que le dashboard web, via des commandes du type :

```bash
ota-cli devices list
ota-cli devices status <device-id>
ota-cli firmware upload --file firmware.bin --device <device-id>
ota-cli update trigger <device-id>
ota-cli rollback trigger <device-id>
ota-cli logs --device <device-id> --tail 100
ota-cli users list   # Admin only
ota-cli vpn status
```

---

## Déploiement

- **Docker Compose** pour lancer l'ensemble (serveur Rust + PostgreSQL + frontend compilé servi par Nginx)
- Image Docker publiée sur Docker Hub pour self-hosting facile
- Documentation d'installation sur Proxmox (VM Ubuntu/Debian ou LXC)
- Support futur natif pour **Sequans Monarch 2 (GM02SP)** ou équivalent 4G/5G

---

## Échelle cible

- Utilisateur moyen : **5 à 10 appareils IoT**
- Maximum simultané raisonnable : **~100 appareils connectés** en même temps (les appareils se connectent uniquement lors d'une mise à jour via VPN, pas en permanence)
- Architecture doit rester légère et fonctionner sur une VM modeste (2 vCPU, 2 Go RAM)

---

## Ce que j'attends en priorité

1. **Structure du projet Rust** (workspace Cargo avec séparation server / cli / common)
2. **Schéma de base de données** (entités : devices, firmware, updates, users, roles, logs)
3. **Conception de la PKI** (quels outils Rust utiliser, flux de génération des certificats)
4. **Choix et conception du mécanisme de signature firmware** (TUF vs custom Ed25519 + manifest)
5. **API REST** (endpoints, authentification JWT + mTLS)
6. **Mécanisme A/B partitioning** côté client (comment le client applique et valide la mise à jour)
7. **Docker Compose** complet pour démarrer le projet

---

## Note sur la signature firmware

Une question reste ouverte à trancher en début de session :

- **TUF (The Update Framework)** : standard industriel (utilisé par Android, Uptane pour automotive). Plus robuste et auditable, mais plus complexe à implémenter.
- **Signature Ed25519 custom + manifest JSON signé** : plus rapide à coder, largement suffisant pour commencer, et migratable vers TUF ultérieurement.

La recommandation est de commencer par **Ed25519 custom** et de prévoir une interface abstraite permettant de brancher TUF plus tard.

---

## Dépendances Rust recommandées

| Usage              | Crate                                |
| ------------------ | ------------------------------------ |
| Serveur HTTP async | `axum` + `tokio`                     |
| Base de données    | `sqlx` (async, PostgreSQL)           |
| TLS                | `rustls` + `tokio-rustls`            |
| Certificats PKI    | `rcgen`                              |
| Signature Ed25519  | `ed25519-dalek`                      |
| CLI                | `clap`                               |
| Sérialisation      | `serde` + `serde_json`               |
| Auth JWT           | `jsonwebtoken`                       |
| Logs               | `tracing` + `tracing-subscriber`     |
| WireGuard (config) | `wireguard-control` ou appel système |

---

_Ce projet est destiné à être open-source. Privilegier la lisibilité du code, la modularité et une documentation inline soignée._
