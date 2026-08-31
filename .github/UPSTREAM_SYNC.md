# Pipeline de mise à jour automatique

```
phpmyadmin/phpmyadmin (upstream)
        │  upstream-sync.yml — tous les jours 06h00 UTC (PR + auto-merge)
        ▼
      master            ← miroir d'upstream + les workflows de sync, rien d'autre
        │  sync-master.yml — tous les jours 06h30 UTC (merge direct)
        ▼
  design-yorkhost       ← branche de déploiement : thème YorkHost + patchs locaux
```

- **`master`** ne porte aucune personnalisation : uniquement le code upstream
  plus `upstream-sync.yml`/`sync-master.yml` et ce document. Les workflows
  planifiés ne tournent que depuis la branche par défaut, d'où leur présence ici.
- **`design-yorkhost`** est la seule branche où l'on committe : thème
  `public/themes/yorkhost/`, patchs (login, session), CI de build.

## Workflows

### `upstream-sync.yml` (06h00 UTC, et manuel via *Actions*)
Récupère `phpmyadmin/phpmyadmin` master ; s'il y a du nouveau, pousse
`upstream-sync`, ouvre une PR vers `master` et l'auto-merge (repli : merge
direct). Conflit → la PR reste ouverte.

### `sync-master.yml` (06h30 UTC, et manuel)
Merge `master` dans `design-yorkhost` et pousse. Les conflits sur des fichiers
`.github/workflows/` supprimés de `design-yorkhost` sont résolus
automatiquement (la suppression est conservée). Tout autre conflit → PR
`master` → `design-yorkhost` pour résolution manuelle.

## Configuration (une fois, dans les réglages GitHub)

- **Settings → General → Allow auto-merge** : pour que les PR de sync attendent
  les checks CI avant de fusionner.
- **Identité pour créer les PR** — les PR créées avec le `GITHUB_TOKEN` par
  défaut **ne déclenchent pas les workflows CI** (limitation GitHub). Par ordre
  de préférence :
  1. **GitHub App (recommandé, pas d'expiration)** : créer une App (*Settings →
     Developer settings → GitHub Apps*) avec permissions *Contents: Read &
     write* et *Pull requests: Read & write*, l'installer sur ce dépôt, puis
     renseigner la variable d'actions `SYNC_APP_ID` et le secret
     `SYNC_APP_PRIVATE_KEY` (clé privée `.pem`). Un token éphémère est généré à
     chaque exécution.
  2. **PAT** : secret `SYNC_TOKEN` (fine-grained, scopé à ce dépôt, contents +
     pull-requests en écriture). Inconvénient : il expire.
- **Settings → Advanced Security** : activer *Dependabot alerts* et *Dependabot
  security updates* pour les CVE des dépendances (composer/yarn).

## Conflits et personnalisations

Pour limiter les conflits lors des merges de `master`, garder les
personnalisations dans des fichiers *nouveaux* (le thème
`public/themes/yorkhost/` en est l'exemple) plutôt que de modifier les fichiers
upstream. Les patchs sur fichiers upstream (ex. `login/header.twig`) peuvent
occasionnellement demander une résolution manuelle via la PR ouverte par
`sync-master.yml`.
