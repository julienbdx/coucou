# Accueil du notch : buddy 75 %, pills 25 % sur une colonne

← [MES_MODIFS.md](../MES_MODIFS.md)

Écrit sur `main` @ `20e1f8b` (0.2.0). Indépendant des autres modifs.

## Ce que ça résout

L'accueil (`OverviewView`) se partageait environ 50/50 : à gauche la carte du buddy et du ticker
(largeur fixe de 322 pt), à droite la carte des pills en 2 colonnes. Le buddy et le ticker manquaient
de place. Maintenant la carte du buddy prend 75 % de la largeur et les pills 25 %, sur une colonne.

## Ce que fait la modif

Dans `NotchBuddy/Sources/App/IslandViewContent.swift` :

1. `OverviewView.body` : le `HStack(spacing: 10)` est enveloppé dans un
   `GeometryReader { geo in … }`. `available = geo.size.width - 10` (10 = espacement du `HStack`).
   Le `HStack` reçoit `.frame(width: geo.size.width, height: geo.size.height)`.
2. Carte de gauche : `.frame(width: 322)` devient `.frame(width: available * 0.75)`.
3. Carte de droite (`CardBackground { AgentPillsView(state:) }`) : ajout de
   `.frame(width: available * 0.25)`.
4. `AgentPillsView.columns` : une seule `GridItem(.flexible(), spacing: 4)` au lieu de deux.

Les `.onChange` / `.onReceive` qui suivaient le `HStack` s'appliquent maintenant au `GeometryReader`.

## À vérifier après réapplication

- 4 pills empilées (`displayTasks` = `prefix(4)`, 28 pt chacune + 4 pt d'espacement) tiennent dans la
  hauteur de l'île (≈ 150 pt).
- Les noms de pills (« Claude Code ») ne sont pas tronqués dans la colonne d'environ 150 pt.

## Réappliquer sur une version plus récente

`git cherry-pick <commit de cette modif>` suffit en général. Sinon, à la main :

1. `grep -n 'struct OverviewView' NotchBuddy/Sources/App/IslandViewContent.swift`, puis repérer le
   `HStack(spacing: 10)` racine de `body` et la carte de gauche terminée par `.frame(width: …)`.
2. Envelopper le `HStack` dans le `GeometryReader` et remplacer les largeurs comme ci-dessus. Garder
   les `.onChange` / `.onReceive` après la fermeture du `GeometryReader`.
3. `grep -n 'struct AgentPillsView'` : réduire `columns` à une seule `GridItem`.

Si l'amont a changé la structure (carte de droite supprimée, largeur déjà proportionnelle, nombre de
pills affichées), adapter les proportions plutôt que de forcer le diff. Si la largeur de l'île n'est
plus 640 pt (`IslandConst.expandedWidth`), rien à changer : les largeurs sont relatives.
