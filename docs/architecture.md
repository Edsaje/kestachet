# Architecture

```mermaid
flowchart TB
    U["👤 Utilisateur<br><i>(Scanne en rayon, gère le budget, com0plète la liste à distance)</i>"] <-- Scanne, modifie le panier, synchronise, consulte et met à jour en direct --> K@{ label: "📦 Système kestachet<br><b>[Système Logiciel Central]</b><br>Application mobile offline-first, calcul de budget,<br>gestion de liste partagée et passerelle d'ingestion" }
    K <-- Transmet les paniers validés (Asynchrone) --> DATA@{ label: "📊 Plateforme Analytics A Propos Des Biens<br><b>[Système Externe]</b><br>Traitement et valorisation des données d'achat" }
    OFF["🌐 Open Food Facts<br><b>[Système Externe]</b><br>Catalogue public des produits alimentaires"] -- Récupère les métadonnées de GTIN inconnus --> DATA

    K@{ shape: rect}
    DATA@{ shape: rect}
```
