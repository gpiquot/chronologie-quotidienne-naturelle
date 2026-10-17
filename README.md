# Chronologie quotidienne naturelle pour Immich

Patch de l'interface web Immich : les journées restent affichées de la plus récente à la plus ancienne, et les photos de chaque journée suivent l'ordre naturel du matin au soir.

Exemple : 17 octobre (08 h, 12 h, 19 h), puis 16 octobre (09 h, 15 h, 21 h).

Le patch s'applique uniquement à la timeline principale `/photos`. Il ne modifie pas le tri des applications mobiles. Les autres vues conservent leur logique de tri ; les tests visuels de non-régression doivent être refaits à chaque version.

## Indépendance de Presenter

Ce projet modifie le code web d'Immich. Il n'est pas un module de `immich-presenter-slideshow` et ne dépend pas de cette application. Le diaporama Presenter possède son propre ordre, défini dans son code. Timestamp Tools et les outils d'import sont également indépendants.

## Version disponible

| Immich | État |
| --- | --- |
| 3.3.3 | Patch appliqué et image construite puis déployée le 17 octobre 2026 ; service sain. Validation visuelle finale du tri par l'utilisateur à confirmer. |

Le comportement était utilisé sur Immich 3.1.0. Le patch 3.3.3 a été régénéré pour tenir compte du contexte modifié des sources officielles. Il touche quatre fichiers : types des options, gestionnaire de timeline, gestionnaire mensuel et page `/photos`.

## Construire sans modifier la production

Dans un dossier neuf où Git et Docker avec BuildKit sont disponibles :

```bash
cd immich/3.3.3
bash build-immich-3.3.3.sh
```

Le script récupère le tag officiel `v3.3.3` dans un nouveau dossier, vérifie et applique le patch, puis construit `immich-server:3.3.3-day-asc` pour `linux/amd64` avec `server/Dockerfile`. Il s'arrête si le dossier de construction ou cette image existent déjà. Il nécessite Internet et suffisamment d'espace disque. Il ne démarre aucun conteneur et ne modifie pas Compose.

## Déployer

Avant la bascule, sauvegarder la VM, PostgreSQL et les configurations ; sauvegarder séparément les médias sur les partages NAS. Prévoir une interruption pour le démarrage et les migrations.

Exemple de surcharge Compose :

```yaml
services:
  immich-server:
    image: immich-server:3.3.3-day-asc
    pull_policy: never
```

Mettre aussi `IMMICH_VERSION=v3.3.3` dans la configuration privée pour aligner le machine learning. Une surcharge qui fixe l'image serveur empêche une modification de cette variable seule de mettre à jour le serveur. Adapter la procédure aux fichiers Compose réellement utilisés. Ne pas modifier automatiquement PostgreSQL ou Valkey sans nécessité documentée.

Les rétrogradations Immich ne sont pas prises en charge : conserver une sauvegarde cohérente et prévoir sa restauration plutôt que relancer simplement l'ancienne image sur une base migrée.

## Tests à chaque version

1. Lire les notes de toutes les versions intermédiaires.
2. Vérifier le patch sur les sources exactes de la cible ; ne pas forcer son application en cas de conflit.
3. Construire l'image et conserver version, commit source, empreinte du patch et identifiant d'image.
4. Après bascule, vérifier santé des services et absence d'erreurs nouvelles.
5. Vérifier `/photos` : jours décroissants, heures croissantes par jour, chargement initial, défilement, insertion en direct, piles, navigation précédente/suivante.
6. Vérifier albums, favoris et autres vues sans option `dayOrder` explicite.

Une compilation réussie ne garantit pas le bon ordre visuel. Les tests de Presenter, si cette application est installée, relèvent de son dépôt distinct.

## Sources et licence amont

Ce dépôt conserve des modifications dérivées d'[Immich](https://github.com/immich-app/immich), dont le code est distribué sous [GNU AGPL v3](https://github.com/immich-app/immich/blob/v3.3.3/LICENSE). Respecter les obligations de cette licence lors de la redistribution des modifications et images.

Documentation officielle : [mise à jour](https://docs.immich.app/install/upgrading/) et [sauvegarde/restauration](https://docs.immich.app/administration/backup-and-restore/).

Ne publier aucun `.env` réel, clé API, jeton, lien de partage personnel, journal privé ou photographie.
