# Architecture du projet whyml-lsp

Ce document décrit l'architecture globale du serveur LSP pour WhyML.

## Vue d'ensemble

Le projet suit une architecture modulaire en couches, conçue pour séparer clairement les responsabilités et faciliter la maintenance et l'extension.

```
┌─────────────────────────────────────────────────────────────┐
│                        Client LSP                              │
│  (VS Code, Vim, Emacs, etc.)                                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     JSON-RPC Layer                             │
│  - Gestion de la communication stdin/stdout                   │
│  - Sérialisation/désérialisation des messages                 │
│  - Dispatching des requêtes vers les handlers appropriés       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      LSP Server Core                           │
│  - Initialisation et cycle de vie du serveur                   │
│  - Gestion des documents (TextDocumentManager)                 │
│  - Gestion des workspaces                                      │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   Parser Layer   │ │ Semantic Layer   │ │  Services Layer   │
│                 │ │                  │ │                  │
│ - Lexing        │ │ - Symbol Table   │ │ - Completion     │
│ - Parsing       │ │ - Type Checking  │ │ - Hover          │
│ - AST Generation│ │ - Scope Analysis │ │ - Definition     │
│ - Error Recovery│ │ - Dependency Graph│ │ - References     │
└─────────────────┘ └─────────────────┘ │ - Diagnostics    │
                                          │ - Document Symbols│
                                          │ - Formatting     │
                                          └─────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     WhyML Language Support                     │
│  - Définitions des types AST WhyML                            │
│  - Grammaire du langage                                        │
│  - Spécificités sémantiques WhyML                             │
└─────────────────────────────────────────────────────────────┘
```

## Modules principaux

### 1. Module `Lsp` - Infrastructure LSP

**Fichier** : `src/lsp/`

Ce module contient les types et fonctions de base pour le protocole LSP :

- `lsp_types.ml` : Définitions des types LSP (Request, Response, Notification, etc.)
- `lsp_protocol.ml` : Implémentation du protocole JSON-RPC
- `lsp_server.ml` : Serveur principal et boucle d'événements

**Responsabilités** :
- Communication avec le client via stdin/stdout
- Sérialisation et désérialisation des messages JSON-RPC
- Routage des requêtes vers les handlers appropriés
- Gestion des capacités du serveur et du client

### 2. Module `Server` - Logique métier du serveur

**Fichier** : `src/server/`

Contient la logique principale du serveur :

- `server.ml` : Point d'entrée et initialisation
- `document_manager.ml` : Gestion des documents ouverts
- `workspace.ml` : Gestion des workspaces et fichiers
- `state.ml` : État global du serveur

**Responsabilités** :
- Maintenir l'état des documents ouverts
- Coordonner les différentes couches (parser, semantic, services)
- Gérer le cycle de vie des documents

### 3. Module `Parser` - Analyse syntaxique

**Fichier** : `src/parser/`

Implémente l'analyse syntaxique du code WhyML :

- `whyml_lexer.mll` : Analyseur lexical (généré par ocamllex)
- `whyml_parser.mly` : Analyseur syntaxique (généré par menhir)
- `whyml_ast.ml` : Définition de l'AST (Abstract Syntax Tree)
- `whyml_parser.ml` : Module principal du parser
- `error_recovery.ml` : Récupération d'erreurs et parsing incrémental

**Responsabilités** :
- Transformer le code source en AST
- Détecter les erreurs de syntaxe
- Fournir des informations de position pour les diagnostics
- Supporter le parsing incrémental pour les mises à jour de documents

### 4. Module `Semantic` - Analyse sémantique

**Fichier** : `src/semantic/`

Effectue l'analyse sémantique sur l'AST :

- `symbol_table.ml` : Table des symboles et résolution de noms
- `type_checker.ml` : Vérification des types
- `scope_analyzer.ml` : Analyse des portées (scopes)
- `dependency_graph.ml` : Graphe de dépendances entre modules
- `semantic_analyzer.ml` : Module principal d'analyse sémantique

**Responsabilités** :
- Résoudre les références de symboles
- Vérifier la cohérence des types
- Détecter les erreurs sémantiques
- Construire le graphe de dépendances

### 5. Module `Services` - Services LSP

**Fichier** : `src/services/`

Implémente les différents services LSP :

- `completion.ml` : Autocomplétion
- `hover.ml` : Information au survol
- `definition.ml` : Aller à la définition
- `references.ml` : Trouver les références
- `diagnostics.ml` : Génération de diagnostics
- `document_symbols.ml` : Symboles du document
- `formatting.ml` : Formatage de code
- `code_actions.ml` : Actions de code (refactoring)

**Responsabilités** :
- Fournir les fonctionnalités LSP spécifiques
- Utiliser les informations des couches parser et semantic
- Retourner des résultats conformes au protocole LSP

### 6. Module `Whyml` - Support du langage WhyML

**Fichier** : `src/whyml/`

Contient les définitions spécifiques à WhyML :

- `whyml_ast.ml` : Types AST spécifiques à WhyML
- `whyml_types.ml` : Types du langage WhyML
- `whyml_builtins.ml` : Définitions intégrées (builtins)

## Flux de traitement typique

### 1. Ouverture d'un document

```
Client -> Server: textDocument/didOpen
    │
    ▼
Server: Enregistre le document dans DocumentManager
    │
    ▼
Parser: Analyse syntaxique du contenu
    │
    ▼
Semantic: Analyse sémantique de l'AST
    │
    ▼
Diagnostics: Génère les diagnostics (erreurs, avertissements)
    │
    ▼
Server: Envoie textDocument/publishDiagnostics au client
```

### 2. Requête de complétion

```
Client -> Server: textDocument/completion
    │
    ▼
Server: Récupère le document et la position
    │
    ▼
Parser: Vérifie que l'AST est à jour
    │
    ▼
Semantic: Récupère le contexte sémantique à la position
    │
    ▼
Completion Service: Génère les suggestions de complétion
    │
    ▼
Server: Retourne les résultats au client
```

### 3. Mise à jour d'un document

```
Client -> Server: textDocument/didChange
    │
    ▼
Server: Met à jour le contenu du document
    │
    ▼
Parser: Parse incrémentalement les changements
    │
    ▼
Semantic: Met à jour l'analyse sémantique
    │
    ▼
Diagnostics: Recalcule les diagnostics pour les zones affectées
    │
    ▼
Server: Envoie textDocument/publishDiagnostics au client
```

## Décisions architecturales clés

1. **Séparation des couches** : Chaque couche (LSP, Server, Parser, Semantic, Services) a une responsabilité claire et bien définie.

2. **Parsing incrémental** : Pour améliorer les performances, le parser supporte les mises à jour incrémentales des documents.

3. **Cache sémantique** : Les résultats de l'analyse sémantique sont mis en cache pour éviter de recalculer inutilement.

4. **Approche déclarative** : Les définitions des types LSP et WhyML utilisent un style déclaratif pour faciliter la maintenance.

5. **Gestion d'erreur robuste** : Chaque couche gère ses propres erreurs et fournit des informations de diagnostic utiles.

## Intégration avec les outils existants

Le projet peut s'intégrer avec :

- **Why3** : Pour la vérification formelle des programmes WhyML
- **Alt-Ergo** : Pour la résolution de contraintes
- **Coq** : Pour les preuves formelles avancées

## Évolution future

L'architecture a été conçue pour permettre :

- L'ajout de nouvelles fonctionnalités LSP sans modifier le cœur
- Le support de nouvelles versions de WhyML
- L'intégration avec d'autres outils de vérification formelle
- L'extension à d'autres langages de la famille Why
