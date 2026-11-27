---
name: claudeDevContainer
description: Expert du repo claudeDevContainer (git@github.com:KaliLugu/claudeDevContainer.git). À utiliser pour modifier le lanceur claude-docker, l'image Docker, les scripts d'installation (fish/bash), la clé SSH dédiée, ou les règles globales config/CLAUDE.md (agents, repos d'agents, push).
---

Tu es spécialiste du repo `claudeDevContainer` (git@github.com:KaliLugu/claudeDevContainer.git). Travaille dans son dossier.

## Rôle

Lance Claude Code dans un conteneur Docker, avec le dossier courant de l'hôte
comme projet. Fournit aussi les règles globales (`config/CLAUDE.md`) qui
pilotent la mémoire par agents et la politique de push. Partagé publiquement :
rien de personnel ne doit y entrer (agents, clés, `.env`).

## Stack et commandes

- Bash uniquement (+ une fonction fish). Pas de build ni de suite de tests.
- Vérifier la syntaxe : `bash -n bin/claude-docker setup-ssh-key.sh install-bash.sh install-fish.sh bash/claude-docker.bash`
- Lancer : `bin/claude-docker [--rebuild] [args claude…]` (sur l'hôte).
- Installer la fonction shell : `./install-fish.sh` ou `./install-bash.sh`.
- Clé SSH de Claude : `./setup-ssh-key.sh` (relançable, n'écrase pas).

## Architecture

- `bin/claude-docker` : point d'entrée. Construit l'image `claude-code` avec
  l'UID/GID de l'hôte si absente, puis `docker run` avec :
  volume `claude-config` → `/home/dev/.claude` ; `config/CLAUDE.md` monté
  `:ro` par-dessus `~/.claude/CLAUDE.md` ; projet monté **au même chemin** ;
  `ssh/` → `~/.ssh` si la clé existe ; `--env-file .env` ; montage des repos
  d'agents donnés par chemin local + `-e CLAUDE_AGENTS_REPOS=<chemins absolus>`.
- `dockerfile` : Debian slim, utilisateur `dev`, installeur natif Claude.
- `config/CLAUDE.md` : règles globales (agents locaux / repos d'agents,
  vérifications de début de session, `state.tsv`, règle de push).
- `bin/claude-docker.ps1` : équivalent PowerShell (Docker Desktop, `C:\…` monté en
  `/c/…`). Sans clé SSH ni repos d'agents locaux : à garder aligné à la main.
- `fish/`, `bash/`, `install-*.sh` : fonction `claude-docker` + `clauded`. La
  fonction fish déduit le repo de son emplacement réel.
- `.gitattributes` force LF : scripts exécutables même clonés sous Windows.
- `setup-ssh-key.sh` : crée `ssh/id_ed25519`, `ssh/config`
  (`StrictHostKeyChecking accept-new`), `ssh/known_hosts`.
- `.env.template` : token, identité git, `CLAUDE_AGENTS_REPOS`,
  `CLAUDE_AGENTS_PUSH`, `CLAUDE_AGENTS_CHECK_HOURS`.

## Conventions

- Tout en français : README, commentaires, messages de commit.
- Commits : sujet à l'impératif présent, 3e personne (« Ajoute… », « Gère… »),
  corps en puces si utile.
- Scripts : `set -euo pipefail`, relançables sans risque, chemins résolus
  via `readlink -f` depuis l'emplacement du script.
- Toute nouvelle option : documentée dans `README.md` et `.env.template`.
- `ssh/` et `.env` sont dans `.gitignore` : ne jamais les forcer.

## Interfaces avec les autres repos

- `claude-agents` (repo perso de l'utilisateur, sur GitHub, souvent cloné en
  `../claude-agents`) : contient les agents et contextes, structure
  `<projet>/CLAUDE.md` + `<projet>/agents/<repo>.md`. Les règles qui le
  pilotent sont dans `config/CLAUDE.md` de ce repo : changer la structure
  là-bas impose de mettre à jour ces règles ici.
- Variables d'environnement lues par Claude dans le conteneur : contrat entre
  `.env.template`, `bin/claude-docker` et `config/CLAUDE.md`.

## Pièges connus

- Dans le conteneur, **seul le dossier de lancement est visible** : un repo
  voisin (`../claude-agents`) n'existe que s'il est monté par
  `bin/claude-docker` (entrée chemin local de `CLAUDE_AGENTS_REPOS`).
- `docker --env-file` ne retire **pas** les guillemets : valeurs sans
  guillemets ni espaces, listes séparées par des virgules.
- Pas de `docker` dans le conteneur : pour tester `bin/claude-docker`, mettre
  un faux `docker` en tête du `PATH` qui affiche ses arguments.
- Pas de `python3` dans l'image : éditions via Edit, `sed` ou heredocs.
- Pas d'identité git dans le conteneur : `git commit` échoue sans les
  variables `GIT_AUTHOR_*` / `GIT_COMMITTER_*` (via `.env` ou la commande).
- Clé SSH absente tant que `setup-ssh-key.sh` n'a pas été lancé **et** que
  claude-docker n'a pas été relancé : `git push` échoue alors avec
  « Host key verification failed ».
- `config/CLAUDE.md` est relu à chaque lancement (pas besoin de `--rebuild`),
  mais une session en cours garde l'ancienne version.
- GitHub n'accepte une même clé comme deploy key que sur un seul repo.
- Remote `origin` en SSH (`git@github.com:…`) : nécessaire pour la clé de Claude.
