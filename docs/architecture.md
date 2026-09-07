# Architecture C4 - kestachet

Ce document décrit l'architecture du système **kestachet** selon le modèle C4.

---

## 1. Niveau 1 : Contexte Système (C1)

Le diagramme de contexte présente les acteurs et les systèmes externes qui interagissent avec le système central **kestachet**.

```mermaid
flowchart TB
    U["👤 Utilisateur<br><i>(Scanne en rayon, gère le budget, complète la liste à distance)</i>"] <-- Scanne, modifie le panier, synchronise, consulte et met à jour en direct --> K@{ label: "📦 Système kestachet<br><b>[Système Logiciel Central]</b><br>Application mobile offline-first, calcul de budget,<br>gestion de liste partagée et passerelle d'ingestion" }
    K <-- Transmet les paniers validés (Asynchrone) --> DATA@{ label: "📊 Plateforme Analytics A Propos Des Biens<br><b>[Système Externe]</b><br>Traitement et valorisation des données d'achat" }
    OFF["🌐 Open Food Facts<br><b>[Système Externe]</b><br>Catalogue public des produits alimentaires"] -- Récupère les métadonnées de GTIN inconnus --> DATA

    K@{ shape: rect}
    DATA@{ shape: rect}
```

---

## 2. Niveau 2 : Conteneurs (C2)

Le diagramme de conteneurs zoome à l'intérieur du système **kestachet** pour détailler les applications exécutables, les API, les bases de données et les services d'ingestion qui le composent.

```mermaid
flowchart TB
    U["👤 Utilisateur<br><i>(Scanne en rayon, gère le budget, complète la liste à distance)</i>"]

    subgraph KESTACHET ["📦 Système kestachet [Périmètre du Système]"]
        direction TB

        APP["📱 Application Mobile<br><b>[Conteneur : Mobile App]</b><br>UI, scan de code-barres (GTIN), calcul de budget temps réel, synchronisation offline-first"]

        LOCAL_DB[("💾 Base Locale Mobile<br><b>[Conteneur : SQLite / Room]</b><br>Cache local des listes, des paniers actifs et des règles de budget (Offline-first)")]

        API["⚙️ API & Service de Synchronisation<br><b>[Conteneur : Backend Service]</b><br>Authentification, gestion des listes partagées en temps réel et validation des paniers"]

        CENTRAL_DB[("🗄️ Base de Données Centrale<br><b>[Conteneur : PostgreSQL]</b><br>Stockage persistant des comptes, listes partagées, historiques et états de paniers")]

        INGEST["📤 Passerelle d'Ingestion<br><b>[Conteneur : Queue / Worker]</b><br>Mise en file d'attente et transmission résiliente des paniers validés"]
    end

    DATA@{ label: "📊 Plateforme Analytics A Propos Des Biens<br><b>[Système Externe]</b><br>Traitement et valorisation des données d'achat" }
    OFF["🌐 Open Food Facts<br><b>[Système Externe]</b><br>Catalogue public des produits alimentaires"]

    %% Interactions Utilisateur
    U <-- "Interagit avec l'interface, scanne les articles, suit le budget" --> APP

    %% Interactions Internes Mobile
    APP <-- "Lit et écrit les données en local<br><i>(Fonctionnement hors-ligne)</i>" --> LOCAL_DB

    %% Synchronisation Réseau
    APP <-- "Synchronise les listes partagées & transmet les paniers<br><i>[HTTPS / WebSocket]</i>" --> API

    %% Persistance Centrale
    API <-- "Lecture / Écriture des données partagées<br><i>[SQL]</i>" --> CENTRAL_DB

    %% Ingestion asynchrone
    API -- "Transmet les paniers validés" --> INGEST
    INGEST -- "Envoi asynchrone des paniers validés<br><i>[HTTPS / Webhook / Queue]</i>" --> DATA

    %% Flux externe Open Food Facts
    OFF -- "Récupère les métadonnées de GTIN inconnus" --> DATA

    DATA@{ shape: rect}
```

### Description des Conteneurs

| Conteneur | Rôle & Responsabilités | Technologie indicative |
| :--- | :--- | :--- |
| **Application Mobile** | Interface principale de l'utilisateur. Embarque la logique de scan caméra, de calcul de budget en temps réel et de gestion de panier, utilisable sans connexion internet. | Flutter, React Native, Kotlin ou Swift |
| **Base Locale Mobile** | Assure la capacité **Offline-first**. Toutes les actions en magasin sont enregistrées immédiatement en local avant synchronisation. | SQLite / Room / WatermelonDB |
| **API Backend & Sync** | Reçoit les modifications de listes partagées, réconcilie les écritures concurrentes et enregistre les paniers validés. | Java (Spring Boot), Node.js, Go, etc. |
| **Base Centrale** | Stocke les profils, les listes de courses partagées entre membres, et l'historique complet des données du service. | PostgreSQL |
| **Passerelle d'Ingestion** | Découple kestachet de la plateforme Analytics externe via une file d'attente pour garantir qu'aucune donnée d'achat validée n'est perdue en cas d'indisponibilité du tiers. | RabbitMQ / Redis Queue / Worker Service |
