# whyml-lsp

Un serveur de langage (Language Server Protocol) pour WhyML, implémenté en OCaml.

## Description

Ce projet a pour objectif de fournir une implémentation complète d'un **Language Server Protocol (LSP)** pour le langage **WhyML**, en utilisant **OCaml** comme langage de développement principal. Le serveur permettra aux éditeurs de code et IDE de fournir des fonctionnalités avancées telles que :

- **Analyse syntaxique** et validation de code WhyML
- **Autocomplétion** intelligente
- **Navigation dans le code** (définitions, références, etc.)
- **Diagnostics** (détection d'erreurs, avertissements)
- **Documentation contextuelle**
- **Refactoring** et transformations de code

## Prérequis

- [OCaml](https://ocaml.org/) (version 4.14 ou supérieure recommandée)
- [Dune](https://dune.build/) (pour la gestion du build)
- [opam](https://opam.ocaml.org/) (pour la gestion des dépendances)
- Un éditeur compatible LSP (VS Code, Vim/Neovim, Emacs, etc.)

## Installation

### 1. Cloner le dépôt

```bash
git clone https://github.com/mlepadec/whyml-lsp.git
cd whyml-lsp
```

### 2. Installer les dépendances

```bash
opam install . --deps-only
```

### 3. Construire le projet

```bash
opam install .
# ou avec dune directement
dune build
```

## Utilisation

### Lancement du serveur

```bash
dune exec src/whyml_lsp.exe
```

Le serveur écoutera par défaut sur `stdin/stdout` pour communiquer avec le client LSP.

### Configuration avec un client LSP

#### VS Code

1. Installer l'extension [LSP Client](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-lsp)
2. Ajouter la configuration suivante dans `settings.json` :

```json
{
  "lsp.whyml-lsp.command": ["dune", "exec", "src/whyml_lsp.exe"],
  "lsp.whyml-lsp.filetypes": ["whyml"],
  "lsp.whyml-lsp.syntaxes": ["whyml"]
}
```

#### Vim/Neovim

Utiliser [coc.nvim](https://github.com/neoclide/coc.nvim) ou [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) avec une configuration similaire.

## Structure du projet

```
whyml-lsp/
├── src/
│   ├── server/           # Implémentation du serveur LSP
│   ├── parser/           # Analyseur syntaxique WhyML
│   ├── semantic/         # Analyse sémantique
│   ├── lsp/              # Types et protocole LSP
│   └── whyml_lsp.ml      # Point d'entrée principal
├── test/
│   └── tests/            # Tests unitaires et d'intégration
├── docs/
│   ├── ARCHITECTURE.md   # Documentation d'architecture
│   └── MEMOIRE_TECHNIQUE.md # Mémoire des choix techniques
├── dune-project
├── dune
└── README.md
```

## Développement

### Exécuter les tests

```bash
dune test
```

### Générer la documentation

```bash
dune doc
```

## Contribution

Les contributions sont les bienvenues ! Veuillez ouvrir une *Pull Request* pour toute amélioration ou correction.

## Licence

Ce projet est sous licence [MIT](LICENSE).

## Ressources

- [Language Server Protocol (LSP)](https://microsoft.github.io/language-server-protocol/)
- [WhyML](https://why3.lri.fr/) - Le langage cible
- [OCaml](https://ocaml.org/) - Langage d'implémentation
- [lsp-server](https://github.com/ocaml/lsp-server) - Référence pour l'implémentation LSP en OCaml
