---
tags:
  - C4Model
  - Architecture
  - Diagrams
  - Kestachet
  - C1
  - C2
  - C3
  - C4
  - Mermaid
date: 2026-09-08
course: 4.1 | Projet de synthèse — Kestachet
subject: Dossier Complet des Schémas d'Architecture (Hexagonale & C1 à C4)
---

# 📐 Kestachet — Dossier Complet des Schémas (Hexagonale & C1 à C4)

> [!NOTE] **Présentation des 4 Niveaux de Zoom C4**
> Le **C4 Model** (créé par Simon Brown) permet de décrire l'architecture logicielle comme une carte Google Maps :
> - **Niveau 1 (C1) — Contexte Système** : La vue d'ensemble (qui utilise le système et avec quels services externes il communique).
> - **Niveau 2 (C2) — Conteneurs** : Les grosses briques logicielles déployables (Apps mobiles, Web, APIs, Bases de données).
> - **Niveau 3 (C3) — Composants** : Le zoom à l'intérieur d'un conteneur spécifique (ici le service d'API).
> - **Niveau 4 (C4) — Code** : Le zoom microscopique sur les classes UML et l'algorithme métier.

---


---

## ⬡ 0. Schéma d'Architecture Hexagonale (Étape 1 : Application Mobile)

Ce schéma représente l'**Architecture Hexagonale (*Ports & Adaptateurs*)** de l'application mobile, demandée à l'**Étape 1** du sujet. 
Il illustre la règle d'or : **le cœur métier est pur et isolé**, ce sont les détails techniques (caméra, base SQLite, réseau) qui s'adaptent au métier grâce à l'inversion de dépendance.

```mermaid
flowchart LR
    subgraph MONDE_EXTERIEUR_ENTREE ["Monde Extérieur (Entrée)"]
        USER["👤 Doigt Utilisateur"]
        CAM["📷 Caméra Téléphone"]
    end

    subgraph ADAPTATEURS_ENTREE ["Adaptateurs d'Entrée (Driving)"]
        UI_ADAPTER["Adaptateur UI<br><i>[Jetpack Compose / Flutter]</i>"]
        CAM_ADAPTER["Adaptateur Caméra<br><i>[Google ML Kit API]</i>"]
    end

    subgraph HEXAGONE ["⬡ CŒUR MÉTIER PUR (L'Hexagone)"]
        direction TB
        
        subgraph PORTS_ENTREE ["Ports d'Entrée (Interfaces)"]
            PORT_SCAN["<<interface>><br>ScanArticlePort"]
            PORT_BUDGET["<<interface>><br>CalculBudgetPort"]
        end

        subgraph DOMAINE ["🧠 Domaine Métier (Pur)"]
            PANIER["Panier (Cart)"]
            MOTEUR["Moteur de Calcul du Budget<br><b>[Pattern Strategy]</b><br>Vert < 80% | Orange | Rouge >= 100%"]
        end

        subgraph PORTS_SORTIE ["Ports de Sortie (Interfaces)"]
            PORT_REPO["<<interface>><br><b>[Pattern Repository]</b><br>PanierRepositoryPort"]
            PORT_SYNC["<<interface>><br>SyncGatewayPort"]
            PORT_BARCODE_STRAT["<<interface>><br><b>[Pattern Strategy]</b><br>BarcodeDecoderPort"]
        end
    end

    subgraph ADAPTATEURS_SORTIE ["Adaptateurs de Sortie (Driven)"]
        SQLITE_ADAPTER["Adaptateur SQLite<br><i>[Room / SQLite]</i>"]
        WS_ADAPTER["Adaptateur Réseau<br><i>[WebSocket / HTTPS]</i>"]
        MLKIT_STRAT["Adaptateur MLKit Natif"]
    end

    subgraph MONDE_EXTERIEUR_SORTIE ["Monde Extérieur (Sortie)"]
        FLASH_STORAGE[("💾 Mémoire Flash du téléphone")]
        SERVER_CLOUD["☁️ Serveur Kestachet"]
    end

    %% Connexions d'Entrée
    USER --> UI_ADAPTER
    CAM --> CAM_ADAPTER
    UI_ADAPTER --> PORT_BUDGET
    CAM_ADAPTER --> PORT_SCAN

    %% Liens internes vers Domaine
    PORT_SCAN --> PANIER
    PORT_BUDGET --> MOTEUR
    PANIER --> PORT_REPO
    PANIER --> PORT_SYNC
    PORT_SCAN --> PORT_BARCODE_STRAT

    %% Inversion de dépendance (Les adaptateurs implémentent les ports)
    SQLITE_ADAPTER -.->|implémente| PORT_REPO
    WS_ADAPTER -.->|implémente| PORT_SYNC
    MLKIT_STRAT -.->|implémente| PORT_BARCODE_STRAT

    %% Connexions vers le matériel et systèmes de Sortie
    SQLITE_ADAPTER --> FLASH_STORAGE
    WS_ADAPTER --> SERVER_CLOUD
```

### 🗣️ Ce qu'on dit au jury sur l'Architecture Hexagonale (en 30 secondes) :
> *"Voici notre architecture hexagonale pour l'application mobile. Le cœur au centre est du code pur sans aucune dépendance : il gère le panier et le calcul budgétaire. La caméra et la base SQLite sont des adaptateurs interchangeables branchés sur des ports (interfaces). Grâce au pattern **Strategy**, si demain Google change son SDK de scan caméra, nous changeons uniquement l'adaptateur sans toucher à une seule ligne du calcul de budget."*

## 🌍 1. Niveau 1 : Contexte Système (C1)

Le schéma de contexte présente les acteurs humains et les systèmes informatiques externes en relation avec l'application **kestachet**.

```mermaid
flowchart TB
    U["👤 Utilisateur<br><i>(Scanne en rayon, gère le budget, complète la liste)</i>"]

    K["📦 Système kestachet<br><b>[Système Logiciel Central]</b><br>Application mobile et web de courses partagées,<br>calcul de budget hors-ligne et synchronisation"]

    DATA["📊 Plateforme Analytics A Propos Des Biens<br><b>[Système Externe]</b><br>Traitement et valorisation des données d'achat"]

    OFF["🌐 Open Food Facts<br><b>[Système Externe]</b><br>Catalogue public des produits alimentaires"]

    %% Interactions C1
    U <-->|"Scanne en rayon, consulte et met à jour en direct"| K
    K -->|"Transmet les paniers validés (Asynchrone)"| DATA
    OFF -->|"Récupère les métadonnées de GTIN inconnus"| DATA
```

### 🗣️ Ce qu'on dit au jury sur le C1 (en 20 secondes) :
> *"Le niveau 1 montre notre périmètre : l'utilisateur interagit avec le système central **kestachet** pour scanner ses articles et suivre son budget. Quand un panier est validé, les données sont transmises de manière asynchrone à la plateforme analytique d'**A Propos Des Biens**, qui s'enrichit automatiquement via le catalogue externe **Open Food Facts** pour les produits inconnus."*

---

## 📦 2. Niveau 2 : Conteneurs (C2)

Le schéma de conteneurs zoome dans le système central **kestachet** pour identifier les applications, les bases de données et les protocoles réseau utilisés.

```mermaid
flowchart TB
    subgraph USERS ["Acteurs"]
        U["👤 Utilisateur<br><i>(Scanne en rayon, gère le budget, complète la liste)</i>"]
    end

    subgraph K ["📦 Système kestachet (Système Logiciel Central)"]
        direction TB
        subgraph MOBILE ["Terminal Mobile"]
            APP["📱 Application Mobile<br><b>[Container: Kotlin / Flutter]</b><br>Scan caméra, calcul de budget hors-ligne,<br>gestion du cache local"]
            LOCAL_DB[("💾 Base locale<br><b>[Container: SQLite]</b><br>Cache articles et journal des modifications")]
        end
        subgraph WEB ["Terminal Web"]
            AP["💻 Application Web<br><b>[Container: HTTP / React]</b><br>Consultation et mise à jour du panier à distance"]
        end

        API["⚡ Service d'API & Synchronisation<br><b>[Container: Spring Boot / Node]</b><br>Synchronisation temps réel du foyer,<br>gestion des listes et transmission des paniers"]

        PROD_DB[("🗄️ BDD Opérationnelle<br><b>[Container: PostgreSQL]</b><br>Comptes utilisateurs, listes actives<br>et catalogue de référence")]
    end

    subgraph EXTERNALS ["Systèmes Externes"]
        DATA["📊 Plateforme Analytics A Propos Des Biens<br><b>[Système Externe]</b><br>Traitement et valorisation des données d'achat"]
        OFF["🌐 Open Food Facts<br><b>[Système Externe]</b><br>Catalogue public des produits alimentaires"]
    end

    %% Relations Utilisateur <--> Kestachet
    U <-->|"Scanne, modifie le panier, consulte"| APP
    APP <-->|"Lecture / Écriture locale sans réseau"| LOCAL_DB

    U <-->|"Consulte et met à jour à distance"| AP
    AP <-->|"Synchronisation (WSS / HTTPS)"| API

    %% Relations internes Kestachet
    APP <-->|"Synchronisation dès reconnexion (WSS / HTTPS)"| API
    API <-->|"Persistance des listes partagées (SQL)"| PROD_DB

    %% Relations Kestachet <--> Externes
    API <-->|"Transmet les paniers validés (Asynchrone)"| DATA
    OFF -->|"Récupère les métadonnées de GTIN inconnus"| DATA
```

### 🗣️ Ce qu'on dit au jury sur le C2 (en 30 secondes) :
> *"Le niveau 2 détaille nos conteneurs : sur le mobile, une base **SQLite** permet à l'utilisateur de scanner et calculer son budget même en sous-sol sans réseau (**Offline-First**). Dès qu'il capte la 4G, l'application synchronise ses modifications via WebSockets/HTTPS avec notre **Service d'API**. Ce service met à jour la base centrale **PostgreSQL** pour la famille et transfère les paniers validés à la plateforme **Analytics** en arrière-plan."*

---

## 🧩 3. Niveau 3 : Composants (C3)

Dans notre système, deux conteneurs principaux méritent un zoom C3 :
- **3.1 L'Application Mobile** (le client qui tourne hors-ligne en rayon).
- **3.2 Le Service d'API & Synchronisation** (le serveur central qui réconcilie les données).

---

### 3.1 Schéma C3 : Zoom sur l'"Application Mobile" [Container: Kotlin / Flutter]

Ce schéma fait un zoom à l'intérieur du conteneur **`📱 Application Mobile`** pour montrer ses composants logiciels internes et comment ils communiquent avec la caméra, l'écran, SQLite et le serveur :

```mermaid
flowchart TB
    %% Périphériques et acteurs extérieurs
    subgraph EXTERIEUR_C2 ["Extérieur (Depuis le C2)"]
        U_ACTOR["👤 Utilisateur"]
        CAM_HARDWARE["📷 Caméra du Téléphone"]
        SQLITE_EXT[("💾 Base locale SQLite<br><i>[Container: SQLite]</i>")]
        API_EXT["⚡ Service d'API & Synchronisation<br><i>[Container: Spring Boot / Node]</i>"]
    end

    %% ZOOM C3 SUR L'APPLICATION MOBILE
    subgraph APP_CONTAINER ["📱 Application Mobile [Container: Kotlin / Flutter]"]
        direction TB

        subgraph UI_COMPONENTS ["1. Couche Interface Utilisateur (UI)"]
            SCAN_SCREEN["📸 Écran de Scan<br><b>[Component: View]</b><br>Aperçu caméra et visée du code-barres"]
            CART_SCREEN["🛒 Écran du Panier & Liste<br><b>[Component: View]</b><br>Articles, quantités et prix saisis"]
            BUDGET_GAUGE_WIDGET["🚦 Jauge de Budget Visuelle<br><b>[Component: Widget]</b><br>Vert (<80%), Orange, Rouge (>=100%)"]
            VIEW_MODEL["📊 Cart & Budget ViewModel<br><b>[Component: State Manager / Observer]</b><br>Diffuse l'état réactif aux écrans"]
        end

        subgraph CORE_COMPONENTS ["2. Couche Métier (Logique Pure)"]
            SCAN_SERVICE["🔍 Décodeur de Code-barres<br><b>[Component: Service / Strategy]</b><br>Traduit l'image en GTIN via Google ML Kit"]
            CART_SERVICE["🛒 Gestionnaire de Panier<br><b>[Component: Service]</b><br>Ajoute, modifie les articles et vérifie le panier"]
            BUDGET_CALCULATOR["⚙️ Calculateur Budgétaire<br><b>[Component: Service / Strategy]</b><br>Calcule le total et détermine le statut couleur"]
        end

        subgraph DATA_COMPONENTS ["3. Couche Persistance & Synchronisation Locale"]
            LOCAL_REPO["💾 Dépôt de Données Local<br><b>[Component: Repository]</b><br>Lit et écrit les articles dans SQLite"]
            SYNC_WORKER["🔄 Moteur de Synchronisation Déconnecté<br><b>[Component: Background Service]</b><br>Empile les deltas et surveille le réseau"]
            NETWORK_CLIENT["🌐 Client Réseau Distant<br><b>[Component: HTTP / WebSocket Client]</b><br>Pousse les deltas dès retour de la 4G"]
        end
    end

    %% Flux Utilisateur et Matériel
    U_ACTOR -->|"Scanne un article"| SCAN_SCREEN
    U_ACTOR -->|"Modifie une quantité / consulte"| CART_SCREEN
    CAM_HARDWARE -->|"Flux vidéo brut"| SCAN_SCREEN

    %% Flux UI vers Métier
    SCAN_SCREEN -->|"Trame photo"| SCAN_SERVICE
    SCAN_SERVICE -->|"GTIN décodé (ex: 3017620422003)"| CART_SERVICE
    CART_SCREEN -->|"Action utilisateur"| VIEW_MODEL
    VIEW_MODEL -->|"Demande de modification"| CART_SERVICE

    %% Métier interne
    CART_SERVICE -->|"Transmet le panier"| BUDGET_CALCULATOR
    BUDGET_CALCULATOR -->|"Nouveau total & statut couleur"| VIEW_MODEL
    VIEW_MODEL -.->|"Met à jour l'affichage"| BUDGET_GAUGE_WIDGET
    VIEW_MODEL -.->|"Met à jour la liste"| CART_SCREEN

    %% Métier vers Données locales
    CART_SERVICE -->|"Enregistre le scan en local"| LOCAL_REPO
    LOCAL_REPO -->|"Écriture immédiate (10 ms)"| SQLITE_EXT
    LOCAL_REPO -->|"Ajoute à la file d'attente (Outbox)"| SYNC_WORKER

    %% Synchro vers le serveur
    SYNC_WORKER -->|"Transmet les deltas en attente"| NETWORK_CLIENT
    NETWORK_CLIENT -->|"WSS / HTTPS dès retour 4G"| API_EXT
```

#### Rôle des composants de l'application mobile (à dire au jury) :
1. **Écran de Scan & Décodeur (`SCAN_SERVICE`)** : Capture le flux de la caméra physique et décode le code-barres en GTIN sans bloquer l'interface.
2. **Gestionnaire de Panier & Calculateur de Budget (`BUDGET_CALCULATOR`)** : Le cœur métier. Il calcule le montant et bascule la jauge au vert, orange ou rouge directement sur le processeur du smartphone.
3. **Dépôt Local (`LOCAL_REPO`)** : Persiste chaque article dans la base **SQLite** en 10 millisecondes. L'application ne dépend d'aucun serveur pour fonctionner.
4. **Moteur de Synchronisation (`SYNC_WORKER`)** : Enregistre chaque action dans un journal local. Dès que la 4G revient, il pousse les modifications vers l'API sans perturber l'utilisateur.

---

### 3.2 Schéma C3 : Zoom sur le "Service d'API & Synchronisation" [Container: Spring Boot / Node]

```mermaid
flowchart TB
    %% Liens entrants depuis le C2
    subgraph CLIENTS ["Clients (Depuis C2)"]
        APP_IN["📱 Application Mobile<br><i>[Container: Kotlin/Flutter]</i>"]
        WEB_IN["💻 Application Web<br><i>[Container: React/HTTP]</i>"]
    end

    %% ZOOM C3 SUR LE SERVICE D'API
    subgraph API_CONTAINER ["⚡ Service d'API & Synchronisation [Container: Spring Boot / Node]"]
        direction TB

        subgraph ENTREE ["1. Couche Contrôleur (Entrée)"]
            WS_CTRL["🔌 WebSocket Sync Handler<br><b>[Component: Controller]</b><br>Gère les flux bidirectionnels WSS temps réel"]
            REST_CTRL["🌐 REST API Controller<br><b>[Component: Controller]</b><br>Endpoints HTTP (Auth, validation finale)"]
            AUTH_GUARD["🛡️ Security & JWT Filter<br><b>[Component: Middleware]</b><br>Vérifie l'identité de l'utilisateur et du foyer"]
        end

        subgraph METIER ["2. Couche Métier (Services)"]
            SYNC_ENGINE["⚖️ Moteur de Résolution de Conflits<br><b>[Component: Service / CRDT]</b><br>Fusionne les deltas hors-ligne (1 + 1 = 2)"]
            CART_SERVICE["🛒 Service Panier & Listes<br><b>[Component: Service]</b><br>Gère les articles, totaux et règles de foyer"]
        end

        subgraph SORTIE ["3. Couche Accès Données & Sortie"]
            DATA_REPO["💾 Data Access Repository<br><b>[Component: Repository]</b><br>Requêtes SQL optimisées vers PostgreSQL"]
            ASYNC_DISPATCHER["📤 Async Analytics Dispatcher<br><b>[Component: Queue Producer]</b><br>Dépose les paniers validés sans bloquer le client"]
        end
    end

    %% Cibles sortantes vers C2
    subgraph SORTIES_C2 ["Persistance & Systèmes Externes (Depuis C2)"]
        DB_OUT[("🗄️ BDD Opérationnelle<br><i>[PostgreSQL]</i>")]
        DATA_OUT["📊 Plateforme Analytics<br><i>[A Propos Des Biens]</i>"]
    end

    %% Flux internes
    APP_IN -->|"WSS (Deltas de scans)"| WS_CTRL
    WEB_IN -->|"HTTPS / WSS"| REST_CTRL

    WS_CTRL --> AUTH_GUARD
    REST_CTRL --> AUTH_GUARD

    AUTH_GUARD --> SYNC_ENGINE
    AUTH_GUARD --> CART_SERVICE

    SYNC_ENGINE -->|"Applique les deltas fusionnés"| CART_SERVICE

    CART_SERVICE -->|"Lit et écrit les listes"| DATA_REPO
    CART_SERVICE -->|"Lors de la validation du panier"| ASYNC_DISPATCHER

    DATA_REPO -->|"SQL (JDBC/Pool)"| DB_OUT
    ASYNC_DISPATCHER -->|"Envoi asynchrone (Queue/HTTP)"| DATA_OUT
```

### 🗣️ Ce qu'on dit au jury sur le C3 (en 30 secondes) :
> *"Le niveau 3 zoome sur l'API : les requêtes arrivent par les contrôleurs **WebSocket** et **REST**, filtrées par un module de sécurité **JWT**. Le composant clé est le **Moteur de Résolution de Conflits** qui fusionne automatiquement les ajouts d'articles sans écrasement (CRDT). Enfin, le **Repository** persiste l'état dans PostgreSQL et le **Dispatcher Asynchrone** envoie les paniers vers Analytics sans faire attendre le client."*

---

## 💻 4. Niveau 4 : Code (C4)

Le schéma de code zoome à l'échelle des classes orientées objet (diagramme de classes UML) sur le composant métier **`Moteur de Résolution de Conflits`** et **`Service Panier`**.

```mermaid
classDiagram
    direction TB

    class Cart {
        +UUID id
        +UUID householdId
        +Double budgetTarget
        +List~CartItem~ items
        +addItem(CartItem item)
        +updateQuantity(String gtin, int delta)
        +calculateTotal() Double
        +getBudgetStatus() BudgetStatus
    }

    class CartItem {
        +String gtin
        +String label
        +Double price
        +int quantity
        +Long lastUpdatedTimestamp
        +incrementQuantity(int delta)
        +updatePrice(Double newPrice, Long timestamp)
    }

    class BudgetStatus {
        <<enumeration>>
        GREEN
        ORANGE
        RED
    }

    class ConflictResolutionService {
        +mergeDeltas(Cart cart, List~SyncDelta~ deltas) Cart
        -applyPNCounter(CartItem item, int delta)
        -resolvePriceLWW(CartItem item, Double newPrice, Long timestamp)
    }

    class SyncDelta {
        +UUID operationId
        +String gtin
        +String actionType
        +int deltaQuantity
        +Double price
        +Long timestampNtp
    }

    class CartRepositoryPort {
        <<interface>>
        +findById(UUID cartId) Cart
        +save(Cart cart) void
    }

    class PostgresCartRepository {
        -JdbcTemplate jdbcTemplate
        +findById(UUID cartId) Cart
        +save(Cart cart) void
    }

    %% Relations UML
    Cart "1" *-- "0..*" CartItem : contient
    Cart ..> BudgetStatus : évalue
    ConflictResolutionService ..> Cart : réconcilie
    ConflictResolutionService ..> SyncDelta : consomme
    PostgresCartRepository ..|> CartRepositoryPort : implémente
```

### Algorithme concret d'implémentation (Java / Spring Boot) :

```java
@Service
public class ConflictResolutionService {

    /**
     * Fusionne les deltas hors-ligne reçus du smartphone sans écraser les données.
     */
    public Cart mergeDeltas(Cart cart, List<SyncDelta> incomingDeltas) {
        for (SyncDelta delta : incomingDeltas) {
            // Récupère l'article existant ou le crée s'il est nouveau
            CartItem item = cart.findItemByGtin(delta.getGtin())
                .orElseGet(() -> cart.createItem(delta.getGtin()));

            // 1. Règle CRDT (PN-Counter) : on additionne le delta (+1, +2, etc.)
            item.incrementQuantity(delta.getDeltaQuantity());

            // 2. Règle Last-Write-Wins (LWW) sur le prix si saisi manuellement
            if (delta.getPrice() != null && delta.getTimestampNtp() > item.getLastUpdatedTimestamp()) {
                item.updatePrice(delta.getPrice(), delta.getTimestampNtp());
            }
        }
        return cart;
    }
}
```

### 🗣️ Ce qu'on dit au jury sur le C4 (en 20 secondes) :
> *"Le niveau 4 illustre notre modèle objet : la classe `Cart` regroupe des `CartItem`. La méthode `mergeDeltas` applique l'addition mathématique sur les quantités : deux téléphones ayant scanné 1 lait chacun produisent $1 + 1 = 2$ bouteilles. L'interface `CartRepositoryPort` respecte l'inversion de dépendance pour isoler le stockage SQL."*
