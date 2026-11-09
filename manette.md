# Ta manette

## Ta manette @showdialog

Ton micro:bit devient la manette de ton jeu d'arcade. Tu l'inclines pour monter et descendre, tu appuies sur A pour tirer.

Et il réagit au jeu : un son et une icône quand tu détruis un ennemi ou quand tu es touché, et tes vies qui s'affichent.

## De la place pour recevoir

Le jeu va envoyer des mots à ta manette. Réserve-leur de la place.

- :mouse pointer: Clique sur **Avancé**, puis sur ``||serial:Communication série||``.
- :paper plane: Glisse ``||serial:série définir taille tampon RX à 128||`` dans ``||basic(noclick):au démarrage||``.

Sans cette réserve, les mots qui arrivent pendant qu'une icône s'affiche se perdent.

```blocks
// @highlight
serial.setRxBufferSize(128)
```

## Monter et descendre

Incline ta manette vers l'avant pour monter, vers l'arrière pour descendre.

- :mouse pointer: Dans le ``||basic(noclick):toujours||`` déjà présent, mets un ``||logic:si alors sinon si alors||`` de ``||logic:Logique||``.
- :mouse pointer: Conditions : ``||input:accélération (mg) y||`` de ``||input:Entrées||``, plus petite que **-300**, puis plus grande que **300**.
- :mouse pointer: Dans chaque branche, ``||serial:série écrire ligne||`` avec **haut**, puis **bas**.

-300 et 300 sont des seuils : trouve les tiens si ta manette réagit trop tôt ou trop tard.

```blockconfig.local
serial.writeLine("haut")
```

```blocks
serial.setRxBufferSize(128)
// @highlight
basic.forever(function () {
    if (input.acceleration(Dimension.Y) < -300) {
        serial.writeLine("haut")
    } else if (input.acceleration(Dimension.Y) > 300) {
        serial.writeLine("bas")
    }
})
```

## Tirer

Le bouton A de ta manette tire.

- :mouse pointer: Dans le même ``||basic:toujours||``, sous le premier **si**, ajoute ``||logic:si alors||``.
- :mouse pointer: Condition : ``||input:bouton A est pressé||``. Dans la branche : ``||serial:série écrire ligne||`` avec **A**.
- :mouse pointer: À la fin du **toujours**, ajoute ``||basic:pause (ms) 20||``.

Tant que tu tiens le bouton, la manette répète **A** toutes les 20 ms.

```blockconfig.local
serial.writeLine("A")
basic.pause(20)
```

```blocks
serial.setRxBufferSize(128)
basic.forever(function () {
    if (input.acceleration(Dimension.Y) < -300) {
        serial.writeLine("haut")
    } else if (input.acceleration(Dimension.Y) > 300) {
        serial.writeLine("bas")
    }
    // @highlight
    if (input.buttonIsPressed(Button.A)) {
        serial.writeLine("A")
    }
    // @highlight
    basic.pause(20)
})
```

## Écouter le jeu

Ta manette lit les mots que le jeu lui envoie, un par ligne.

- :mouse pointer: Sors un deuxième ``||basic:toujours||``.
- :mouse pointer: Dans ``||variables:Variables||``, crée une variable **mot**, et mets ``||variables:définir mot à||`` dans ce **toujours**.
- :mouse pointer: Dans son trou, ``||serial:série lire jusque nouvelle ligne||``.

La manette attend un mot, le range dans **mot**, puis recommence.

```blockconfig.local
let mot = serial.readUntil(serial.delimiters(Delimiters.NewLine))
```

```blocks
// @highlight
basic.forever(function () {
    // @highlight
    let mot = serial.readUntil(serial.delimiters(Delimiters.NewLine))
})
```

## Bien et mal

Un son et une icône quand tu détruis un ennemi, ou quand tu es touché.

- :mouse pointer: Sous **définir mot**, mets un ``||logic:si alors sinon si alors||``.
- :mouse pointer: Conditions : ``||logic:mot = "bien"||``, puis ``||logic:mot = "mal"||``.
- :mouse pointer: Pour **bien** : ``||music:jouer tonalité 1319 Hz pour 1/16 temps en arrière-plan||`` et ``||basic:montrer l'icône||`` ✔.
- :mouse pointer: Pour **mal** : une tonalité grave, pour 1/4 temps, et l'icône ✖.

Choisis tes sons et tes icônes : c'est ta manette.

```blockconfig.local
let mot = ""
if (mot == "bien") {
}
music.play(music.tonePlayable(1319, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
```

```blocks
basic.forever(function () {
    let mot = serial.readUntil(serial.delimiters(Delimiters.NewLine))
    // @highlight
    if (mot == "bien") {
        music.play(music.tonePlayable(1319, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
        basic.showIcon(IconNames.Yes)
    } else if (mot == "mal") {
        music.play(music.tonePlayable(131, music.beat(BeatFraction.Quarter)), music.PlaybackMode.InBackground)
        basic.showIcon(IconNames.No)
    }
})
```

## Tes vies

Le jeu envoie aussi tes vies, un chiffre.

- :mouse pointer: Clique sur le **+** du **si** pour ajouter un **sinon**.
- :mouse pointer: Dedans, mets ``||basic:afficher texte||`` avec la variable **mot**.

Tout mot qui n'est ni **bien** ni **mal** s'affiche : le chiffre de tes vies.

```blocks
basic.forever(function () {
    let mot = serial.readUntil(serial.delimiters(Delimiters.NewLine))
    if (mot == "bien") {
        music.play(music.tonePlayable(1319, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
        basic.showIcon(IconNames.Yes)
    } else if (mot == "mal") {
        music.play(music.tonePlayable(131, music.beat(BeatFraction.Quarter)), music.PlaybackMode.InBackground)
        basic.showIcon(IconNames.No)
    } else {
        // @highlight
        basic.showString(mot)
    }
})
```

## Télécharge ta manette @showdialog

Branche ton micro:bit et clique sur **Télécharger**.

Puis **ferme cet onglet MakeCode** : tant qu'il est ouvert, il garde la liaison avec ton micro:bit et vole des lettres aux mots de ta manette.
