# Changelog

Toutes les évolutions notables du dépôt `its-dp-fab-adp-alexandrie-managed` (configuration de déploiement, pipelines,
documentation) sont consignées ici, de la plus récente à la plus ancienne.

Format : [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/). Ce dépôt n'est pas versionné
par tag : chaque entrée est datée (`## AAAA-MM-JJ`). Les évolutions du socle commun sont décrites
dans le `CHANGELOG.md` de `its-dp-fab-cicd-common`, versionné par tag `vX.Y.Z`.

## 2026-11-01

### Changed

- Alignement sur les autres dépôts consommateurs (socle `its-dp-fab-cicd-common`, tag `v1.5.0`,
  catégorie `adp` ; modèle : dépôt ADP IBS, lui-même aligné sur les dépôts BDP), à partir du
  commit `8caed28` :
  - `CICD/deployment/config.yml` : workspaces `FR - DP - <ENV> - FAB - ADP - Alexandrie - MANAGED`
    pour DEV, ITG, PPR et PRD (confirmés le 01/11/2026, tiret simple) ; les marqueurs
    `__A_RELEVER__WORKSPACE_ALEXANDRIE_<ENV>__` sont retirés.
  - `CICD/pipelines/CI.yml` : SonarQube `Discovery_DP-FAB-ADP-Alexandrie-Managed`,
    « [Discovery] DP FAB ADP Alexandrie Managed », porte « [Discovery] Starter », repris de
    l'ancienne CI ; marqueurs `__A_RELEVER__SONAR_*_ALEXANDRIE__` retirés.
  - `CICD/pipelines/CD.yml` : source CI `its-dp-fab-adp-alexandrie-managed-CI`, reprise de l'ancienne
    CD (marqueur `__A_RELEVER__PIPELINE_CI_ALEXANDRIE__` retiré) ; commentaire de la décision du
    31/10/2026 (3 service connections ARM distinctes ITG, PPR, PRD ; noms à renseigner).
  - `CICD/pipelines/PR-Validation.yml`, `CICD/deployment/dependencies.yml` et
    `CICD/deployment/parameter.yml` : en-têtes et commentaires alignés, contenu inchangé
    (`category: adp`, `framework.required: true`, aucune règle).
  - `CICD/ref_dynamique/01_inventaire_github.py` à `04_recoupement.py` : la copie de l'outillage
    Framework/MDS est remplacée par l'outillage des consommateurs, adapté à Alexandrie (dépôt
    seul, une branche, sorties hors du dépôt, activités imbriquées) ; `02` relève aussi le
    workspace Framework du même environnement, `03` propose les règles Framework par
    environnement et `framework_items_adp.yml`, `04` signale `FRAMEWORK_NON_DECLARE`.
  - `CICD/ref_dynamique/templates/config.yml` et `parameter.yml` : modèles ADP au format du socle
    (noms de workspaces, aucun GUID, aucune clé en minuscules).
  - README réécrit au format des autres consommateurs.

### Removed

- `fabric/test.md` (contenu « test », référencé par aucun pipeline ni script ; suppression validée).
  `fabric/.gitkeep`, présent depuis le 30/10/2026, est conservé.

### Vérifié

- `yamllint` (règles du socle) sur `CICD/deployment` et `CICD/pipelines` : sans erreur.
- `python -m compileall CICD/ref_dynamique` : sans erreur ; syntaxe compatible Python 3.9.
- `deploy.py validate` (socle `v1.5.0`) : code 4 attendu (« Aucun item Fabric »). Sur une copie de
  travail munie d'un item fictif : 0 erreur, `repository_directory` résolu vers `fabric/`, matrice
  `adp` acceptée.
- Scripts 01, 03 et 04 exécutés sur le dépôt (vide) : aucune écriture dans le dépôt.

### À confirmer

- Organisation et service connection GitHub ; noms des 3 service connections ARM ITG, PPR et PRD (création prévue).
- Casse exacte du pipeline CI dans Azure DevOps ; nom exact de la CD Framework.
- Types d'items, règles, `framework.items`, connexions Fabric, Variable Libraries, `guid_policy` et
  `data_dependencies`, après synchronisation Git Integration.
- Point V-01 : le filtre `contents: "!/*.json"` de l'artefact CI ne doit pas exclure les fichiers d'items.

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
