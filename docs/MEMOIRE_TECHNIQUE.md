# Mémoire Technique - whyml-lsp

Ce document trace l'historique des **choix techniques** effectués durant le développement du serveur LSP pour WhyML, ainsi que les **raisons** qui ont motivé ces choix.

## Table des matières

1. [Choix du langage d'implémentation](#choix-du-langage-dimplémentation)
2. [Choix des bibliothèques](#choix-des-bibliothèques)
3. [Architecture logicielle](#architecture-logicielle)
4. [Implémentation du protocole LSP](#implémentation-du-protocole-lsp)
5. [Gestion du parsing](#gestion-du-parsing)
6. [Analyse sémantique](#analyse-sémantique)
7. [Gestion des performances](#gestion-des-performances)
8. [Intégration continue](#intégration-continue)

---

## 1. Choix du langage d'implémentation

### Décision : Utiliser OCaml

**Date** : Début du projet

**Alternatives envisagées** :
- Rust
- TypeScript
- Haskell
- Python

**Raisons du choix** :

1. **Affinités avec le domaine** : WhyML est un langage formel utilisé dans la vérification de programmes. OCaml, en tant que langage fonctionnel avec un système de types puissant, est particulièrement adapté pour manipuler des structures de données complexes comme les AST et effectuer des analyses statiques.

2. **Écosystème existant** : Il existe déjà des implémentations LSP en OCaml (comme [ocaml-lsp-server](https://github.com/ocaml/ocaml-lsp-server)) qui peuvent servir de référence et fournir des bibliothèques réutilisables.

3. **Sécurité et robustesse** : Le système de types statique d'OCaml permet de détecter de nombreuses erreurs à la compilation, réduisant ainsi les bugs en production. La gestion mémoire automatique (GC) évite les problèmes de fuites mémoire.

4. **Performance** : OCaml compile vers du code natif efficace, ce qui est important pour un serveur qui doit traiter rapidement de gros fichiers.

5. **Expressivité** : Les types algébriques (variants, records) d'OCaml permettent de modéliser naturellement les AST et les structures de données du LSP.

**Compromis acceptés** :
- Courbe d'apprentissage plus raide pour les contributeurs ne connaissant pas OCaml
- Écosystème moins riche que celui de TypeScript ou Python pour certaines fonctionnalités

---

## 2. Choix des bibliothèques

### Décision : Utiliser Dune comme système de build

**Date** : Début du projet

**Alternatives envisagées** :
- Make
- Oasis
- Om

**Raisons du choix** :

1. **Standard de la communauté** : Dune est devenu le système de build standard pour les projets OCaml modernes.

2. **Fonctionnalités avancées** : Dune supporte nativement :
   - La gestion des dépendances
   - La compilation séparée
   - Les tests unitaires
   - La génération de documentation
   - Les workspaces multi-projets

3. **Intégration avec opam** : Dune s'intègre parfaitement avec opam pour la gestion des dépendances externes.

4. **Simplicité** : Fichiers de configuration clairs et concis.

---

### Décision : Utiliser Menhir pour le parsing

**Date** : Début du projet

**Alternatives envisagées** :
- ocamlyacc
- Parsing manuel
- Parser combinators (ex: Parsexp)

**Raisons du choix** :

1. **Puissance** : Menhir est un générateur de parseurs LR(1) avec des fonctionnalités avancées comme la détection et la récupération d'erreurs.

2. **Qualité des messages d'erreur** : Menhir génère des messages d'erreur de parsing plus clairs et exploitables que ocamlyacc.

3. **Modularité** : Menhir permet une meilleure séparation entre le lexer et le parser.

4. **Maintenance active** : Menhir est activement maintenu par François Pottier.

**Compromis acceptés** :
- Courbe d'apprentissage pour la syntaxe des fichiers .mly
- Temps de compilation légèrement plus long

---

### Décision : Utiliser ocamllex pour le lexing

**Date** : Début du projet

**Alternatives envisagées** :
- Lexer manuel
- Sedlex
- Ulex

**Raisons du choix** :

1. **Standard** : ocamllex est l'outil standard pour la génération de lexers en OCaml.

2. **Performance** : Les lexers générés par ocamllex sont très performants.

3. **Intégration avec Menhir** : ocamllex s'intègre naturellement avec Menhir.

---

## 3. Architecture logicielle

### Décision : Architecture en couches

**Date** : Début du projet

**Alternatives envisagées** :
- Architecture monolithique
- Architecture micro-services
- Architecture basée sur des plugins

**Raisons du choix** :

1. **Séparation des responsabilités** : Chaque couche a une responsabilité claire, ce qui facilite la compréhension et la maintenance du code.

2. **Testabilité** : Les couches peuvent être testées indépendamment les unes des autres.

3. **Évolutivité** : Il est facile d'ajouter de nouvelles fonctionnalités ou de modifier une couche sans affecter les autres.

4. **Réutilisabilité** : Certaines couches (comme le parser) peuvent être réutilisées dans d'autres contextes.

**Couches définies** :
- LSP (communication)
- Server (coordination)
- Parser (analyse syntaxique)
- Semantic (analyse sémantique)
- Services (fonctionnalités LSP)

---

### Décision : Parsing incrémental

**Date** : Conception initiale

**Alternatives envisagées** :
- Re-parsing complet à chaque changement
- Parsing différentiel

**Raisons du choix** :

1. **Performance** : Re-parser l'intégralité d'un gros fichier à chaque modification serait trop coûteux.

2. **Expérience utilisateur** : Les éditeurs s'attendent à des retours rapides (diagnostics, complétion) même pour de gros fichiers.

3. **Complexité gérable** : Le parsing incrémental est complexe à implémenter, mais les bénéfices en termes de performance justifient l'effort.

**Implémentation prévue** :
- Utiliser la position des modifications pour identifier les zones à re-parser
- Maintenir un cache des AST partiels
- Fusionner les résultats du parsing incrémental avec l'AST existant

---

### Décision : Cache sémantique

**Date** : Conception initiale

**Alternatives envisagées** :
- Recalcul complet à chaque changement
- Cache partiel

**Raisons du choix** :

1. **Performance** : L'analyse sémantique peut être coûteuse, surtout pour de gros projets.

2. **Consistance** : Le cache permet de maintenir une vue cohérente de l'état sémantique du projet.

3. **Invalidation intelligente** : Seules les parties affectées par une modification sont recalculées.

**Stratégie d'invalidation** :
- Invalider le cache pour un fichier lors de sa modification
- Invalider les dépendants dans le graphe de dépendances
- Recalculer de manière paresseuse (lazy) lorsque nécessaire

---

## 4. Implémentation du protocole LSP

### Décision : Implémentation manuelle de JSON-RPC

**Date** : Début du développement

**Alternatives envisagées** :
- Utiliser une bibliothèque JSON-RPC existante
- Utiliser lsp-server (bibliothèque OCaml pour LSP)

**Raisons du choix** :

1. **Contrôle total** : Une implémentation manuelle permet un contrôle fin sur la sérialisation, le dispatching, et la gestion des erreurs.

2. **Apprentissage** : Implémenter manuellement permet de mieux comprendre le protocole LSP.

3. **Flexibilité** : Possibilité d'optimiser pour les besoins spécifiques de WhyML.

**Compromis acceptés** :
- Plus de code à écrire et maintenir
- Risque d'erreurs dans l'implémentation du protocole

**Évolution possible** :
Si la maintenance devient trop lourde, migration vers [lsp-server](https://github.com/ocaml/lsp-server) pourrait être envisagée.

---

### Décision : Communication via stdin/stdout

**Date** : Début du développement

**Alternatives envisagées** :
- Socket TCP
- Socket Unix
- Processus séparé avec IPC

**Raisons du choix** :

1. **Standard LSP** : Le protocole LSP spécifie que la communication se fait via stdin/stdout par défaut.

2. **Simplicité** : Pas besoin de gérer des connexions réseau ou des sockets.

3. **Portabilité** : Fonctionne sur tous les systèmes d'exploitation sans modification.

4. **Intégration facile** : Les clients LSP s'attendent à ce mode de communication.

---

## 5. Gestion du parsing

### Décision : Utiliser un AST générique

**Date** : Conception du parser

**Alternatives envisagées** :
- AST spécifique à chaque constructeur
- AST minimal

**Raisons du choix** :

1. **Flexibilité** : Un AST générique peut représenter toutes les constructions de WhyML.

2. **Extensibilité** : Facile d'ajouter de nouvelles constructions sans casser l'existant.

3. **Partage de code** : Les outils d'analyse (sémantique, services) peuvent travailler sur un AST unique.

**Structure de l'AST** :
```ocaml
type expression =
  | EVar of string * position
  | EApp of expression * expression list * position
  | ELet of string * expression * expression * position
  | EIf of expression * expression * expression * position
  | ELiteral of literal * position
  | EBinaryOp of binary_op * expression * expression * position
  | EUnaryOp of unary_op * expression * position
  | ERecord of (string * expression) list * position
  | EFieldAccess of expression * string * position
  | EMatch of expression * (pattern * expression) list * position
  | EAnnot of expression * type_expr * position
```

---

### Décision : Gestion des erreurs de parsing

**Date** : Conception du parser

**Alternatives envisagées** :
- Arrêter le parsing à la première erreur
- Ignorer les parties invalides

**Raisons du choix** : **Récupération d'erreurs (Error Recovery)**

1. **Expérience utilisateur** : Même avec des erreurs de syntaxe, l'utilisateur doit pouvoir voir d'autres diagnostics.

2. **Diagnostics multiples** : Permet de rapporter plusieurs erreurs simultanément.

3. **Édition continue** : L'utilisateur peut continuer à éditer même avec des erreurs.

**Stratégie implémentée** :
- Utiliser les capacités de récupération d'erreurs de Menhir
- Insérer des nœuds « erreur » dans l'AST pour représenter les parties invalides
- Continuer le parsing après une erreur
- Collecter toutes les erreurs pour les rapporter comme diagnostics

---

## 6. Analyse sémantique

### Décision : Table des symboles hiérarchique

**Date** : Conception de l'analyse sémantique

**Alternatives envisagées** :
- Table des symboles plate
- Table des symboles basée sur les fichiers

**Raisons du choix** :

1. **Portées imbriquées** : WhyML supporte les portées imbriquées (let-in, modules, etc.).

2. **Résolution de noms** : Permet une résolution de noms efficace en tenant compte de la portée courante.

3. **Shadowing** : Gère correctement le shadowing de variables (redéfinition dans une portée interne).

**Implémentation** :
```ocaml
type scope = {
  symbols : (string, symbol_info) Hashtbl.t;
  parent : scope option;
}
```

---

### Décision : Vérification de types explicite

**Date** : Conception de l'analyse sémantique

**Alternatives envisagées** :
- Inférence de types complète
- Vérification de types partielle

**Raisons du choix** :

1. **Fidélité à WhyML** : WhyML a un système de types explicite avec des annotations de types.

2. **Précision** : La vérification de types explicite permet de détecter plus d'erreurs.

3. **Performance** : L'inférence de types complète peut être coûteuse.

**Compromis acceptés** :
- Moins flexible que l'inférence complète
- Peut nécessiter plus d'annotations de la part de l'utilisateur

---

## 7. Gestion des performances

### Décision : Évaluation paresseuse (Lazy Evaluation)

**Date** : Conception de l'architecture

**Alternatives envisagées** :
- Évaluation stricte
- Cache agressif

**Raisons du choix** :

1. **Optimisation des ressources** : Ne calculer que ce qui est nécessaire.

2. **Réactivité** : Permet de retourner rapidement des résultats partiels.

3. **Adaptation à l'utilisation** : Seules les parties du code effectivement utilisées sont analysées en profondeur.

**Implémentation en OCaml** :
Utilisation du type `Lazy.t` pour les calculs coûteux :
```ocaml
val semantic_info : (unit -> semantic_result) Lazy.t
```

---

### Décision : Mise en cache des résultats

**Date** : Conception de l'architecture

**Alternatives envisagées** :
- Pas de cache
- Cache externe (Redis, etc.)

**Raisons du choix** :

1. **Performance** : Éviter de recalculer des informations déjà calculées.

2. **Simplicité** : Un cache en mémoire est simple à implémenter et efficace.

3. **Cohérence** : Le cache est local au processus du serveur.

**Types de cache** :
- Cache des AST parsés
- Cache des informations sémantiques
- Cache des résultats de complétion
- Cache des diagnostics

---

### Décision : Traitement par lots (Batching)

**Date** : Optimisation des performances

**Alternatives envisagées** :
- Traitement immédiat de chaque requête
- File d'attente simple

**Raisons du choix** :

1. **Réduction de la latence** : Regrouper plusieurs modifications avant de déclencher un re-parsing.

2. **Optimisation des ressources** : Éviter de déclencher plusieurs analyses coûteuses pour des modifications rapprochées.

3. **Expérience utilisateur** : L'utilisateur perçoit une meilleure réactivité.

**Implémentation** :
- Bufferiser les modifications pendant un court délai (ex: 100ms)
- Déclencher le traitement lorsque le buffer est plein ou que le délai est écoulé
- Utiliser Lwt ou Async pour la gestion asynchrone

---

## 8. Intégration continue

### Décision : Utiliser GitHub Actions

**Date** : Mise en place de la CI

**Alternatives envisagées** :
- GitLab CI
- Travis CI
- CircleCI

**Raisons du choix** :

1. **Intégration native avec GitHub** : Pas besoin de configuration externe.

2. **Gratuité pour les projets open source** : GitHub Actions offre des minutes de calcul gratuites.

3. **Flexibilité** : Permet de définir des workflows complexes.

4. **Communauté** : Large adoption dans la communauté OCaml.

**Workflow prévu** :
- Build et test sur chaque push et pull request
- Vérification de la compilation
- Exécution des tests unitaires
- Vérification du formatage du code (avec ocamlformat)
- Génération de la documentation

---

## Historique des révisions

| Date | Auteur | Modification |
|------|--------|--------------|
| [Date de création] | [Auteur] | Création initiale du mémoire technique |

---

## Glossaire

- **AST** : Abstract Syntax Tree (Arbre de Syntaxe Abstraite)
- **LSP** : Language Server Protocol
- **JSON-RPC** : Protocole de communication utilisé par LSP
- **Menhir** : Générateur de parseurs LR(1) pour OCaml
- **ocamllex** : Générateur de lexers pour OCaml
- **Dune** : Système de build pour OCaml
- **opam** : Gestionnaire de paquets pour OCaml

---

## Références

- [Language Server Protocol Specification](https://microsoft.github.io/language-server-protocol/specifications/lsp/3.17/specification/)
- [WhyML Documentation](https://why3.lri.fr/doc/)
- [OCaml Documentation](https://ocaml.org/docs/)
- [Menhir Manual](https://gallium.inria.fr/~fpottier/menhir/menhir-manual.html)
- [Dune Manual](https://dune.readthedocs.io/)
