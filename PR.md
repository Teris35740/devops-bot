# DevOps and bots : PR

Authors : **Louis FLÉCHAIRE - Paul JOUAN - Tanguy MAIRE AMIGOT - Ones CHERIF**

## Contexte

Dans le cadre du module DevOps, nous avons dus configuré et intégré différents Software Bots afin d'automatiser les tâches répétitives, d'améliorer notre flux de travail et d'en apprendre davantage sur leur fonctionnement. L'objectif principal de ces ajouts est d'assurer une intégration continue fiable, de maintenir la sécurité de nos dépendances à jour et d'automatiser la communication.

## Objectifs

L’objectif de cette automatisation est de :

- sécuriser les mises à jour du projet,
- détecter rapidement les erreurs,
- améliorer la collaboration entre développeurs,
- automatiser les tâches répétitives,
- maintenir une meilleure qualité logicielle dans le temps.

Ces outils permettent de mettre en place une chaîne d’intégration continue (CI/CD) proche des pratiques utilisées dans les environnements professionnels.

---

# Fonctionnement global des bots

Les différents bots travaillent ensemble dans une chaîne automatisée :

1. Dependabot détecte une dépendance obsolète ou vulnérable.
2. Il crée automatiquement une Pull Request de mise à jour.
3. Le workflow CI/CD se lance automatiquement pour tester cette mise à jour.
4. Le Frontend Angular et le Backend Java sont compilés automatiquement.
5. Une notification est envoyée sur Discord avec le résultat du build.
6. Une fois la Pull Request fusionnée, Release Drafter prépare automatiquement les futures notes de version.

Cette automatisation permet de sécuriser les mises à jour tout en réduisant le travail manuel des développeurs.

---

# Nos Bots DevOps

Ce document résume les différents robots (*Software Bots*) mis en place sur notre projet afin d’automatiser plusieurs tâches importantes du cycle de développement logiciel.

---

# 1. Le Bot de mise à jour (Dependabot)

**Fichier :** `.github/dependabot.yml`

---

## Objectif

Dependabot automatise la maintenance des dépendances du projet.

Dans un projet moderne, les librairies JavaScript ou Java évoluent constamment :

- correction de bugs,
- amélioration des performances,
- correctifs de sécurité.

Sans maintenance régulière, le projet accumule de la dette technique et peut devenir vulnérable.

---

## Ce que nous avons mis en place

Nous avons créé un fichier de configuration YAML permettant à GitHub d’activer automatiquement Dependabot sur le projet.

Le bot surveille :

- le Frontend Angular (`npm`),
- le Backend Java (`maven`).

### Configuration utilisée

```yaml
version: 2

updates:
  # Bot pour le Frontend
  - package-ecosystem: "npm"
    directory: "/front"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5

  # Bot pour le Backend
  - package-ecosystem: "maven"
    directory: "/api"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
```

---

## Explication de la configuration YAML

### `package-ecosystem`

Définit le gestionnaire de dépendances surveillé :

- `npm` pour Angular / JavaScript,
- `maven` pour Java.

---

### `directory`

Indique le dossier du projet à analyser :

```yaml
directory: "/front"
```

Ici, Dependabot analyse toutes les dépendances présentes dans le Frontend.

---

### `schedule`

Définit la fréquence des vérifications automatiques.

```yaml
schedule:
  interval: "weekly"
```

GitHub vérifie automatiquement les mises à jour chaque semaine.

---

### `open-pull-requests-limit`

Limite le nombre de Pull Requests ouvertes automatiquement.

```yaml
open-pull-requests-limit: 5
```

Cela évite d’avoir trop de mises à jour simultanées.

---

## Ce que le bot fait

Dependabot analyse automatiquement les dépendances du Front-End et du Back-End.

S’il détecte :

- une version plus récente,
- une faille de sécurité,
- ou une dépendance obsolète,

il crée automatiquement une Pull Request contenant la mise à jour nécessaire.

---

## Son utilité dans une démarche DevOps

Dependabot joue un rôle essentiel dans la maintenance continue du projet :

- réduction des vulnérabilités,
- automatisation des mises à jour,
- limitation de la dette technique,
- amélioration de la stabilité du projet.

Couplé à la CI/CD, chaque mise à jour proposée est automatiquement testée avant intégration.

---

# 2. Le Bot de Vérification (CI/CD)

**Fichier :** `.github/workflows/ci.yml`

---

## Objectif

Ce workflow met en place une chaîne d’intégration continue (*Continuous Integration*).

À chaque modification du projet, GitHub lance automatiquement plusieurs étapes permettant de vérifier que l’application fonctionne correctement.

---

## Déclenchement automatique du workflow

Le workflow se lance automatiquement lorsqu’un développeur :

- pousse du code sur `main` ou `master`,
- ouvre une Pull Request.

### Configuration utilisée

```yaml
name: Bot CI/CD Doodle

on:
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]
```

Cette automatisation garantit que chaque modification importante est vérifiée avant intégration.

---

## Structure du workflow

### Nom du workflow

```yaml
name: Bot CI/CD Doodle
```

Permet d’identifier facilement le workflow dans l’onglet **Actions** de GitHub.

---

### Environnement d’exécution

```yaml
runs-on: ubuntu-latest
```

GitHub crée automatiquement une machine virtuelle Linux temporaire pour exécuter les étapes du pipeline.

---

## Étapes principales du workflow

### 1. Récupération du code

```yaml
- name: Récupération du code
  uses: actions/checkout@v3
```

Cette étape clone automatiquement le dépôt GitHub dans la machine virtuelle.

---

### 2. Setup du Frontend Angular

```yaml
- name: Setup Node.js (Frontend)
  uses: actions/setup-node@v3
  with:
    node-version: '14'
```

Cette étape installe Node.js version 14 afin de pouvoir compiler le Frontend Angular.

---

### 3. Build du Frontend

```yaml
- name: Build Frontend
  working-directory: ./front
  run: |
    npm install
    npm run build --if-present
```

Le bot :

- installe les dépendances npm,
- compile automatiquement l’application Angular.

---

### 4. Setup du Backend Java

```yaml
- name: ☕ Setup Java (Backend)
  uses: actions/setup-java@v3
  with:
    distribution: 'temurin'
    java-version: '11'
```

Cette étape installe Java 11 avec la distribution Temurin afin de compiler l’API Java.

---

### 5. Build du Backend

```yaml
- name: Build Backend
  working-directory: ./api
  run: ./mvnw clean package -DskipTests
```

Le bot compile automatiquement le Backend Java grâce à Maven.

La commande :

```bash
./mvnw clean package
```

permet :

- de nettoyer les anciens fichiers compilés,
- de reconstruire complètement l’API.

L’option :

```bash
-DskipTests
```

désactive temporairement les tests pendant la compilation.

---

## Ce que le bot fait

À chaque modification importante du projet, le bot :

1. télécharge le code,
2. prépare un environnement propre,
3. installe les dépendances,
4. compile le Frontend Angular,
5. compile le Backend Java,
6. vérifie que le projet peut être construit sans erreur.

Si une erreur apparaît, GitHub bloque l’intégration et affiche l’échec dans l’onglet **Actions**.

---

## Son utilité dans une démarche DevOps

Ce bot agit comme un filet de sécurité automatique :

- il évite d’intégrer du code cassé,
- il valide les mises à jour proposées par Dependabot,
- il améliore la qualité du code,
- il réduit les erreurs humaines,
- il sécurise les futures mises en production.

---

# 3. Le Bot de Notification (Discord)

**Où ?**

- À la fin du fichier `.github/workflows/ci.yml`
- Dans les *Secrets* GitHub (`DISCORD_WEBHOOK`)

---

## Objectif

Ce système permet d’envoyer automatiquement des notifications sur Discord après l’exécution de la CI/CD.

L’objectif est de tenir l’équipe informée en temps réel de l’état du projet.

---

## Configuration utilisée

```yaml
- name: Bot Notification Discord
  if: always()
  uses: Ilshidur/action-discord@master
  env:
    DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
  with:
    args: 'Build du projet Doodle par ${{ github.actor }} terminé ! Résultat : **${{ job.status }}**'
```

---

## Explication de la configuration

### `if: always()`

Cette instruction force l’envoi du message Discord même si le build échoue.

Ainsi, l’équipe est toujours informée du résultat.

---

### `DISCORD_WEBHOOK`

```yaml
DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
```

Le Webhook Discord est stocké dans les *Secrets GitHub* afin de sécuriser les informations sensibles.

Cela évite d’exposer le lien Discord publiquement dans le code source.

---

### `${{ github.actor }}`

Affiche automatiquement le nom du développeur ayant déclenché le workflow.

---

### `${{ job.status }}`

Affiche automatiquement le résultat du pipeline :

- `success`
- `failure`

---

## Ce que le bot fait

Une fois le workflow terminé :

- un message est envoyé automatiquement sur Discord,
- l’équipe est informée du succès ou de l’échec du build,
- les développeurs peuvent réagir rapidement en cas de problème.

---

## Son utilité dans une démarche DevOps

Ce bot améliore :

- la communication d’équipe,
- la rapidité de réaction,
- la visibilité sur l’état du projet.

Les notifications automatiques sont très utilisées dans les pipelines DevOps professionnels.

---

# 4. Le Bot de gestion des Releases (Release Drafter)

**Fichiers :**

- `.github/workflows/release-drafter.yml`
- `.github/release-drafter.yml`

---

## Objectif

Release Drafter automatise la préparation des notes de version (*Changelog*).

L’objectif est de garder une trace claire des évolutions du projet sans devoir rédiger manuellement chaque release.

---

## Workflow GitHub Actions

### Déclenchement

```yaml
name: Release Drafter

on:
  push:
    branches:
      - main
      - master
```

Le workflow se lance automatiquement après chaque mise à jour importante du projet.

---

### Permissions GitHub

```yaml
permissions:
  contents: read
  pull-requests: write
```

Ces permissions permettent au bot :

- de lire le contenu du dépôt,
- de modifier automatiquement les brouillons de release.

---

## Configuration du Release Drafter

### Nom automatique des versions

```yaml
name-template: 'v$RESOLVED_VERSION'
tag-template: 'v$RESOLVED_VERSION'
```

Les versions sont générées automatiquement selon un format standardisé.

---

### Catégories automatiques

```yaml
categories:
  - title: 'Nouvelles Fonctionnalités'
    labels:
      - 'feature'
      - 'enhancement'
```

Les Pull Requests sont automatiquement classées selon leurs labels GitHub.

---

### Gestion des dépendances et sécurité

```yaml
- title: 'Dépendances & Sécurité (Dependabot)'
  labels:
    - 'dependencies'
    - 'security'
```

Les mises à jour Dependabot apparaissent automatiquement dans une catégorie dédiée.

---

### Format des changements

```yaml
change-template: '- $TITLE par @$AUTHOR dans la PR #$NUMBER'
```

Chaque modification est automatiquement affichée avec :

- le titre,
- l’auteur,
- le numéro de Pull Request.

---

### Template du changelog

```yaml
template: |
  ## Quoi de neuf dans cette version ?
  $CHANGES
```

Release Drafter génère automatiquement une release lisible et structurée.

---

## Ce que le bot fait

À chaque fusion de Pull Request sur `main` :

- Release Drafter analyse les labels associés,
- il met à jour automatiquement un brouillon de release,
- il génère une liste organisée des nouveautés, corrections et améliorations.

---

## Son utilité dans une démarche DevOps

Ce bot professionnalise la gestion des versions :

- génération automatique du changelog,
- meilleure traçabilité des modifications,
- gain de temps lors des releases,
- meilleure communication autour des nouvelles versions.

Cela facilite également la maintenance et le suivi du projet sur le long terme.

---

# Rôle des Software Bots dans une démarche DevOps

Dans une approche DevOps, les *Software Bots* jouent un rôle essentiel dans l’automatisation du cycle de développement logiciel.

Le DevOps repose principalement sur trois grands objectifs :

- automatiser les tâches répétitives,
- améliorer la qualité et la stabilité des applications,
- accélérer les livraisons tout en réduisant les erreurs humaines.

Les bots permettent justement d’atteindre ces objectifs en exécutant automatiquement certaines tâches importantes sans intervention manuelle.

---

## Pourquoi les Software Bots sont importants ?

Dans un projet classique sans automatisation, les développeurs doivent souvent :

- vérifier manuellement les dépendances,
- lancer les compilations,
- exécuter les tests,
- rédiger les changelogs,
- prévenir l’équipe en cas d’erreur,
- surveiller les failles de sécurité.

Ces tâches prennent du temps et augmentent les risques d’oubli ou d’erreur humaine.

Les bots DevOps permettent donc :

- un gain de temps important,
- une meilleure fiabilité du projet,
- une maintenance continue,
- une détection rapide des problèmes,
- une meilleure collaboration entre développeurs.

---

## Automatisation et Intégration Continue

Les bots sont au cœur des pratiques modernes d’intégration continue (*Continuous Integration*).

À chaque modification du code :

1. les bots récupèrent automatiquement le projet,
2. installent les dépendances,
3. compilent l’application,
4. exécutent des vérifications,
5. notifient les développeurs du résultat.

Cette automatisation garantit qu’une nouvelle modification ne casse pas le projet principal.

---

## Maintenance et Sécurité

Les bots comme Dependabot jouent également un rôle important dans la cybersécurité du projet.

Ils permettent :

- d’identifier rapidement les dépendances vulnérables,
- d’appliquer plus facilement les correctifs,
- de maintenir un environnement logiciel à jour.

Dans les projets modernes contenant de nombreuses bibliothèques externes, cette surveillance automatique devient essentielle.

---

## Collaboration et Communication

Les bots améliorent aussi la communication au sein d’une équipe de développement.

Grâce aux notifications automatiques :

- toute l’équipe est informée rapidement,
- les erreurs sont détectées plus tôt,
- les développeurs peuvent réagir immédiatement.

Cela réduit les temps de correction et améliore l’organisation du travail.

---

## Standardisation et Professionnalisation

Les bots permettent également de standardiser les processus :

- mêmes vérifications pour tous les développeurs,
- mêmes règles de validation,
- même structure de release,
- mêmes contrôles de qualité.

Cette standardisation est très importante dans les environnements professionnels car elle améliore :

- la stabilité des projets,
- la maintenabilité,
- la qualité globale du développement logiciel.


# Conclusion

La mise en place de ces bots nous a permis de construire une première chaîne DevOps automatisée autour du projet.

Grâce à GitHub Actions, Dependabot, Discord Webhooks et Release Drafter, nous avons automatisé :

- la maintenance des dépendances,
- les vérifications du projet,
- les notifications d’équipe,
- et la préparation des releases.

Cette approche améliore :

- la qualité du code,
- la stabilité de l’application,
- la sécurité du projet,
- la rapidité de développement,
- et la collaboration au sein de l’équipe.