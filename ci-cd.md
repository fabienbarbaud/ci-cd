---
marp: true
theme: default
---
<!-- page_number: true -->
<!-- footer: Intégration continue (CI) et déploiement continu (CD) -->

Intégration continue (CI) et déploiement continu (CD)
===

![](images/schema-devops-en-entreprise.png)
Source de l'illustration : nemesis-studio.com

##### Mise en pratique CI-CD sous Gitlab

###### par [Fabien Barbaud](fabien.barbaud@timeonegroup.com) - [@BarbaudFabien](https://twitter.com/BarbaudFabien)

---

# Intégration continue

*Continuous integration*

L'**intégration continue** est un ensemble de pratiques utilisées en génie logiciel consistant **à vérifier** à chaque **modification de code source** que le résultat des modifications ne produit **pas de régression** dans l'application développée.

[Wikipedia](https://fr.wikipedia.org/wiki/Int%C3%A9gration_continue)

---

# Déploiement continu

Le **déploiement continu** ou *Continuous deployment* (CD) en anglais, est une approche d'ingénierie logicielle dans laquelle les fonctionnalités logicielles sont **livrées fréquemment** par le biais de **déploiements automatisés**.

[Wikipedia](https://fr.wikipedia.org/wiki/D%C3%A9ploiement_continu)

---

# Notre objectif :

![](images/CI-CD.png)

---

# Docker Swarm

![](images/docker-swarm.png)

[Orchestrateur](https://fr.wikipedia.org/wiki/Orchestration_informatique) [Docker](https://docs.docker.com/get-started/swarm-deploy/)

```bash
$ docker swarm init
```

---

# Gitlab

![](images/langfr-330px-GitLab_logo.svg.png)

**GitLab** est un **logiciel libre** de **forge** basé sur git proposant les fonctionnalités de wiki, un système de suivi des bugs, **l’intégration continue et la livraison continue**.

[Wikipedia](https://fr.wikipedia.org/wiki/GitLab)

---

# Gitlab

```bash
$ git clone https://github.com/fabienbarbaud/gitlab-docker.git
$ cd gitlab-docker
$ docker network create -d overlay cesi --scope swarm --attachable
$ docker stack deploy --compose-file docker-compose.yml gitlab
```

```bash
$ docker service ls
$ docker service logs --tail 50 -f gitlab_gitlab
```

http://localhost
u: `root`
p: `MySuperSecretAndSecurePass0rd!`

---

# Gitlab

![](images/ssh_key.png)
http://localhost/-/profile/keys
http://localhost/help/ssh/README#generate-an-ssh-key-pair

```bash
$ ssh git@localhost
```

---

# Gitlab

- Créer un premier projet "test" - http://localhost/root/test
- Cloner ce projet sur votre poste
- Faire un premier commit
- Un push

---

# git-flow

Le workflow Gitflow définit **un modèle de création de branche strict** conçu autour de la **livraison de projet**. Cela fournit un **framework** solide pour la gestion de projets plus importants.

[Atlassian](https://www.atlassian.com/fr/git/tutorials/comparing-workflows/gitflow-workflow)

---

# git-flow

## Branches develop et master

![width:700px](images/gitflow-1.svg)

---

# git-flow

## Branche feature

![width:700px](images/gitflow-2.svg)

---

# git-flow

## Branche release

![width:700px](images/gitflow-3.svg)

---

# git-flow

## Branche hotfix

![width:600px](images/gitflow-4.svg)

---

# git-flow

## Les commandes

```bash
$ git flow init
$ git flow feature start ma-feature
$ git flow finish -p
$ git flow release start 0.1.0
$ git flow finish -p
$ git flow hotfix start 0.1.1
$ git flow finish -p
```

---

# Linter

## Outils d'analyse statique

- Javascript : ESLint
- PHP : PHPLint, PHPStan
- Python : pylint

---

# Test unitaire

En programmation informatique, le test unitaire (ou « T.U. », ou « U.T. » en anglais) est une procédure permettant de **vérifier** le bon **fonctionnement** d'une **partie précise** d'un logiciel ou d'une **portion d'un programme** (appelée « unité » ou « module »).

[Wikipedia](https://fr.wikipedia.org/wiki/Test_unitaire)

---

# Test fonctionnel

Les tests fonctionnels sont destinés à s’assurer que, **dans le contexte d’utilisation réelle**, le comportement fonctionnel obtenu est **bien conforme** avec celui attendu.

Un test fonctionnel a donc pour objectif de **dérouler un scénario** composé d’une liste d’actions, et pour chaque action d’effectuer une **liste de vérifications** validant la conformité de l’exigence avec l’attendu.

[Qu’est-ce qu’un test fonctionnel ?
](https://horustest.io/blog/definition-test-fonctionnel/)

---

# Un exemple sur un app Python : Flaskex

## L'app Flaskex

```bash
$ git clone https://github.com/fabienbarbaud/flaskex
$ cd flaskex
$ docker-compose up app
```

http://localhost:5000

---

# Un exemple sur un app Python : Flaskex

## Le linter

```bash
$ docker-compose up linter
```

---

# Un exemple sur un app Python : Flaskex

## Les tests

```bash
$ docker-compose up test
```

---

# Selenium

## Selenium automates browsers. That's it!

![](images/selenium_logo_large.png)

https://www.selenium.dev/

---

# Selenium

## Selenium IDE

Open source record and playback test automation for the web

https://www.selenium.dev/selenium-ide/

---

# CI-CD

## Gitlab runner

```
$ docker ps
$ docker exec -it gitlab_gitlab-runner.1.3hdx4ch6guq5c88xpmed9yk0s bash
bash-5.0# gitlab-runner \
    register -n \
    --name "Docker Runner" \
    --executor docker \
    --docker-image docker:latest \
    --docker-volumes /var/run/docker.sock:/var/run/docker.sock \
    --url http://gitlab_gitlab \
    --clone-url http://gitlab_gitlab \
    --docker-privileged \
    --docker-network-mode cesi \
    --registration-token z5xxGJZZpzJp5Wz_s4h3
```

---

# CI-CD

## .gitlab-ci.yml

```yaml
image: docker:latest

stages:
  - build
  - linter
  - test

build:
  stage: build
  script: 
    - docker build -t image .

linter:
  stage: linter
  script:
    - docker run image bash docker/start_linter.sh
```

---

# CI-CD

## Démarrage du service

```bash
$ docker build -f Dockerfile_prod -t root/flaskex:latest .
$ docker service create -p 5000:5000 --name flaskex root/flaskex:latest
```

---

# CI-CD

## Déploiement

```yaml
deploy:
  stage: deploy
  only:
    - tags
  script:
    - docker build -f Dockerfile_prod -t $CI_PROJECT_PATH:$CI_COMMIT_REF_NAME .
    - docker service update --image $CI_PROJECT_PATH:$CI_COMMIT_REF_NAME flaskex
```

---

# TP

Mettre en place une application de votre choix dans le langage de votre choix comprenant :
- un linter
- des tests unitaires
- un déploiement

Tout ça sur un projet en CI-CD avec Gitlab et Docker Swarm