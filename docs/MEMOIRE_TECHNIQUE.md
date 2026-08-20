# Mémoire Technique - whyml-lsp

Ce document a pour vocation de tracer les **choix techniques** effectués durant le développement du serveur LSP pour WhyML, ainsi que les **raisons** qui ont motivé ces choix.

Pour l'instant, seul le langage d'implémentation (OCaml) a été décidé. Les autres choix techniques restent à déterminer.

## Questions ouvertes sur les choix techniques

### Choix du langage d'implémentation

✅ **Décision : OCaml** (déjà validé)

---

### Système de build

- **Quel système de build utiliser ?**
  - Dune (standard moderne pour OCaml) ?
  - Make (plus traditionnel) ?
  - Oasis ou Om (alternatives) ?

- **Quels avantages rechercher ?**
  - Simplicité de configuration ?
  - Intégration avec opam ?
  - Support des tests et de la documentation ?

---

### Gestion des dépendances

- **Comment gérer les dépendances externes ?**
  - Utiliser opam pour toutes les dépendances ?
  - Intégrer certaines dépendances directement dans le projet ?

- **Quelles dépendances potentielles identifier ?**
  - Bibliothèque JSON (ex: yojson) ?
  - Bibliothèque LSP (ex: lsp-server) ?
  - Bibliothèque de parsing (ex: Menhir, ocamllex) ?
  - Bibliothèque de concurrence (ex: Lwt, Async) ?

---

### Implémentation du protocole LSP

- **Comment implémenter le protocole JSON-RPC ?**
  - Utiliser une bibliothèque existante (ex: [lsp-server](https://github.com/ocaml/lsp-server)) ?
    - Avantages : Moins de code à maintenir, conformité garantie
    - Inconvénients : Moins de contrôle, dépendance externe
  - Implémenter manuellement ?
    - Avantages : Contrôle total, optimisation possible
    - Inconvénients : Plus de code, risque d'erreurs

- **Comment gérer la communication ?**
  - stdin/stdout (standard LSP) ?
  - Socket TCP (pour un serveur distant) ?
  - Les deux ?

---

### Parsing

- **Quel générateur de parseur utiliser ?**
  - Menhir (LR(1), récupération d'erreurs avancée) ?
  - ocamlyacc (LR(0), plus simple) ?
  - Parser manuel (contrôle total) ?
  - Parser combinators (ex: Parsexp) ?

- **Quel générateur de lexer utiliser ?**
  - ocamllex (standard) ?
  - Sedlex (meilleure gestion des Unicode) ?
  - Ulex (alternative) ?
  - Lexer manuel ?

- **Comment gérer les erreurs de parsing ?**
  - Récupération d'erreurs (error recovery) pour continuer après une erreur ?
  - Arrêt à la première erreur ?
  - Comment rapporter les erreurs au client LSP ?

- **Faut-il supporter le parsing incrémental ?**
  - Si oui, comment implémenter cette fonctionnalité ?
  - Quels sont les compromis entre complexité et performance ?

---

### Analyse sémantique

- **Comment structurer la table des symboles ?**
  - Hiérarchique (pour les portées imbriquées) ?
  - Plate (plus simple) ?

- **Comment gérer la vérification des types ?**
  - Vérification explicite uniquement (WhyML a des annotations de types) ?
  - Ajouter de l'inférence de types ?

- **Comment gérer les dépendances entre modules ?**
  - Construire un graphe de dépendances ?
  - Comment gérer les modifications en cascade ?

---

### Services LSP

- **Quels services implémenter en priorité ?**
  - Quels sont les services les plus utiles pour les utilisateurs de WhyML ?

- **Comment organiser l'implémentation des services ?**
  - Un module par service ?
  - Un module unique pour tous les services ?

---

### Gestion des performances

- **Comment optimiser les performances ?**
  - Mise en cache des résultats (AST, informations sémantiques) ?
  - Traitement par lots (batching) pour les modifications rapides ?
  - Évaluation paresseuse (Lazy.t) ?

- **Comment gérer la concurrence ?**
  - Lwt ?
  - Async ?
  - Threads natifs ?
  - Pas de concurrence (traitement séquentiel) ?

---

### Intégration continue

- **Quel outil de CI utiliser ?**
  - GitHub Actions (intégration native avec GitHub) ?
  - GitLab CI ?
  - Travis CI ?
  - CircleCI ?

- **Quels workflows configurer ?**
  - Build et test sur chaque push ?
  - Vérification du formatage du code ?
  - Génération de la documentation ?

---

### Intégration avec Why3

- **Comment intégrer avec Why3 pour la vérification formelle ?**
  - Appeler Why3 comme un processus externe ?
  - Intégrer Why3 comme une bibliothèque ?
  - Quelle est la meilleure approche pour récupérer les résultats de vérification ?

---

## Historique des révisions

| Date | Auteur | Modification |
|------|--------|--------------|
| [À remplir] | [À remplir] | Choix du langage : OCaml |
