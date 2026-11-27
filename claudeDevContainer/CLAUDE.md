# Projet claudeDevContainer

Mono-repo. Lanceur de Claude Code dans Docker, partagé publiquement ; les
données personnelles (agents, clés, `.env`) vivent ailleurs.

## Repos

| Chemin | Rôle | Agent |
|---|---|---|
| `.` (claudeDevContainer) | Image Docker, lanceur `claude-docker`, règles globales `config/CLAUDE.md` | `claudeDevContainer` |

## Liens

- **claude-agents** (ce repo d'agents) : `config/CLAUDE.md` de
  claudeDevContainer définit comment il est cloné, lu, mis à jour et publié.
  Toute évolution de la structure de claude-agents doit être répercutée dans
  ces règles, et inversement.
- Pas de déploiement : les utilisateurs font `git pull` puis relancent
  `claude-docker` (ou `--rebuild` si le `dockerfile` change).

## À savoir

- Push sur claudeDevContainer : jamais sans accord explicite.
- Tester depuis le conteneur reste limité : pas de `docker`, et seul le
  dossier de lancement est monté.
