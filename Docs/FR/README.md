# Composer des terminaux avec GFX.UI.Terminal

`GFX.UI.Terminal` étend `GFX.UI` avec deux intentions : `Terminal.Shell()`
communique avec le système d’exploitation, tandis que `Terminal.Console()`
exécute des commandes possédées par l’application.

Les deux sont des `UI.CustomView`. Ils participent à la composition, aux
contraintes, au focus, au routage clavier/texte et à l’invalidation sans placer
leur état dans un monde ECS. Le package ne dépend ni de `GFX.ECS`, ni de
`GFX.Scene2D`, ni de `GFX.Rendering`.

[Read this documentation in English.](../EN/README.md)

## Installer le package

```text
silex install GFX.UI.Terminal
```

GFX.UI.Terminal demande Silex 0.39.0 ou une version plus récente.

## Ouvrir un terminal système

```sx
use GFX.UI
use GFX.UI.Terminal
use GFX.Viewer

var terminal = Terminal.Shell()
Viewer.show(UI.Session(terminal, autofocus:true))
```

`Terminal.Shell` possède par défaut le processus et son cycle de vie. Son
transport PTY ou ConPTY, les écritures, le polling non bloquant et le nettoyage
restent internes. `UI.Runtime.update(delta)` draine la sortie quand il visite
le contenu; aucune boucle de processus ni système ECS n’est requis.

`autofocus:true` donne immédiatement le focus au premier contrôle de saisie.
`Ctrl+C` est envoyé au shell par défaut. Une fenêtre à session unique peut
choisir `close_on_interrupt:true`, qui ferme uniquement le `ShellSession`.

## Déclarer une console applicative

`Terminal.Console` ne possède aucun processus et n’accorde aucun accès shell.
L’application déclare ses commandes :

```sx
use GFX.UI
use GFX.UI.Terminal

func status(arguments:str) str { return "application is running" }
func echo(arguments:str) str { return arguments }

var console = Terminal.Console(
    commands:[
        Terminal.Command("status", status, description:"Show application status"),
        Terminal.Command("echo", echo, description:"Print the provided text"),
    ],
    prompt:"silex> ",
    welcome:["type 'help' to list commands"],
)
var session = UI.Session(console)
```

`Terminal.Command` accepte un nom d’un mot, une action
`func(arguments:str) str` et une description optionnelle. `help` et `clear`
sont intégrées. Les doublons sont rejetés, un nom inconnu produit un diagnostic
local et une réponse vide restaure simplement le prompt.

## Posséder plusieurs sessions

Une application à onglets peut rendre le cycle de vie explicite :

```sx
var process = Terminal.ShellSession()
var terminal = Terminal.Shell(
    session:process,
    on_exit:func() {
        // Remove the corresponding tab here.
    },
)

match process.close() {
    failure(error) => { panic(error.operation + ": " + error.detail) }
    success => {}
}
```

`ShellSession.close()` termine uniquement ce shell. `on_exit` s’exécute une
fois après l’observation de la sortie, qu’elle soit demandée ou initiée par le
shell. La vue ignore les événements tardifs au lieu de les transformer en
erreurs de pipe fermé. L’application décide séparément de retirer l’onglet ou
de fermer sa fenêtre.

## Comprendre le rendu et l’émulation

Les deux contrôles partagent `Screen`, `Theme`, `ViewSettings` et le renderer
Canvas. Ils emploient `Canvas.Font.monospace()` par défaut. Les contraintes du
parent déterminent le cadre alloué et la peinture se limite aux cellules
visibles.

Les cellules adjacentes de même couleur et graisse sont réunies en runs Canvas
à avance fixe. Les positions de colonnes restent exactes et le nombre de
textures texte et de dessins Scene2D dépend des changements de style, pas du
nombre de caractères.

L’émulateur reconnaît UTF-8 imprimable, retour chariot, saut de ligne,
backspace, tabulation, mouvements et positionnement CSI, effacement de l’écran
ou de la ligne et les 16 couleurs ANSI. Les séquences inconnues sont ignorées.
Écran alternatif, régions de scroll, liens OSC, sélection, presse-papiers,
protocoles souris et largeur des graphèmes restent de futures extensions.

Les applications [DedicatedSystemTerminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/DedicatedSystemTerminal.sx),
[ApplicationCommandConsole](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ApplicationCommandConsole.sx)
et [ComposedSystemTerminal](https://github.com/Matanek/Silex-Examples/blob/main/Sources/ComposedSystemTerminal.sx)
appartiennent à Silex-Examples. Le benchmark
[TerminalScreenRendering](https://github.com/Matanek/Silex-Benchmarks/blob/main/Sources/TerminalScreenRendering.sx)
appartient à Silex-Benchmarks.
