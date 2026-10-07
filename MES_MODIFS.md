# Mes modifs locales

Modifs perso par-dessus Coucou, hors périmètre de l'amont. Chaque modif = un commit dédié et une
fiche détaillée dans [`modifs/`](modifs/) (problème résolu, fichiers, comment la réappliquer sur une
version plus récente).

## Ajouter une modif

Une modif = un commit sur `version-perso`, une ligne dans le tableau ci-dessous et une fiche dans
`modifs/` (problème résolu, fichiers touchés, comment la réappliquer sur une version plus récente).

Le commit (code + fiche + ligne du tableau ensemble) doit **finir par** la ligne :

```
Documented in MES_MODIFS.md and modifs/<fiche>.md
```

Cette ligne est le filtre de `/coucou-build` : il rejoue sur `X.Y.Z-perso` les commits de
`version-perso` dont le message contient `Documented in MES_MODIFS.md`, dans l'ordre. Sans elle, le
commit est ignoré au prochain build. Inversement, un commit qui n'est pas une modif (comme celui
qui a ajouté ce paragraphe) ne doit pas la contenir.

Ajouter la ligne du tableau **à la fin** du tableau : les commits sont rejoués dans l'ordre et
chaque ligne se greffe sur la précédente.

| Modif | Description | Fiche |
|---|---|---|
| Routage `--agent claude` | Claude Code fonctionne depuis n'importe quel terminal (Ghostty…) et la pill s'appelle « Claude Code ». | [modifs/routage-agent-claude.md](modifs/routage-agent-claude.md) |
| Carte d'intégration épurée | Sous-titre « Connected » sans « loading… » et bouton « Open Visual Studio Code » masqué. | [modifs/carte-integration-epuree.md](modifs/carte-integration-epuree.md) |
| Jauges du plan Claude plus larges | Barres « 5 hours » et « week » de 50 à 90 pt (libellé de 40 à 52 pt). | [modifs/jauges-plan-claude-larges.md](modifs/jauges-plan-claude-larges.md) |
