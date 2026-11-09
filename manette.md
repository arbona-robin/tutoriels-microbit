# Ta manette

## Ta manette @showdialog

Ton micro:bit devient la manette de ton jeu d'arcade. D'abord, avec les seuils que tu as notés, il envoie tes gestes : tu l'inclines pour monter et descendre, tu appuies sur A pour tirer.

Ensuite, il écoute ton jeu : un son et une icône quand tu détruis un ennemi ou quand tu es touché, et tes vies qui s'affichent.

## Monter et descendre

Ta manette envoie **haut** quand tu l'inclines vers l'avant, **bas** vers l'arrière. Garde sous la main les seuils que tu as notés.

- :mouse pointer: Dans le ``||basic(noclick):toujours||`` déjà présent, mets ``||logic:si alors sinon si alors||`` de ``||logic:Logique||``.
- :mouse pointer: Dans chaque condition, mets le bloc de comparaison ``||logic:0 < 0||`` de ``||logic:Logique||`` : à gauche ``||input:accélération (mg) y||`` de ``||input:Entrées||``, à droite ton seuil. Choisis le signe < pour l'avant, > pour l'arrière.
- :mouse pointer: Dans chaque branche, ``||serial:série écrire ligne||``, dans **Avancé**, ``||serial:Communication série||`` : **haut**, puis **bas**.

```blockconfig.local
if (true) {
} else if (true) {
}
serial.writeLine("haut")
```

```blocks
// @highlight
basic.forever(function () {
    if (input.acceleration(Dimension.Y) < -300) {
        serial.writeLine("haut")
    } else if (input.acceleration(Dimension.Y) > 300) {
        serial.writeLine("bas")
    }
})
```

## Un mot toutes les 20 ms

Sans pause, ta manette enverrait des milliers de mots par seconde et noierait la liaison série.

- :mouse pointer: À la fin du ``||basic(noclick):toujours||``, ajoute ``||basic:pause (ms) 20||`` de ``||basic:Base||``.

Avec la pause, ta manette envoie 50 mots par seconde : assez pour que ton héros bouge sans à-coups.

```blockconfig.local
basic.pause(20)
```

```blocks
basic.forever(function () {
    if (input.acceleration(Dimension.Y) < -300) {
        serial.writeLine("haut")
    } else if (input.acceleration(Dimension.Y) > 300) {
        serial.writeLine("bas")
    }
    // @highlight
    basic.pause(20)
})
```

## Tirer

Ta manette envoie **A** quand tu appuies sur son bouton A.

- :mouse pointer: Avant la pause, ajoute ``||logic:si alors||``, avec ``||input:bouton A est pressé||`` de ``||input:Entrées||``.
- :mouse pointer: Dans la branche : ``||serial:série écrire ligne||`` avec **A**, en majuscule.

La page relais attend exactement ces mots : **haut**, **bas** et **A**, au caractère près. **A** et **B** gardent la majuscule du nom des boutons.

```blockconfig.local
if (input.buttonIsPressed(Button.A)) {
}
serial.writeLine("A")
```

```blocks
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
    basic.pause(20)
})
```

## De la place pour recevoir

Ton jeu envoie des mots à ta manette. Réserve-leur de la place.

- :mouse pointer: Dans **Avancé**, ``||serial:Communication série||``, glisse ``||serial:série définir taille tampon RX à 128||`` dans ``||basic(noclick):au démarrage||``.

Sans cette réserve, les mots qui arrivent pendant qu'une icône s'affiche se perdent.

```blocks
// @highlight
serial.setRxBufferSize(128)
basic.forever(function () {
    if (input.acceleration(Dimension.Y) < -300) {
        serial.writeLine("haut")
    } else if (input.acceleration(Dimension.Y) > 300) {
        serial.writeLine("bas")
    }
    if (input.buttonIsPressed(Button.A)) {
        serial.writeLine("A")
    }
    basic.pause(20)
})
```

## Écouter le jeu

Ta manette lit les mots que le jeu lui envoie, un par ligne.

- :mouse pointer: Dans ``||basic:Base||``, sors un deuxième ``||basic:toujours||``.
- :mouse pointer: Dans ``||variables:Variables||``, crée une variable **mot**, et mets ``||variables:définir mot à||`` dans ce **toujours**.
- :mouse pointer: Dans son trou, ``||serial:série lire jusque nouvelle ligne||``.

Les deux **toujours** tournent en même temps : l'un envoie tes gestes, l'autre écoute le jeu.

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

- :mouse pointer: Sous **définir mot**, mets le bloc ``||logic:si mot = "bien" alors sinon si mot = "mal" alors||`` de ``||logic:Logique||`` : un seul bloc pour les deux mots.
- :mouse pointer: Pour **bien** : ``||music:jouer tonalité 1319 Hz pour 1/16 temps en arrière-plan||`` et ``||basic:montrer l'icône||`` ✔.
- :mouse pointer: Pour **mal** : une tonalité grave, pour 1/4 temps, et l'icône ✖.

Choisis tes sons et tes icônes : c'est ta manette.

```blockconfig.local
let mot = ""
if (mot == "bien") {
} else if (mot == "mal") {
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

- :mouse pointer: Clique sur le **+** en bas du bloc **si mot = bien … sinon si mot = mal** pour ajouter un **sinon**.
- :mouse pointer: Dedans, mets ``||basic:afficher texte||`` avec la variable **mot**.

Tout mot qui n'est ni **bien** ni **mal** s'affiche : le chiffre de tes vies. Si **mal** s'affiche en lettres, ton **sinon** n'est pas dans le même bloc que **mal**.

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

Clique sur **Télécharger**.

Puis **ferme cet onglet MakeCode** : tant qu'il est ouvert, il garde la liaison série avec ton micro:bit et vole des lettres aux mots de ta manette.

Tu peux maintenant jouer avec ta manette, dans la page relais.
