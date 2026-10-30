# Changelog

Toutes les évolutions notables du dépôt `its-dp-fab-adp-alexandrie-managed` (configuration de déploiement, pipelines,
documentation) sont consignées ici, de la plus récente à la plus ancienne.

Format : [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/). Ce dépôt n'est pas versionné
par tag : chaque entrée est datée (`## AAAA-MM-JJ`). Les évolutions du socle commun sont décrites
dans le `CHANGELOG.md` de `its-dp-fab-cicd-common`, versionné par tag `vX.Y.Z`.

## 2026-10-30

### Added

- `fabric/.gitkeep` : dossier des items suivi par Git avant la première synchronisation.
- README : sections « Secrets et service connections », « Référencement dynamique et Git
  Integration », « Marqueurs à relever » et « Limites ».
- Journal des changements du dépôt (ce fichier).

### Changed

- Fichiers de configuration issus d'une copie du dépôt Framework, réécrits pour l'ADP Alexandrie
  (socle `its-dp-fab-cicd-common`, tag `v1.5.0`, référence ADP : dépôt MDS) :
  - `CICD/pipelines/CD.yml` : `repository: its-dp-fab-adp-alexandrie-managed`, `role: consumer`
    (seules valeurs admises par le template : `framework`, `consumer`), porte Framework `ado`.
  - `CICD/pipelines/CI.yml` : template KPMG conservé ; SonarQube et nom du pipeline à relever.
  - `CICD/pipelines/PR-Validation.yml` : en-tête Alexandrie, contenu inchangé.
  - `CICD/deployment/config.yml` : workspaces Alexandrie, `item_types_in_scope` vide, clé
    `core.parameter` retirée, `folder_exclude_regex` et `features` retirés.
  - `CICD/deployment/parameter.yml` : 56 règles Framework retirées ; aucune règle.
  - `CICD/deployment/dependencies.yml` : `role: consumer`, `category: adp`,
    `framework.required: true` (workspaces Framework, `ado_gate`), listes vides à relever.
  - `.gitignore` : convention du dépôt MDS.
  - README réécrit pour Alexandrie.

### À confirmer

- Noms des workspaces Alexandrie ITG, PPR et PRD.
- Organisation et service connection GitHub ; service connections ARM ITG, PPR et PRD.
- Nom du pipeline CI Alexandrie et nom exact de la CD Framework (`its-dp-fab-item-framework-managed-cd` attendu).
- Projet SonarQube Alexandrie.
- Types d'items, items Framework consommés, connexions Fabric, Variable Libraries et
  `guid_policy`, après synchronisation Git Integration du workspace DEV.
- Point V-01 : le filtre `contents: "!/*.json"` de l'artefact CI ne doit pas exclure les fichiers d'items.
- `CICD/ref_dynamique/` : copie de l'outillage Framework/MDS, non adaptée à Alexandrie.
