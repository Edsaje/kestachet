---
tags:
  - Architecture
  - Kestachet
  - FicheDebutant
  - Etudiant
  - Soutenance
  - Revision
date: 2026-09-08
course: 4.1 | Projet de synthèse — Kestachet
subject: Architecture Logicielle & Distribuée
status: Fiche Simple pour Étudiant Débutant
---

# 🛒 Kestachet — Le Guide Simple "Vu d'un Étudiant Débutant"

> [!NOTE] **C'est quoi ce document ?**
> C'est une fiche de révision **sans aucun jargon compliqué**, expliquée avec des analogies simples de la vie de tous les jours. Elle permet à n'importe quel étudiant débutant de comprendre le projet de A à Z et de réussir sa soutenance de 5 minutes devant le jury.

---

## 🎬 1. Le "Film" de l'application : que se passe-t-il pour de vrai ?

Imagine que tu es en train de faire tes courses au sous-sol d'un grand supermarché (zéro barre de 4G) :

```
             EN RAYON (Au sous-sol, 0 barre de réseau)
                          
   [Toi] ──► Tu scannes un paquet de pâtes (1,50 €)
               │
               ▼
   [Ton Téléphone] ──► 1. Écrit dans sa mémoire interne (SQLite) : "+1 paquet de pâtes"
                       2. Additionne ton panier : 14,50 € / 60 € max
                       3. Allume la jauge en VERT 🟢 (calculé en 10 ms sans Internet)
                       4. Met un mot dans sa boîte d'envoi : "À envoyer dès le retour de la 4G"

══════════════════════════════════════════════════════════════════════════════════
             À LA SORTIE DU MAGASIN (Sur le parking, retour de la 4G)

   [Ton Téléphone] ──► Envoie automatiquement les articles scannés au serveur
               │
               ▼
   [Le Serveur] ────► 1. Met à jour la liste officielle du foyer dans PostgreSQL
                      2. Fait biper le téléphone de ton coloc / ta copine resté(e) à la maison
                      3. Dépose le ticket dans une boîte d'attente pour que les 
                         analystes étudient les prix SANS faire ramer l'application
```

---

## 🎨 2. Le Schéma "Tableau Blanc" (Les 3 gros blocs du système)

```
 📱 1. LE TÉLÉPHONE                  ☁️ 2. LE SERVEUR                🏢 3. LA STARTUP
 (Dans le supermarché)               (Dans le Cloud)                 (A Propos Des Biens)

┌──────────────────────┐          ┌──────────────────────┐        ┌──────────────────────┐
│  • Caméra (Scan)     │          │  • Reçoit les scans  │        │  • Reçoit les paniers│
│  • Calcul du budget  │ ───────► │  • Met à jour la BDD │ ─────► │    en tâche de fond  │
│  • Base SQLite       │ (Quand y │  • Partage avec la   │ (Sans  │  • Cherche les infos │
│    (Marche sans 4G)  │ a du     │    famille           │ bloquer│    sur OpenFoodFacts │
└──────────────────────┘  réseau) └──────────────────────┘ l'app) └──────────────────────┘
```

---

## 🍝 3. L'erreur du débutant (Code Spaghetti) VS L'Architecture Hexagonale

Pour comprendre pourquoi les profs demandent une **Architecture Hexagonale**, regarde la différence entre ce qu'un débutant code et ce qu'une bonne architecture produit :

### ❌ Le piège du débutant : Le fichier géant "Fourre-tout"
Le débutant crée un seul fichier `MonEcranPanier.java` (ou `.dart` / `.jsx`) de 600 lignes :
```
[MonEcranPanier]
  ├── Allume la caméra du téléphone
  ├── Fait la requête SQL "INSERT INTO panier..." directement
  ├── Fait le calcul : "if (total > 60) bouton.setColor(ROUGE)"
  └── Fait l'appel réseau : "fetch('https://api.kestachet.com/sync')"
```
💥 **Le problème :** Si demain la caméra plante ou si la base de données change, tout l'écran est cassé et le calcul du budget ne marche plus !

### ✅ Ce qu'on a fait : L'Architecture Hexagonale (Les 3 tiroirs bien rangés)

```
┌────────────────────────────────────────────────────────┐
│ 1. LE TIROIR ÉCRAN (UI)                                │
│    Il s'occupe juste d'afficher du vert ou du rouge.   │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│ 2. LE CŒUR MÉTIER (Pur code sans aucune dépendance)    │
│    Il calcule : Total = Total + Prix.                  │
│    Il applique la règle : Vert < 80%, Orange, Rouge.   │
│    Il s'en fiche complètement de savoir comment la     │
│    caméra ou la base SQLite fonctionnent !             │
└───────────────────────────┬────────────────────────────┘
                            │
┌───────────────────────────▼────────────────────────────┐
│ 3. LES TIROIRS TECHNIQUES (Adaptateurs)                │
│    • Un adaptateur Caméra (lit le code-barres)         │
│    • Un adaptateur SQLite (écrit sur le stockage flash)│
└────────────────────────────────────────────────────────┘
```

> [!TIP] **La métaphore de la console Nintendo Switch (Pour le commercial)**
> Le **jeu** (le calcul du budget), c'est le cœur au milieu de la console. 
> La caméra, c'est juste une **manette branchée sur un port USB universel**. 
> Si demain Apple ou Google sortent un nouveau modèle de caméra, on a juste besoin de changer l'adaptateur de la manette. **On ne touche pas à une seule ligne du calcul de budget.** 
> Ça prend **2 jours** au lieu de 3 semaines de refonte risquée !

---

## 🥛 4. La synchronisation expliquée avec un carnet (CRDT)

Imagine qu'Alice et Bob font les courses ensemble dans le même supermarché. Ils sont tous les deux au sous-sol sans 4G :

* **La mauvaise méthode (Écrasement d'état) :**
  * Alice scanne 1 lait $
ightarrow$ Son téléphone enregistre : *"Quantité = 1"*.
  * Bob scanne 1 lait $
ightarrow$ Son téléphone enregistre : *"Quantité = 1"*.
  * À la sortie du magasin, le serveur reçoit les deux fichiers. Le fichier de Bob écrase celui d'Alice : **le panier affiche 1 bouteille au lieu de 2 ! (Catastrophe)**.

* **Notre bonne méthode (CRDT / Compteurs d'opérations) :**
  * Alice note : *"Ajouter +1 lait"*.
  * Bob note : *"Ajouter +1 lait"*.
  * À la sortie du magasin, le serveur applique les deux actions : $+1 + 1 = 2$ bouteilles ! **Aucune donnée n'est écrasée.**
  * Pour le prix (si Alice a mis 1,20 € et Bob 1,25 €), le serveur garde le prix le plus récemment tapé (*Last-Write-Wins*).

---

## 📦 5. Pourquoi une File d'Attente pour Open Food Facts ?

Imagine que tu es à la caisse d'un supermarché :

* **Sans file d'attente :** La caissière refuse de te laisser partir tant qu'elle n'a pas appelé un fournisseur en Suisse pour savoir qui fabrique tes yaourts. **Tu attends 5 minutes devant la caisse pour rien.**
* **Avec notre file d'attente (Message Queue) :** La caissière valide ton ticket en **2 secondes**, te donne ton reçu et te souhaite une bonne journée. Pendant que tu ranges tes courses dans le coffre, un employé en arrière-plan s'occupe tranquillement d'appeler le fournisseur pour enregistrer les yaourts.

C'est exactement ce que fait notre architecture :
1. L'utilisateur clique sur "Terminer mes courses" $
ightarrow$ le serveur lui répond `OK` en **50 millisecondes**.
2. Le panier part dans une file d'attente.
3. Un petit programme indépendant (**Worker**) interroge Open Food Facts en arrière-plan.
4. Si le site d'Open Food Facts tombe en panne le samedi après-midi, notre **Disjoncteur (Circuit Breaker)** coupe les appels et réessaie la nuit. **L'utilisateur n'est jamais bloqué.**

---

## ☁️ 6. Le Cloud expliqué simplement : Pourquoi le Serverless ?

Dans un supermarché, les gens font leurs courses :
* Le samedi après-midi : **10 000 personnes en même temps** !
* Le mardi à 3h du matin : **0 personne**.

Si on loue des serveurs classiques à l'année, on paie des centaines d'euros par mois pour des machines qui tournent dans le vide la nuit.

**Notre solution : Le Serverless (Payer à l'usage)**
* Le samedi à 16h : le Cloud démarre 50 instances pour absorber la foule.
* La nuit : tout s'éteint automatiquement. **La facture tombe à 0 €.**
* Et en choisissant des datacenters à **Paris**, on respecte à 100% le RGPD pour protéger les habitudes de consommation des familles.

---

## 🗣️ 7. Votre Soutenance de 5 Minutes (Script prêt à lire)

Voici le texte exact à dire devant le jury, minute par minute, avec des mots naturels :

- **[0:00 - 1:00] Intro & La Métaphore (Pour le Commercial)**
  > *"Bonjour messieurs. On sait tous pourquoi les applications de courses échouent : au sous-sol du supermarché, il n'y a pas de réseau et l'écran se fige. Avec Kestachet, notre promesse est simple : zéro frustration en magasin. Pour que vos futures évolutions ne coûtent pas une fortune, on a pensé l'application comme une console Switch : le moteur de budget est au centre, et la caméra est juste une manette interchangeable. C'est l'Architecture Hexagonale."*

- **[1:00 - 2:00] Le Cœur Mobile (Pour le CTO)**
  > *"Techniquement, Antonin, notre cœur métier est pur et indépendant de la caméra et de SQLite. Le calcul du budget s'exécute sur le téléphone en moins de 10 millisecondes. En utilisant les design patterns Strategy et Repository, on peut changer la caméra ou le système de stockage sans toucher à une seule ligne du calcul de budget."*

- **[2:00 - 3:00] Le Mode Hors-Ligne & Zéro Conflit (Pour tous)**
  > *"Quand l'utilisateur perd le réseau, tout est sauvegardé dans la base SQLite de son téléphone, et la jauge budgétaire passe au vert ou rouge en direct. Si deux personnes du foyer scannent du lait en même temps en zone blanche, le système enregistre des opérations relatives : à la sortie, le serveur fait 1 + 1 = 2 bouteilles grâce aux CRDTs, sans écraser de données."*

- **[3:00 - 4:00] Le Backend & Open Food Facts (Pour le CTO)**
  > *"Côté serveur, la validation libère l'utilisateur en 50 millisecondes. Le panier part dans une file d'attente. Si un produit est inconnu, notre worker va chercher les infos sur Open Food Facts en arrière-plan. Et si Open Food Facts plante un samedi, notre disjoncteur protège le serveur et rejoue les requêtes la nuit."*

- **[4:00 - 5:00] Le Cloud & Conclusion**
  > *"Pour l'hébergement, nous avons choisi du Serverless à Paris : zéro coût la nuit quand les magasins sont fermés, conformité RGPD totale, et tout est sous forme de conteneurs Docker pour ne pas être prisonnier d'un fournisseur. Kestachet est prête pour des millions d'utilisateurs. Merci pour votre attention, nous sommes à votre disposition pour vos questions !"*

---

## 📋 8. Les 8 Compétences validées pour le Professeur

| N° | Compétence | Ce que vous montrez pour avoir 20/20 |
| :---: | :--- | :--- |
| **1** | **Design Patterns** | Patron **Repository** (SQLite), **Strategy** (caméra), **Observer** (jauge de budget). |
| **2** | **Architecture logicielle** | L'**Architecture Hexagonale** séparant le panier du matériel. |
| **3** | **Défendre l'architecture** | **+** 100% testable hors-ligne / **-** Quelques interfaces de plus à écrire au début. |
| **4** | **Architecture distribuée** | Le schéma **C4 Niveau 2 (Container)** avec App, API, BDD, Queues et Workers. |
| **5** | **Défendre la distribuée** | **+** L'utilisateur n'attend jamais en caisse / **-** Petit délai de synchro à la sortie. |
| **6** | **Choisir le cloud** | 3 critères : 1. Coût (Serverless), 2. Données en France (RGPD), 3. Services gérés. |
| **7** | **Défendre le cloud** | **+** Facture à 0€ la nuit / **-** Risque de dépendance (évité grâce à Docker). |
| **8** | **Mémo des concepts clés** | Définitions claires d'Hexagonale, Offline-First, CRDT, C4, Serverless, Message Queue. |
