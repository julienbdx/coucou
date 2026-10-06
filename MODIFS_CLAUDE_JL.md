# Modifs locales JL — Claude Code sous Ghostty

> Notes personnelles de Julien, hors périmètre du dépôt amont.
> Patch écrit le 2026-10-06 sur `main` @ `8b3322b` (0.2.0 préparée). Non committé.

## Le problème

Coucou n'accepte les évènements de Claude Code que si le terminal hôte contient `vscode`
(`isVSCodeEditor` dans `HookServer.swift`) ou si c'est Cursor. Sous Ghostty (`TERM_PROGRAM=ghostty`),
tout est rejeté : `Ignored PreToolUse from ghostty (...)` dans `~/Library/Logs/NotchBuddy/nb.log`,
et les demandes de permission / `AskUserQuestion` répondent `{"permissionDecision":"ask"}`.

## Le patch

Les hooks Claude Code passent `--agent claude` à `nb-hook`, et l'app route `coucou_agent == "claude"`
vers la pill `integration_claude` quel que soit le terminal (Cursor garde la priorité). La pill est
aussi renommée « Claude Code » à l'affichage (l'ID `integration_claude` ne change pas).

Contenu : routage + 2 gardes dans `HookServer.swift`, les 2 installeurs de hooks (App Store / GitHub),
les 2 copies du `nb-hook.py` embarqué (la branche `--ask` ignorait `--agent`), puis le renommage
Mac et Windows. Il supprime aussi la carte de droite (liste des pills) de `OverviewView`, dans
`IslandViewContent.swift` : la carte de gauche passe de `.frame(width: 322)` à `.frame(maxWidth: .infinity)`
(`AgentPillsView` reste, inutilisé ; `docs/SPEC.md` et `windows/` ne sont pas mis à jour sur ce point). Dans la carte d'intégration (`IntegrationCardView`), le sous-titre « Connected · loading… » devient « Connected », et le bouton « Open Visual Studio Code » de la pill Claude Code est masqué (`if false && …`). Le diff contient aussi les 4 fichiers de renommage ; `Localizable.xcstrings` n'en fait pas partie.

```diff
diff --git a/NotchBuddy/Sources/App/HookServer.swift b/NotchBuddy/Sources/App/HookServer.swift
index 3219d0a..7372592 100644
--- a/NotchBuddy/Sources/App/HookServer.swift
+++ b/NotchBuddy/Sources/App/HookServer.swift
@@ -356,3 +356,3 @@ final class HookServer: @unchecked Sendable {
         // • Cursor bundle ID → agent_cursor
-        // • VS Code → integration_claude
+        // • VS Code, or --agent claude from any terminal → integration_claude
         #if !APPSTORE
@@ -373,3 +373,3 @@ final class HookServer: @unchecked Sendable {
             isExternalAgent = false
-        } else if isVSCodeEditor {
+        } else if isVSCodeEditor || rawAgent == "claude" {
             agentId = "integration_claude"
@@ -557,3 +557,4 @@ final class HookServer: @unchecked Sendable {
     /// Validates a coucou_agent name: lowercase, digits and hyphens, 1–24 chars.
-    /// "claude" is reserved and rejected so it cannot impersonate the Claude Code pill.
+    /// "claude" is reserved and rejected as an external pill: it is routed to the Claude Code
+    /// pill (integration_claude) by the host checks instead, whatever terminal runs the agent.
     /// Returns the name unchanged if valid, nil otherwise.
@@ -678,3 +679,3 @@ final class HookServer: @unchecked Sendable {
         }
-        guard isCodexRequest || isCopilotRequest || isMuseRequest || isCursorEditor || isVSCodeEditor else {
+        guard isCodexRequest || isCopilotRequest || isMuseRequest || isCursorEditor || isVSCodeEditor || rawAgent == "claude" else {
             Task.detached { [weak self] in
@@ -848,3 +849,3 @@ final class HookServer: @unchecked Sendable {
         }
-        guard isCodexRequest || isCursorEditor || isVSCodeEditor else {
+        guard isCodexRequest || isCursorEditor || isVSCodeEditor || rawAgent == "claude" else {
             Task.detached { [weak self] in
@@ -1233,3 +1234,3 @@ final class HookServer: @unchecked Sendable {
             existing.removeAll { ($0["hooks"] as? [[String: Any]])?.contains { ($0["command"] as? String)?.contains("NotchBuddy") == true || ($0["command"] as? String)?.contains("coucou") == true } ?? false }
-            existing.append(["hooks": [["type": "command", "command": quotedCmd, "timeout": timeout]]])
+            existing.append(["hooks": [["type": "command", "command": "\(quotedCmd) --agent claude", "timeout": timeout]]])
             hooks[event] = existing
@@ -1240,3 +1241,3 @@ final class HookServer: @unchecked Sendable {
             "matcher": "AskUserQuestion",
-            "hooks": [["type": "command", "command": "\(quotedCmd) --ask", "timeout": 130]],
+            "hooks": [["type": "command", "command": "\(quotedCmd) --ask --agent claude", "timeout": 130]],
         ])
@@ -1484,3 +1485,3 @@ final class HookServer: @unchecked Sendable {
             } ?? false }
-            existing.append(["hooks": [["type": "command", "command": quotedCmd, "timeout": timeout]]])
+            existing.append(["hooks": [["type": "command", "command": "\(quotedCmd) --agent claude", "timeout": timeout]]])
             hooks[event] = existing
@@ -1491,3 +1492,3 @@ final class HookServer: @unchecked Sendable {
             "matcher": "AskUserQuestion",
-            "hooks": [["type": "command", "command": "\(quotedCmd) --ask", "timeout": 130]],
+            "hooks": [["type": "command", "command": "\(quotedCmd) --ask --agent claude", "timeout": 130]],
         ])
@@ -2554,2 +2555,5 @@ def main():
         payload['coucou_kind'] = 'ask_user_question'
+        _args = sys.argv[1:]
+        if '--agent' in _args and _args.index('--agent') + 1 < len(_args):
+            payload.setdefault('coucou_agent', _args[_args.index('--agent') + 1])
         env = os.environ
@@ -2846,2 +2850,5 @@ def main():
         payload['coucou_kind'] = 'ask_user_question'
+        _args = sys.argv[1:]
+        if '--agent' in _args and _args.index('--agent') + 1 < len(_args):
+            payload.setdefault('coucou_agent', _args[_args.index('--agent') + 1])
         env = os.environ
diff --git a/NotchBuddy/Sources/App/IslandViewContent.swift b/NotchBuddy/Sources/App/IslandViewContent.swift
index fe623cf..5bddd15 100644
--- a/NotchBuddy/Sources/App/IslandViewContent.swift
+++ b/NotchBuddy/Sources/App/IslandViewContent.swift
@@ -141,8 +141,3 @@ struct OverviewView: View {
             }
-            .frame(width: 322)
-
-            // Right card: agent pills
-            CardBackground(wash: nil) {
-                AgentPillsView(state: state)
-            }
+            .frame(maxWidth: .infinity)
         }
@@ -1787,3 +1782,3 @@ struct IntegrationCardView: View {
             }
-            return String(localized: "Connected · loading…")
+            return String(localized: "Connected")
         } else {
@@ -1921,3 +1916,3 @@ struct IntegrationCardView: View {
                 HStack(spacing: 8) {
-                    if task.id == "integration_claude" {
+                    if false && task.id == "integration_claude" {
                         Button("Open Visual Studio Code") { openVSCode() }
@@ -3738,5 +3733,5 @@ struct AgentPill: View {
 
-    // VS Code pill always shows "VS Code" label regardless of active project name
+    // Claude Code pill always shows "Claude Code" label regardless of active project name
     private var displayName: String {
-        task.id == "integration_claude" ? "VS Code" : task.name
+        task.id == "integration_claude" ? "Claude Code" : task.name
     }
diff --git a/NotchBuddy/Sources/CoucouKit/MochiActivityState.swift b/NotchBuddy/Sources/CoucouKit/MochiActivityState.swift
index 4bb5130..cc10e37 100644
--- a/NotchBuddy/Sources/CoucouKit/MochiActivityState.swift
+++ b/NotchBuddy/Sources/CoucouKit/MochiActivityState.swift
@@ -65,3 +65,3 @@ struct MochiActivityState: Codable, Hashable, Sendable {
     static let placeholder = MochiActivityState(
-        pillId: "integration_claude", agent: "VS Code", color: "#4A86E8",
+        pillId: "integration_claude", agent: "Claude Code", color: "#4A86E8",
         state: "working", statusText: "working · 3/7", tone: "working",
diff --git a/NotchBuddy/Sources/CoucouKit/PillCatalog.swift b/NotchBuddy/Sources/CoucouKit/PillCatalog.swift
index 3f61225..7cf9a05 100644
--- a/NotchBuddy/Sources/CoucouKit/PillCatalog.swift
+++ b/NotchBuddy/Sources/CoucouKit/PillCatalog.swift
@@ -50,3 +50,3 @@ enum PillCatalog {
         // ── Where you code ───────────────────────────────────────────────────
-        .init(id: "integration_claude",  name: "VS Code",     color: "#F5F6F8",
+        .init(id: "integration_claude",  name: "Claude Code", color: "#F5F6F8",
               category: .workspace, subtitle: "Integration",  source: .claudeCode),
diff --git a/NotchBuddy/Sources/Widgets/CoucouWidgets.swift b/NotchBuddy/Sources/Widgets/CoucouWidgets.swift
index 4491b2d..d87d7ad 100644
--- a/NotchBuddy/Sources/Widgets/CoucouWidgets.swift
+++ b/NotchBuddy/Sources/Widgets/CoucouWidgets.swift
@@ -391,3 +391,3 @@ extension SharedSession {
     static let samples: [SharedSession] = [
-        SharedSession(id: "integration_claude", title: "coucou", agent: "VS Code", color: "#4A86E8",
+        SharedSession(id: "integration_claude", title: "coucou", agent: "Claude Code", color: "#4A86E8",
                       state: "approval", statusText: "waiting for your OK", tone: .waiting, urgency: 0,
diff --git a/windows/src/core/state.ts b/windows/src/core/state.ts
index 01236b8..8115952 100644
--- a/windows/src/core/state.ts
+++ b/windows/src/core/state.ts
@@ -60,3 +60,3 @@ const task = (
 export const INTEGRATION_AGENTS: AgentTask[] = [
-  task("integration_claude", "VS Code", "#F5F6F8", "claudeCode"),
+  task("integration_claude", "Claude Code", "#F5F6F8", "claudeCode"),
   task("integration_resend", "Resend", "#22C55E", "n8n"),
diff --git a/windows/src/island/hooks.ts b/windows/src/island/hooks.ts
index d90a78f..73a96e4 100644
--- a/windows/src/island/hooks.ts
+++ b/windows/src/island/hooks.ts
@@ -134,3 +134,3 @@ function clearSession() {
   t.stepIndex = 0;
-  t.name = "VS Code";
+  t.name = "Claude Code";
   t.pillBadge = null;
diff --git a/windows/src/views/integrations.ts b/windows/src/views/integrations.ts
index b8ad73c..848a2d9 100644
--- a/windows/src/views/integrations.ts
+++ b/windows/src/views/integrations.ts
@@ -112,3 +112,3 @@ function idleCard(task: AgentTask, openSettings: () => void): HTMLElement {
     { class: "int-card" },
-    header(task.color, task.id === "integration_claude" ? "VS Code" : task.name, "Integration"),
+    header(task.color, task.id === "integration_claude" ? "Claude Code" : task.name, "Integration"),
     h("div", { class: "int-status" }, dot(statusColor, 5), h("span", { text: label })),
diff --git a/windows/src/views/views.ts b/windows/src/views/views.ts
index ac0ac7b..1c53c17 100644
--- a/windows/src/views/views.ts
+++ b/windows/src/views/views.ts
@@ -229,3 +229,3 @@ function buildOverview(actions: ViewActions): ViewHost {
 function buildPill(task: AgentTask, actions: ViewActions): HTMLElement {
-  const label = task.id === "integration_claude" ? "VS Code" : task.name;
+  const label = task.id === "integration_claude" ? "Claude Code" : task.name;
   const canvas = createMiniBot(task, 24);
```

## Réappliquer sur la prochaine version

```sh
cd /Users/julien/Developpements/coucou
git pull
awk '/^```diff/{f=1;next} /^```$/{f=0} f' MODIFS_CLAUDE_JL.md > /tmp/ghostty.patch
git apply --3way /tmp/ghostty.patch
```

Si ça échoue, refaire à la main dans `NotchBuddy/Sources/App/HookServer.swift` (chercher les
motifs, les numéros de ligne bougent) :

1. `processEvent` : `else if isVSCodeEditor {` → `else if isVSCodeEditor || rawAgent == "claude" {`
2. Garde permission : ajouter `|| rawAgent == "claude"` au `guard isCodexRequest || isCopilotRequest …`
3. Garde `AskUserQuestion` : ajouter `|| rawAgent == "claude"` au `guard isCodexRequest || isCursorEditor …`
4. Les 2 installeurs : commande des évènements `\(quotedCmd) --agent claude`, et `\(quotedCmd) --ask --agent claude`
5. Les 2 copies Python (`def main():`, après `payload['coucou_kind'] = 'ask_user_question'`) : lire `--agent` dans `sys.argv` et faire `payload.setdefault('coucou_agent', …)`
6. `OverviewView` : supprimer le bloc `CardBackground(wash: nil) { AgentPillsView(state: state) }` de droite et remplacer `.frame(width: 322)` par `.frame(maxWidth: .infinity)` sur la carte de gauche.
7. Renommage : `PillCatalog.swift` (`name: "Claude Code"`), le libellé `displayName` d'`IslandViewContent.swift`, et les chaînes `"VS Code"` restantes liées à `integration_claude` (`MochiActivityState.swift`, `CoucouWidgets.swift`, `windows/src/**`).
8. `IntegrationCardView` : `String(localized: "Connected · loading…")` → `String(localized: "Connected")`, et `if task.id == "integration_claude" {` (bouton « Open Visual Studio Code ») → `if false && task.id == "integration_claude" {`.

Avant d'appliquer, vérifier que l'amont n'a pas déjà traité le sujet :
`grep -n 'rawAgent == "claude"\|raw != "claude"' NotchBuddy/Sources/App/HookServer.swift`.

## Construire, installer

```sh
cd NotchBuddy && export DEVELOPER_DIR=/Applications/Xcode-beta.app/Contents/Developer
xcodegen && xcodebuild -scheme NotchBuddy -configuration Debug build > /tmp/xb.txt 2>&1; tail -3 /tmp/xb.txt
```

`xcode-select` pointe sur les Command Line Tools, d'où `DEVELOPER_DIR`. Dernier build : réussi (2026-10-06).

**Obligatoire après installation** : Settings → Agents → « Claude Code Hooks » pour réinstaller. Les
hooks déjà présents dans `~/.claude/settings.json` n'ont pas `--agent claude`, et `hooksNeedUpdate()`
ne le détecte pas. La réinstallation réécrit aussi `~/Library/Application Support/NotchBuddy/nb-hook.py`
(donc plus besoin de l'ancien patch `term_program` / `~/nb-hook.py.ghostty-patch`).

## Vérifier

```sh
tail -f ~/Library/Logs/NotchBuddy/nb.log
```

Lancer une commande dans Claude Code sous Ghostty. Attendu : `PreToolUse Bash`. Si on lit encore
`Ignored PreToolUse from ghostty`, les hooks n'ont pas été réinstallés. Tester ensuite une
`AskUserQuestion` : le chemin `--ask` n'écrit rien dans `nb.log`, c'est normal.

## Pièges

- Le wrapper `nb-hook` (shell) ne fait rien, silencieusement, si `xcode-select -p` échoue (il utilise
  le `/usr/bin/python3` des Command Line Tools). Un log muet peut venir de là.
- Pas de test dans `tests/` : `HookServer` est privé et les tests existants sont des scripts autonomes.
- **Non validé de bout en bout** : le build passe et `AskUserQuestion` marchait, mais avec l'ancien
  patch `term_program` encore actif. À confirmer avec les hooks réinstallés et ce patch seul.
