# Carte d'intégration épurée (sous-titre et bouton VS Code)

← [MES_MODIFS.md](../MES_MODIFS.md)

Commit d'origine : `ae7a86e` (branche `claude-agent-routing`). Écrit sur `main` @ `8b3322b`.
Dépend de [routage-agent-claude.md](routage-agent-claude.md) (la pill s'appelle « Claude Code »).

## Ce que ça résout

Dans la carte d'intégration (`IntegrationCardView`) :

- le sous-titre « Connected · loading… » reste affiché tel quel tant qu'aucune info de modèle n'est
  disponible, ce qui donne l'impression d'un chargement qui ne finit jamais ;
- le bouton « Open Visual Studio Code » de la pill Claude Code n'a plus de sens : on utilise
  Claude Code depuis un terminal (Ghostty), pas depuis VS Code.

## Ce que fait la modif

Dans `NotchBuddy/Sources/App/IslandViewContent.swift`, `IntegrationCardView` :

1. `String(localized: "Connected · loading…")` devient `String(localized: "Connected")`.
2. `if task.id == "integration_claude" {` devient `if false && task.id == "integration_claude" {`
   devant le bouton « Open Visual Studio Code ». Le code reste en place, simplement désactivé
   (diff minimal, facile à retirer).

La clé `"Connected"` existe déjà dans `Localizable.xcstrings`, aucune traduction à ajouter.

## Réappliquer sur une version plus récente

`git cherry-pick <commit de cette modif>` suffit en général. Sinon, à la main :

1. `grep -n 'Connected · loading' NotchBuddy/Sources/App/IslandViewContent.swift` → remplacer la
   chaîne par `"Connected"`.
2. `grep -n 'Open Visual Studio Code' NotchBuddy/Sources/App/IslandViewContent.swift` → sur le
   `if task.id == "integration_claude"` qui précède le bouton, préfixer par `false && `.
3. Vérifier que `"Connected"` est bien dans `Localizable.xcstrings` (`grep -c '"Connected"'`).

Si l'amont a changé ces textes, les fonctions à regarder sont le calcul du sous-titre de statut
(`return String(localized: "Key configured · \(model)")` juste au-dessus) et le `HStack` des boutons
d'action en bas de la carte.
