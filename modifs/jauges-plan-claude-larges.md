# Jauges « 5 hours » et « week » plus larges

← [MES_MODIFS.md](../MES_MODIFS.md)

Commit d'origine : `57108f2` (branche `claude-agent-routing`). Écrit sur `main` @ `8b3322b`.

## Ce que ça résout

Dans la carte du plan Claude (`ClaudePlanCardView`), les barres de jauge « 5 hours » et « week »
faisaient 50 pt de large : trop petites pour lire l'avancement d'un coup d'œil.

## Ce que fait la modif

Dans `NotchBuddy/Sources/App/ClaudePlanCardView.swift`, `GaugeRowView` :

- le libellé passe de `.frame(width: 40, …)` à `.frame(width: 52, …)` ;
- la barre passe de 50 pt à 90 pt, aux **quatre** endroits qui portent la largeur : le `Capsule`
  de fond, le `Capsule` de remplissage (`max(0, 90 * CGFloat(pct / 100))`), le `.frame` du
  `ZStack`, et le commentaire « Fixed-width bar (~90pt) ».

## Réappliquer sur une version plus récente

`git cherry-pick <commit de cette modif>` suffit en général. Sinon, à la main :

1. Ouvrir `ClaudePlanCardView.swift`, trouver `GaugeRowView` (`grep -n 'GaugeRowView'`).
2. Libellé : `width: 40` → `width: 52`.
3. Barre : remplacer **toutes** les occurrences de `50` liées à la barre par `90` (fond, remplissage
   proportionnel, cadre du `ZStack`). Si on en oublie une, la jauge se tronque ou déborde.
4. Vérifier que la carte reste dans la largeur disponible de l'île (ouvrir la carte du plan
   Claude dans le notch).

Si l'amont a rendu la barre flexible (largeur dynamique), cette modif n'est plus nécessaire.
