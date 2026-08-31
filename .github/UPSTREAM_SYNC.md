# Synchronisation automatique avec phpMyAdmin upstream

Le workflow [`upstream-sync.yml`](workflows/upstream-sync.yml) tourne tous les jours à 06h00 UTC
(et manuellement via l'onglet *Actions* → *Sync upstream phpMyAdmin* → *Run workflow*).

## Ce qu'il fait

1. Récupère la branche `master` de `phpmyadmin/phpmyadmin` (upstream).
2. S'il y a de nouveaux commits, pousse la branche `upstream-sync` sur ce fork.
3. Ouvre (ou réutilise) une PR `upstream-sync` → `master`.
4. Active l'auto-merge sur la PR ; si l'auto-merge est désactivé au niveau du dépôt,
   il tente un merge direct. En cas de conflit, la PR reste ouverte pour résolution manuelle.

## Configuration recommandée (une fois, dans les réglages GitHub)

- **Settings → General → Allow auto-merge** : à cocher pour que les PR attendent
  les checks CI avant de fusionner.
- **Branch protection sur `master`** avec des *required status checks* : sans ça,
  l'auto-merge fusionne dès que la PR est propre, sans attendre la CI.
- **Secret `SYNC_TOKEN`** (optionnel mais conseillé) : un Personal Access Token
  (scope `repo` / contents + pull-requests en écriture). Les PR créées avec le
  `GITHUB_TOKEN` par défaut **ne déclenchent pas les workflows CI** (limitation
  GitHub) ; avec un PAT dans `SYNC_TOKEN`, la CI se lance normalement sur les PR
  de synchronisation.

## Conflits et personnalisations

Pour que les mises à jour upstream fusionnent sans conflit, garder les
personnalisations (thème, design) dans des fichiers *nouveaux* (par exemple un
thème dédié sous `public/themes/<nom>/`) plutôt que de modifier les fichiers
upstream existants.
