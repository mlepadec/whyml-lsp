# Architecture du projet whyml-lsp

Ce document a pour vocation de définir l'architecture du serveur LSP pour WhyML.

## Questions ouvertes sur l'architecture

### Structure globale

- **Quelle architecture globale adopter ?**
  - Architecture en couches (LSP → Server → Parser → Semantic → Services) ?
  - Architecture modulaire avec des interfaces claires entre composants ?
  - Architecture événementielle (event-driven) pour gérer les mises à jour de documents ?

- **Comment organiser les modules principaux ?**
  - Faut-il séparer clairement la gestion du protocole LSP de la logique métier ?
  - Comment structurer les dépendances entre modules (ex: Parser → Semantic → Services) ?

---

### Communication LSP

- **Comment implémenter la couche de communication JSON-RPC ?**
  - Utiliser une bibliothèque existante (ex: [lsp-server](https://github.com/ocaml/lsp-server)) ?
  - Implémenter manuellement pour un contrôle total ?

- **Quel mécanisme de sérialisation/désérialisation utiliser ?**
  - Utiliser une bibliothèque JSON existante (ex: [yojson](https://github.com/ocaml-community/yojson)) ?
  - Implémenter un parseur JSON personnalisé ?

---

### Gestion des documents

- **Comment gérer l'état des documents ouverts ?**
  - Maintenir une table des documents avec leur contenu et leur AST ?
  - Comment gérer les mises à jour incrémentales des documents ?

- **Comment gérer les workspaces (dossiers) ?**
  - Faut-il supporter les workspaces multi-racines ?
  - Comment détecter et charger les fichiers pertinents dans un workspace ?

---

### Parsing

- **Comment implémenter le parser WhyML ?**
  - Utiliser Menhir pour le parsing LR(1) ?
  - Utiliser ocamlyacc pour une solution plus simple ?
  - Implémenter un parser manuel pour plus de contrôle ?

- **Comment gérer le lexing ?**
  - Utiliser ocamllex ?
  - Utiliser une alternative comme Sedlex ou Ulex ?

- **Faut-il supporter le parsing incrémental ?**
  - Si oui, comment identifier les zones à re-parser après une modification ?
  - Comment fusionner les résultats du parsing incrémental avec l'AST existant ?

---

### Analyse sémantique

- **Comment structurer l'analyse sémantique ?**
  - Table des symboles hiérarchique pour gérer les portées imbriquées ?
  - Table des symboles plate pour simplifier l'implémentation ?

- **Comment gérer la vérification des types ?**
  - Implémenter une vérification de types explicite (WhyML a des annotations de types) ?
  - Ajouter de l'inférence de types pour plus de flexibilité ?

- **Comment gérer les dépendances entre modules ?**
  - Construire un graphe de dépendances pour l'ordre de compilation ?
  - Comment gérer les modifications en cascade ?

---

### Services LSP

- **Quels services LSP implémenter en priorité ?**
  - Complétion (textDocument/completion) ?
  - Hover (textDocument/hover) ?
  - Définition (textDocument/definition) ?
  - Références (textDocument/references) ?
  - Diagnostics (textDocument/publishDiagnostics) ?
  - Symboles du document (textDocument/documentSymbol) ?
  - Formatage (textDocument/formatting) ?

- **Comment organiser les services ?**
  - Un module par service ?
  - Un module unique pour tous les services ?

---

### Intégration avec WhyML

- **Comment représenter l'AST WhyML ?**
  - Définir des types OCaml pour chaque constructeur WhyML ?
  - Utiliser une représentation générique (ex: avec des variants) ?

- **Comment gérer les spécificités de WhyML ?**
  - Comment représenter les annotations de types ?
  - Comment gérer les preuves et les contrats ?
  - Comment intégrer avec Why3 pour la vérification formelle ?

---

### Performances

- **Comment optimiser les performances ?**
  - Utiliser de la mise en cache (AST, informations sémantiques, diagnostics) ?
  - Implémenter du traitement par lots (batching) pour les modifications rapides ?
  - Utiliser de l'évaluation paresseuse (Lazy.t) pour les calculs coûteux ?

- **Comment gérer la concurrence ?**
  - Utiliser Lwt pour les opérations asynchrones ?
  - Utiliser Async ?
  - Gérer manuellement les threads ?
