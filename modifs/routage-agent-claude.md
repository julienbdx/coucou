# Routage `--agent claude` vers la pill « Claude Code »

← [MES_MODIFS.md](../MES_MODIFS.md)

Commit d'origine : `1a5d918` (branche `claude-agent-routing`). Écrit sur `main` @ `8b3322b` (0.2.0 préparée).

## Ce que ça résout

Coucou n'accepte les évènements de Claude Code que si le terminal hôte est un VS Code
(`isVSCodeEditor` dans `HookServer.swift`) ou Cursor. Sous Ghostty (`TERM_PROGRAM=ghostty`),
tout est rejeté :

- `~/Library/Logs/NotchBuddy/nb.log` : `Ignored PreToolUse from ghostty (...)`
- les demandes de permission et les `AskUserQuestion` répondent `{"permissionDecision":"ask"}`,
  donc Claude Code retombe sur sa propre invite dans le terminal et rien n'apparaît dans le notch.

## Ce que fait la modif

1. Les hooks Claude Code installés par Coucou passent `--agent claude` à `nb-hook`
   (et `--ask --agent claude` pour le hook dédié `AskUserQuestion`).
2. L'app route `coucou_agent == "claude"` vers la pill `integration_claude`, quel que soit le
   terminal. Cursor garde la priorité.
3. La pill s'affiche « Claude Code » au lieu de « VS Code ». L'ID `integration_claude` ne change
   **pas** (contrat stable : Keychain, UserDefaults, routage des hooks).

## Fichiers touchés

| Fichier | Changement |
|---|---|
| `NotchBuddy/Sources/App/HookServer.swift` | routage `\|\| rawAgent == "claude"`, 2 gardes `guard`, 2 installeurs de hooks, 2 copies du `nb-hook.py` embarqué |
| `NotchBuddy/Sources/App/IslandViewContent.swift` | `AgentPill.displayName` : « Claude Code » |
| `NotchBuddy/Sources/CoucouKit/PillCatalog.swift` | `name: "Claude Code"` |
| `NotchBuddy/Sources/CoucouKit/MochiActivityState.swift` | `placeholder.agent` |
| `NotchBuddy/Sources/Widgets/CoucouWidgets.swift` | `SharedSession.samples` |
| `windows/src/core/state.ts`, `island/hooks.ts`, `views/integrations.ts`, `views/views.ts` | renommage côté Windows/Linux |

## Réappliquer sur une version plus récente

Le plus simple : `git cherry-pick 1a5d918` (ou `git diff 1a5d918^ 1a5d918 | git apply --3way`).
Si ça ne passe pas, refaire à la main, dans cet ordre :

1. **`HookServer.swift`, routage** — repérer le bloc de choix d'`agentId` (commentaire
   « Cursor bundle ID → agent_cursor / VS Code → integration_claude »). Remplacer
   `else if isVSCodeEditor` par `else if isVSCodeEditor || rawAgent == "claude"`.
2. **`HookServer.swift`, gardes** — chercher `else { ... "permissionDecision":"ask" }` précédé de
   `guard isCodexRequest || ... || isVSCodeEditor` (il y en a 2 : permission et AskUserQuestion).
   Ajouter `|| rawAgent == "claude"` à chaque `guard`.
3. **`HookServer.swift`, installeurs de hooks** — dans les 2 installeurs (App Store et GitHub), la
   commande ajoutée à `settings.json` devient `"\(quotedCmd) --agent claude"`, et celle du hook
   `AskUserQuestion` `"\(quotedCmd) --ask --agent claude"`.
4. **`nb-hook.py` embarqué** (2 copies dans `HookServer.swift`, branche `--ask`) — après
   `payload['coucou_kind'] = 'ask_user_question'`, ajouter la lecture de `--agent` dans `sys.argv`
   et `payload.setdefault('coucou_agent', ...)`. Sans ça la branche `--ask` ignore `--agent`.
5. **Ne pas toucher à `validateAgent`** : `"claude"` doit rester refusé comme pill externe. Seul le
   commentaire est mis à jour.
6. **Renommage** — remplacer le libellé « VS Code » par « Claude Code » aux 4 endroits Swift
   (`displayName`, `PillCatalog`, `placeholder`, `samples`) et aux 4 endroits `windows/`.
   `grep -rn '"VS Code"' NotchBuddy windows/src` donne la liste.

Si l'amont a entre-temps ajouté un vrai support de `--agent claude` ou de Ghostty, la modif
devient inutile : vérifier d'abord avec `grep -n 'rawAgent' NotchBuddy/Sources/App/HookServer.swift`.

## Après réapplication

- Réinstaller les hooks depuis Coucou (les anciens n'ont pas `--agent claude`). Ne jamais écraser
  `~/.claude/settings.json` à la main : passer par l'installeur (sauvegarde datée + diff).
- Vérifier sous Ghostty : lancer une commande demandant une permission → la carte apparaît dans le
  notch, et `nb.log` n'affiche plus `Ignored PreToolUse from ghostty`.
