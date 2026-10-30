# its-dp-fab-adp-alexandrie-managed — ADP Alexandrie

Dépôt GitHub des items Microsoft Fabric du workspace **ADP Alexandrie**, et de leur configuration de déploiement.

Ce dépôt ne contient **aucun code de déploiement**. Il décrit *quoi* déployer et *où*. La logique (contrôles, publication, vérification, rapport) se trouve dans le dépôt socle commun `its-dp-fab-cicd-common`, appelé par le tag `v1.5.0`.

> **État au 30/10/2026 : le dépôt ne contient encore aucun item Fabric, et rien n'est déployable.**
> `fabric/` ne contient que `.gitkeep` et `test.md`. `deploy.py validate` s'arrête en code 4 (« Aucun item Fabric ») avant les contrôles statiques, et la CD refuserait de toute façon toute publication (`inventory_status: pending`, marqueurs à relever).

| Rubrique | Valeur |
|---|---|
| Catégorie | `adp` (Data Product de type ADP) ; rôle `consumer` dans le socle |
| Workspace DEV | `FR - DP - DEV - FAB - ADP - Alexandrie - MANAGED` |
| Workspaces ITG, PPR, PRD | À relever dans le portail Fabric (marqueurs dans `CICD/deployment/config.yml`) |
| Branches | `develop` (par défaut) · `release/*` · `main` |
| Pipelines Azure DevOps | CI : à créer depuis `CICD/pipelines/CI.yml` (nom à relever) · CD : à créer depuis `CICD/pipelines/CD.yml` · validation de PR : à créer depuis `CICD/pipelines/PR-Validation.yml` |
| Socle commun | `its-dp-fab-cicd-common`, tag `v1.5.0` |
| Statut d'inventaire | `pending` : la CD refuse toute publication tant que l'inventaire n'est pas validé sur la branche réelle |

## Dépendances

**Prérequis de déploiement unique : le Framework** (`its-dp-fab-item-framework-managed`).

- Avant chaque déploiement sur ITG, PPR ou PRD, la CD vérifie que le Framework est **disponible** sur cet environnement :
  - le workspace Framework de l'environnement existe (pré-vol, `PRE_FRAMEWORK_WORKSPACE`) ;
  - les items Framework utilisés par Alexandrie sont présents (pré-vol, `PRE_FRAMEWORK_ITEM`). Ils sont à déclarer dans `framework.items`, aujourd'hui vide ;
  - le dernier stage `Deploy_<ENV>` de la CD Framework a réussi et aucun n'est en cours (porte Azure DevOps). La porte lit l'historique **sur la même branche** : `develop` pour ITG, `release/*` pour PPR, `main` pour PRD.
- Tant que la **nouvelle** CD Framework n'a pas tourné au moins une fois sur l'environnement visé, la porte renvoie `GATE_NEVER_DEPLOYED`. Un Framework publié par l'ancienne CD est présent dans Fabric, mais il est invisible pour la porte.
- La CD **n'exige jamais de redéployer le Framework** : s'il est déjà disponible, Alexandrie se déploie immédiatement.
- **Aucune dépendance vers les BDP ni vers les autres ADP** : ordre libre, déploiement possible en parallèle.

| Item Framework utilisé | Appelé depuis |
|---|---|
| À relever à l'inventaire | — |

Connexions Fabric requises : à relever à l'inventaire, puis à déclarer dans `connections` de `CICD/deployment/dependencies.yml` (vérifiées au pré-vol).

## Arborescence

```text
its-dp-fab-adp-alexandrie-managed/         branches develop (défaut) · release/* · main
│
├── CHANGELOG.md                         # journal des changements du dépôt (entrées datées)
├── README.md                            # ce fichier
│
│  ── Déploiement (lu par Azure DevOps, jamais publié dans Fabric) ──
├── CICD/
│   ├── pipelines/
│   │   ├── CI.yml                       # pipeline ADO de CI (template KPMG) : version, SonarQube, artefact « fabric »
│   │   ├── CD.yml                       # pipeline ADO de CD → CICD/pipelines/templates/cd-pipeline.yml@cicd_common
│   │   └── PR-Validation.yml            # check obligatoire des PR → CICD/pipelines/templates/validate-pipeline.yml@cicd_common
│   ├── deployment/
│   │   ├── config.yml                   # workspaces Alexandrie (DEV connu, ITG/PPR/PRD à relever),
│   │   │                                #   repository_directory: "../../fabric", types (vide)
│   │   ├── parameter.yml                # aucune règle (en-tête : familles de règles attendues)
│   │   └── dependencies.yml             # category adp : Framework seul prérequis (workspaces, porte ADO),
│   │                                    #   items, connexions, Variable Libraries, guid_policy à relever
│   └── ref_dynamique/                   # copie de l'outillage Framework/MDS, NON adapté à Alexandrie (voir plus bas)
│       └── templates/
│
│  ── Contenu du workspace ADP Alexandrie (Git Integration DEV + publication fabric-cicd) ──
└── fabric/
    ├── .gitkeep
    └── test.md
```

Cette arborescence reflète le contenu du dépôt au 30/10/2026. Elle sera complétée par les items du workspace DEV.

`repository_directory` est résolu **relativement au dossier de `config.yml`** (`CICD/deployment/`), par fabric-cicd 1.4.0 comme par le socle : `"../../fabric"` désigne le dossier `fabric/` à la racine du dépôt.

### Rôle des fichiers YAML

| Fichier | Lu par | Rôle |
|---|---|---|
| `CICD/pipelines/CI.yml` | Azure DevOps (template KPMG `PipelineTemplate.yaml@CICD`) | Build sur `develop`, `release/*`, `main` ; publie l'artefact `fabric` consommé par la CD |
| `CICD/pipelines/CD.yml` | Azure DevOps → socle `cd-pipeline.yml` | Un run par build CI ; un seul environnement par run, selon la branche |
| `CICD/pipelines/PR-Validation.yml` | Azure DevOps → socle `validate-pipeline.yml` | Contrôles statiques bloquants sur chaque PR |
| `CICD/deployment/config.yml` | fabric-cicd 1.4.0 et socle | Workspaces cibles par nom, dossier des items, types publiés, publish / unpublish |
| `CICD/deployment/parameter.yml` | fabric-cicd 1.4.0 et socle | Remplacement des références par environnement (`key_value_replace`) |
| `CICD/deployment/dependencies.yml` | socle uniquement | Rôle, catégorie, prérequis Framework, connexions, Variable Libraries, politique GUID |

## Exclusions (`CICD/deployment/config.yml`)

- `item_types_in_scope` est **vide** : il sera rempli uniquement avec les types relevés dans les fichiers `.platform` de `fabric/`. Une liste vide est bloquante (`TYPES_SCOPE_EMPTY`).
- Aucun `folder_exclude_regex` ni feature flag : aucun dossier à exclure n'existe.
- La clé `core.parameter` est absente tant que `parameter.yml` n'a aucune règle. Il faudra l'ajouter (`parameter: "parameter.yml"`) avec la première règle.
- Aucune suppression automatique : `unpublish` est désactivé sur les 4 environnements. Supprimer un item en ITG, PPR ou PRD reste une action manuelle validée.

## Branches et déploiement

| Branche | Rôle | Déploiement | Réglage |
|---|---|---|---|
| `feature/*` | Développement | Aucun | PR vers `develop` |
| `develop` | Branche par défaut ; intégration des features | DEV par Git Integration ; ITG par la CD | Protégée ; workspace DEV connecté à `develop`, dossier Git `fabric` |
| `release/*` | Une branche par release (lot de features), créée depuis `develop` au besoin | PPR par la CD, après approbation | Protégée (PR obligatoire, check PR-Validation) |
| `main` | Reflet de la production | PRD par la CD, après approbation | Protégée (PR obligatoire depuis `release/*`) |

Un run de CD ne déploie qu'**un seul environnement** : celui de la branche d'où vient l'artefact CI. Toute autre branche est refusée. DEV n'est jamais déployé par la CD (`devMode: skip`).

## Secrets et service connections

- **Identité de publication** : le SPN de l'environnement (`authMode: secret`). Ses trois secrets `<ENV>-AZURE-CLIENT-ID`, `-CLIENT-SECRET` et `-TENANT-ID` sont lus dans le coffre **unique** `frfrkvdevdp0001` par la tâche `AzureKeyVault@2` du template imposé.
- **Service connections Azure Resource Manager** : une par environnement (ITG, PPR, PRD), **distinctes**, créées par Ops en fédération d'identité. Chacune est limitée aux 3 secrets de son environnement. Tant que le marqueur `__A_CONFIRMER_SERVICE_CONNECTION_ARM_<ENV>__` est présent, le stage échoue à la compilation.
- Les service connections ARM **ne sont pas** des connexions Fabric : celles-ci sont utilisées par les items à l'exécution et se déclarent dans `dependencies.yml`.
- Aucun secret ni identifiant d'authentification dans ce dépôt : la CI le refuse (`SECRET_*`).

## Référencement dynamique et Git Integration

Le référencement dynamique (règles `parameter.yml`, `framework.items`, connexions, `guid_policy`) **reste à construire**. Procédure :

1. Connecter le workspace DEV `FR - DP - DEV - FAB - ADP - Alexandrie - MANAGED` à la branche `develop`, **dossier Git `fabric`** (Source control de Fabric), puis synchroniser.
2. Relever dans les items synchronisés :
   - les types présents (fichiers `.platform`) → `item_types_in_scope` ;
   - les références au workspace courant → règles `$workspace.$id` / `$items.<Type>.<nom>.$id` ;
   - les références au Framework → règles `$workspace.FR - DP - <ENV> - FAB - Framework - MANAGED.$items.<Type>.<nom>.$id`, et déclaration dans `framework.items` ;
   - les connexions et Variable Libraries ; les GUID sandbox → `guid_policy`.
3. Lancer `deploy.py validate` jusqu'à zéro erreur.

`CICD/ref_dynamique/` est une copie de l'outillage du Framework. Ses scripts et modèles sont codés pour le Framework et MDS (`02_fab_cli.py` s'arrête sur tout autre dépôt) : **ils ne sont pas utilisables en l'état pour Alexandrie**. Ils seront à adapter si l'inventaire le justifie.

## Contribuer

1. Travailler dans le workspace DEV, puis enregistrer sur une branche `feature/*` (Source control de Fabric).
2. Si un item appelle un item du Framework :
   - ajouter la règle correspondante dans `CICD/deployment/parameter.yml`, au format `$workspace.FR - DP - <ENV> - FAB - Framework - MANAGED.$items.<Type>.<nom>.$id` ;
   - déclarer l'item dans `framework.items` de `CICD/deployment/dependencies.yml`.
3. Ne jamais référencer le workspace d'un autre Data Product (ADP, BDP, UCS) dans `parameter.yml` : la CI le refuse (`DEPS_CROSS_WORKSPACE_FORBIDDEN`). Une lecture de données se fait à l'exécution et se déclare, pour information, dans `data_dependencies`.
4. Ouvrir une PR vers `develop` et corriger jusqu'à ce que le check PR-Validation soit vert.

Pour contrôler le dépôt en local, à partir d'un clone du socle (Python 3.12, `pip install -r ../its-dp-fab-cicd-common/CICD/tools/requirements.txt`) :

```bash
python ../its-dp-fab-cicd-common/CICD/tools/deploy.py validate --repo .
```

## Ce qui ne doit pas figurer dans ce dépôt

| Élément | Pourquoi |
|---|---|
| `.github/workflows/CI.yml`, `.github/workflows/CD.yml` | Remplacés par `CI.yml`, `CD.yml` et `PR-Validation.yml` sous `CICD/pipelines/` (pipelines Azure DevOps) |
| `deploy.py` dans `deployment/` ou `CICD/deployment/` | Remplacé par `CICD/tools/deploy.py` du socle |
| Items à la racine du dépôt | Ils vont sous `fabric/` |
| Items copiés d'un autre dépôt, GUID en dur, secrets | Seuls les items du workspace Alexandrie, synchronisés par Git Integration |

## Avant le premier déploiement ITG

- [ ] Remplacer les marqueurs de `CICD/pipelines/CD.yml` et `CICD/pipelines/PR-Validation.yml` :
  - `__A_RELEVER__` : organisation et service connection GitHub, nom du pipeline CI, nom de la CD Framework ;
  - `__A_CONFIRMER_SERVICE_CONNECTION_ARM_ITG__` : service connection Azure Resource Manager ITG, créée par Ops en fédération d'identité, autorisée à lire les 3 secrets ITG dans `frfrkvdevdp0001`.
- [ ] Créer le projet SonarQube et renseigner `CICD/pipelines/CI.yml`.
- [ ] Créer les pipelines ADO CI, CD et PR-Validation ; déclarer PR-Validation comme « Required status check ».
- [ ] Relever le nom exact du workspace ITG dans `CICD/deployment/config.yml`.
- [ ] Synchroniser le workspace DEV dans `fabric/` (Git Integration, dossier `fabric`).
- [ ] Remplir `item_types_in_scope`, `parameter.yml` (et `core.parameter`), `framework.items`, `connections`, `variable_libraries` et `guid_policy` à partir des items réels.
- [ ] Lancer `deploy.py validate` sur la branche réelle jusqu'à zéro erreur.
- [ ] Passer `inventory_status` à `validated` dans une PR revue.
- [ ] Vérifier que la nouvelle CD Framework a réussi au moins une fois sur ITG (sinon `GATE_NEVER_DEPLOYED`), et que le service de build a le droit « View builds » sur sa définition.

## Marqueurs à relever

| Marqueur | Fichier | Source |
|---|---|---|
| `__A_RELEVER__SERVICE_CONNECTION_GITHUB__`, `__A_RELEVER__ORG_GITHUB__` | `CD.yml`, `PR-Validation.yml` | ADO (service connections du projet) |
| `__A_RELEVER__PIPELINE_CI_ALEXANDRIE__` | `CI.yml` (en-tête), `CD.yml` | ADO (nom donné à la définition CI) |
| `__A_RELEVER__PIPELINE_CD_FRAMEWORK__` | `CD.yml`, `dependencies.yml` | ADO (attendu : `its-dp-fab-item-framework-managed-cd`, non confirmé) |
| `__A_RELEVER__SONAR_*_ALEXANDRIE__` | `CI.yml` | Plateforme Technologique |
| `__A_CONFIRMER_SERVICE_CONNECTION_ARM_<ENV>__` | `CD.yml` | Ops |
| `__A_RELEVER__WORKSPACE_ALEXANDRIE_<ENV>__` (ITG, PPR, PRD) | `config.yml` | Portail Fabric |
| `__A_RELEVER__ITEM_TYPES_ALEXANDRIE__` | `config.yml` (commentaire) | Fichiers `.platform` après Git Integration |
| `__A_RELEVER__ITEMS_FRAMEWORK_CONSOMMES__`, `__A_RELEVER__CONNEXIONS_FABRIC_ALEXANDRIE__`, `__A_RELEVER__VARIABLE_LIBRARIES_ALEXANDRIE__`, `__A_RELEVER__GUID_POLICY_ALEXANDRIE__` | `dependencies.yml` (commentaires) | Inventaire des items |

## Limites

- Aucun item : la configuration est **non déployable**. La validation locale et la PR-Validation échouent en code 4 tant que `fabric/` ne contient aucun `.platform`.
- Les contrôles statiques (`CONFIG_*`, `DEPS_*`, `TYPES_*`, `PARAM_*`) ne sont exécutés qu'une fois des items présents : la conformité de `config.yml` et de `dependencies.yml` au socle n'est pas encore prouvée par `deploy.py validate`.
- Un YAML syntaxiquement valide ne garantit pas que le contrat Azure DevOps est respecté (ressources, service connections, Environments) : seule une compilation dans ADO le prouve.
- Le filtre `contents: "!/*.json"` de l'artefact CI (point V-01) n'est pas vérifié.
- `CICD/ref_dynamique/` n'est pas adapté à Alexandrie.
