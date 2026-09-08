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

Ce schéma montre l'organisation interne du conteneur Backend selon l'**architecture en couches classique** articulée autour de **4 composants fondamentaux** :
- **Controller** : Réception et validation des requêtes HTTP/WSS.
- **Service** : Logique métier et orchestration (règles de panier, fusion CRDT).
- **Repository** : Accès aux données et requêtes SQL vers PostgreSQL.
- **Domain Model** : Entités métier manipulées par le système.

```mermaid
flowchart TB
    %% Liens entrants depuis le C2
    subgraph CLIENTS ["Clients (Depuis C2)"]
        APP_IN["📱 Application Mobile<br><i>[Terminal Mobile]</i>"]
        WEB_IN["💻 Application Web<br><i>[Terminal Web]</i>"]
    end

    %% ZOOM C3 SUR LE SERVICE D'API
    subgraph API_CONTAINER ["⚡ Service d'API & Synchronisation [Container: Spring Boot / Node]"]
        direction TB

        CONTROLLER["🎮 Controller (REST & WebSocket)<br><b>[Component: Controller]</b><br>• Reçoit les requêtes HTTP & flux WSS<br>• Valide les entrées / DTOs<br>• Filtre de sécurité JWT & délègue au Service"]

        SERVICE["⚙️ Service Métier (Cart & Sync Service)<br><b>[Component: Service]</b><br>• Contient la logique métier et orchestre les opérations<br>• Résout les conflits hors-ligne (CRDT)<br>• Applique les règles budgétaires du foyer"]

        REPOSITORY["💾 Repository (Data Access Layer)<br><b>[Component: Repository]</b><br>• Gère l'accès aux données<br>• Encapsule les requêtes SQL vers PostgreSQL<br>• Isole la persistance du reste du code"]

        DOMAIN["📦 Domain Model (Entités Métier)<br><b>[Component: Domain Model]</b><br>• User & Household (Comptes & Foyers)<br>• Cart (Panier & règles de budget)<br>• CartItem & Product (Articles & GTIN)"]

        ASYNC_DISPATCHER["📤 Async Analytics Dispatcher<br><b>[Component: Queue Producer]</b><br>• Dépose les paniers validés en file d'attente (asynchrone)<br>• Découple l'API de la plateforme Analytics"]
    end

    %% Cibles sortantes vers C2
    subgraph SORTIES_C2 ["Persistance & Systèmes Externes (Depuis C2)"]
        DB_OUT[("🗄️ BDD Opérationnelle<br><i>[PostgreSQL]</i>")]
        DATA_OUT["📊 Plateforme Analytics<br><i>[A Propos Des Biens]</i>"]
    end

    %% Flux internes
    APP_IN -->|"1. Pousse deltas & scans (WSS)"| CONTROLLER
    WEB_IN -->|"1. Requêtes HTTP (Auth, listes)"| CONTROLLER

    CONTROLLER -->|"2. Valide & délègue la requête"| SERVICE
    SERVICE <-->|"3. Manipule les entités"| DOMAIN
    SERVICE <-->|"4. Lit & sauvegarde l'état"| REPOSITORY
    REPOSITORY <-->|"5. Requêtes SQL"| DB_OUT

    SERVICE -->|"6. Notifie la validation du panier"| ASYNC_DISPATCHER
    ASYNC_DISPATCHER -->|"7. Envoi asynchrone"| DATA_OUT
```

#### Les 4 Composants Fondamentaux du C3 (à dire au jury) :

| Composant | Rôle standard | Rôle concret dans **Kestachet** |
| :--- | :--- | :--- |
| **🎮 Controller** | Reçoit les requêtes HTTP/WSS, valide les entrées, délègue au service. | Gère les endpoints REST et la connexion WebSocket, valide les DTOs (GTIN, prix, quantités) et vérifie les tokens JWT. |
| **⚙️ Service** | Contient la logique métier, orchestre les opérations. | Résout les conflits de synchronisation (CRDT), recalcule le budget, applique les règles de gestion du foyer et déclenche les événements. |
| **💾 Repository** | Gère l'accès aux données, encapsule les requêtes SQL. | Fournit les méthodes d'accès (CRUD via JPA/Hibernate/Prisma) et exécute les requêtes SQL vers PostgreSQL. |
| **📦 Domain Model** | Représente les entités métier au cœur de l'application. | Objets métier purs : `User`, `Household` (Foyer), `Cart` (Panier), `CartItem` (Article scanné), `Product` (Catalogue). |
| **📤 Async Dispatcher** | Composant de sortie asynchrone (Queue / Broker). | Envoie le panier validé vers la plateforme Analytics *A Propos Des Biens* sans faire attendre l'utilisateur en caisse. |

### 🗣️ Ce qu'on dit au jury sur le C3 (en 30 secondes) :
> *"Le niveau 3 zoome sur l'API et met en valeur notre architecture en couches propre : les requêtes arrivent sur le **Controller** qui valide les données et vérifie l'authentification. Il délègue au **Service** qui contient la logique métier (calculs et réconciliation sans conflit CRDT). Ce service manipule notre **Domain Model** (`Cart`, `User`, `Product`) et fait appel au **Repository** pour exécuter les requêtes SQL dans PostgreSQL. Enfin, lors de la validation du panier, le **Dispatcher Asynchrone** transmet les données d'achat vers Analytics en arrière-plan."*

---

## 💻 4. Niveau 4 : Code (C4) — Version Débutant Simple

> [!TIP] **L'idée clé pour le jury**
> Le **C4 (Niveau Code)** zoome sur le code d'une action très concrète : **l'ajout d'un article dans le panier**.
> On retrouve simplement les **4 briques** vues en cours d'informatique :
> 1. **`CartController`** : Reçoit l'appel du smartphone (endpoint REST).
> 2. **`CartService`** : Contient la logique métier (calcul du budget).
> 3. **`Cart` & `CartItem`** : Représentent le panier et les articles.
> 4. **`CartRepository`** : Sauvegarde dans la base de données PostgreSQL.

```mermaid
classDiagram
    direction TB

    %% 1. LE CONTRÔLEUR (Point d'entrée de l'API)
    class CartController {
        +ajouterArticle(cartId, gtin, quantite)
        +validerPanier(cartId)
    }

    %% 2. LE SERVICE (La logique métier et les règles)
    class CartService {
        +ajouterArticle(cartId, gtin, quantite)
        +verifierBudget(cartId)
        +validerPanier(cartId)
    }

    %% 3. LE DOMAINE (Les objets de données)
    class Cart {
        +Long id
        +Double budgetMax
        +List~CartItem~ articles
        +calculerTotal() Double
        +estDansLeBudget() Boolean
    }

    class CartItem {
        +String gtin
        +String nom
        +Double prix
        +int quantite
    }

    %% 4. LE REPOSITORY (Accès à PostgreSQL)
    class CartRepository {
        +trouverParId(id) Cart
        +sauvegarder(Cart cart)
    }

    %% RELATIONS DIRECTES ET LIMPIDES
    CartController --> CartService : 1. Reçoit l'appel et délègue
    CartService --> CartRepository : 2. Charge et sauvegarde (SQL)
    CartService ..> Cart : 3. Modifie le panier
    Cart "1" *-- "0..*" CartItem : 4. Contient des articles
```

---

### Le Code Java Minimal (Très facile à lire et expliquer) :

```java
// 1. LE CONTROLLER : Reçoit la requête HTTP envoyée par l'application mobile
@RestController
@RequestMapping("/api/panier")
public class CartController {

    @Autowired
    private CartService cartService;

    @PostMapping("/{cartId}/articles")
    public Cart ajouterArticle(@PathVariable Long cartId, 
                              @RequestParam String gtin, 
                              @RequestParam int quantite) {
        return cartService.ajouterArticle(cartId, gtin, quantite);
    }
}

// 2. LE SERVICE : Logique métier (vérification, calcul et sauvegarde)
@Service
public class CartService {

    @Autowired
    private CartRepository cartRepository;

    public Cart ajouterArticle(Long cartId, String gtin, int quantite) {
        // 1. On récupère le panier depuis la base de données PostgreSQL
        Cart cart = cartRepository.trouverParId(cartId);

        // 2. On ajoute l'article ou on incrémente la quantité
        cart.ajouterArticle(gtin, quantite);

        // 3. On sauvegarde le panier mis à jour
        return cartRepository.sauvegarder(cart);
    }
}
```

---

### 🗣️ Ce qu'on dit au jury sur le C4 (Script en 20 secondes) :
> *"Pour notre C4, nous avons zoomé sur le code d'une action essentielle : l'ajout d'un article dans le panier.*
> *1. Le **CartController** reçoit la requête envoyée par le smartphone lors d'un scan.*
> *2. Il appelle le **CartService** qui contient la logique métier et calcule le budget.*
> *3. Ce service manipule notre objet **Cart** qui regroupe les **CartItem**.*
> *4. Enfin, le **CartRepository** sauvegarde le tout dans la base de données PostgreSQL."*

---

## 🔄 5. Diagrammes de Séquence (Dynamique du Système)

Les diagrammes de séquence complètent la vue statique (C1 à C4) en illustrant **l'ordre chronologique des échanges** lors des deux scénarios clés de Kestachet.

---

### 5.1 Séquence 1 : Le Scan en rayon (Mode Hors-Ligne / Offline-First)

Ce schéma montre ce qui se passe quand l'utilisateur scanne un article dans un supermarché **sans aucune connexion réseau** :
- Tout se déroule localement sur le téléphone en **moins de 16 ms**.
- La jauge budgétaire se met à jour immédiatement sur l'écran.
- L'action est mémorisée dans un journal local (Outbox) pour synchronisation future.

```mermaid
sequenceDiagram
    autonumber
    actor U as 👤 Utilisateur
    participant APP as 📱 Application Mobile
    participant DB as 💾 Base Locale (SQLite)
    participant MOTEUR as ⚙️ Calculateur Budget

    Note over U,MOTEUR: Scénario en rayon sans réseau (Zone blanche / Sous-sol)
    
    U->>APP: Vise le code-barres avec la caméra
    APP->>APP: Décode le GTIN en direct (Google ML Kit)
    APP->>DB: Recherche le produit dans le cache local
    DB-->>APP: Produit trouvé (Nom, Prix : 2.50 €)
    
    APP->>MOTEUR: Demande le recalcul du budget
    MOTEUR->>MOTEUR: Additionne le prix et vérifie le budget max
    MOTEUR-->>APP: Nouveau total (ex: 42.50 €) + Jauge ORANGE
    
    APP->>DB: Enregistre le scan & empile dans le journal local (Outbox)
    DB-->>APP: Confirmation d'écriture locale (10 ms)
    
    APP-->>U: Affiche l'article ajouté & met à jour la jauge de budget
```

> [!TIP] **Ce qu'on dit au jury sur la Séquence 1** :
> *"Ce diagramme prouve notre fonctionnement **Offline-First** : chaque étape (scan caméra, recherche de l'article, calcul du budget et enregistrement SQLite) s'exécute à 100% sur le téléphone sans faire appel à internet. L'utilisateur n'a aucun ralentissement."*

---

### 5.2 Séquence 2 : Reconnexion 4G, Synchronisation et Envoi Analytics

Ce schéma montre ce qui se passe quand l'utilisateur sort du magasin et que la connexion est rétablie :
- L'application envoie son journal de modifications en attente.
- L'API fusionne les articles sans conflit (CRDT : $1 + 1 = 2$).
- Lors de la validation finale, l'utilisateur est libéré immédiatement en 50 ms pendant que le panier est transmis à la plateforme Analytics en arrière-plan.

```mermaid
sequenceDiagram
    autonumber
    participant APP as 📱 Application Mobile
    participant API as ⚡ API Backend (Spring Boot)
    participant BDD as 🗄️ BDD Centrale (PostgreSQL)
    participant DATA as 📊 Plateforme Analytics

    Note over APP,DATA: Étape 1 : Synchronisation dès le retour de la 4G
    APP->>APP: Détecte le retour du réseau (4G / Wi-Fi)
    APP->>API: Envoie les scans en attente (Deltas de synchronisation)
    
    API->>BDD: Charge le panier officiel du foyer
    BDD-->>API: Données actuelles du panier
    
    API->>API: Fusionne les scans sans conflit (CRDT : 1 + 1 = 2)
    API->>BDD: Sauvegarde le panier mis à jour
    BDD-->>API: Confirmation SQL
    API-->>APP: Synchronisation réussie (Panier à jour)

    Note over APP,DATA: Étape 2 : Validation des courses en caisse
    APP->>API: "Terminer mes courses" (Validation finale)
    API->>BDD: Marque le panier comme "VALIDÉ"
    API-->>APP: 200 OK (Réponse immédiate en 50 ms pour libérer le client)
    
    API-)DATA: Transmet le panier validé en tâche de fond (Asynchrone)
```

> [!TIP] **Ce qu'on dit au jury sur la Séquence 2** :
> *"Ce second diagramme illustre notre architecture distribuée résiliente : l'API réconcilie automatiquement les ajouts du foyer sans écrasement, et la validation du panier utilise un découplage asynchrone pour ne jamais faire attendre le client devant la caisse."*
