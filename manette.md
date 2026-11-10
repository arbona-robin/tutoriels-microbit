# Ta manette

## Ta manette @showdialog

Ton micro:bit devient la manette de ton jeu d'arcade. D'abord, avec les seuils que tu as notés, il envoie tes gestes : tu l'inclines pour monter et descendre, tu appuies sur A pour tirer.

Ensuite, il écoute ton jeu : un son quand tu tires, quand tu détruis un ennemi ou quand tu es touché, et tes vies sur son écran.

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

## Bip, bien et mal

Un son quand tu tires, un autre quand tu détruis un ennemi, un troisième quand tu es touché.

- :mouse pointer: Sous **définir mot**, mets le bloc ``||logic:si mot = "bip" alors sinon si mot = "bien" alors sinon si mot = "mal" alors||`` de ``||logic:Logique||`` : un seul bloc pour les trois mots.
- :mouse pointer: Dans chaque branche, ``||music:jouer tonalité pour 1/16 temps en arrière-plan||`` de ``||music:Musique||`` : **High G** pour **bip**, **High C** pour **bien**.
- :mouse pointer: Pour **mal**, **Low C** pour **1/4** temps : un son grave, plus long.

**En arrière-plan** : le son se lance, et la boucle lit aussitôt le mot suivant. Choisis tes notes : c'est ta manette.

```blockconfig.local
let mot = ""
if (mot == "bip") {
} else if (mot == "bien") {
} else if (mot == "mal") {
}
music.play(music.tonePlayable(Note.G5, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
```

```blocks
basic.forever(function () {
    let mot = serial.readUntil(serial.delimiters(Delimiters.NewLine))
    // @highlight
    if (mot == "bip") {
        music.play(music.tonePlayable(Note.G5, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
    } else if (mot == "bien") {
        music.play(music.tonePlayable(Note.C5, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
    } else if (mot == "mal") {
        music.play(music.tonePlayable(Note.C3, music.beat(BeatFraction.Quarter)), music.PlaybackMode.InBackground)
    }
})
```

## Le mot pour l'écran

Tout autre mot, comme le chiffre de tes vies, est pour l'écran. La boucle d'écoute le range, sans l'afficher.

- :mouse pointer: Clique sur le **+** en bas du bloc **si mot = bip …** pour ajouter un **sinon**.
- :mouse pointer: Dans ``||variables:Variables||``, crée une variable **ecran**, et mets ``||variables:définir ecran à||`` dans le **sinon**, avec la variable **mot**.

```blockconfig.local
let mot = ""
let ecran = mot
```

```blocks
let ecran = ""
basic.forever(function () {
    let mot = serial.readUntil(serial.delimiters(Delimiters.NewLine))
    if (mot == "bip") {
        music.play(music.tonePlayable(Note.G5, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
    } else if (mot == "bien") {
        music.play(music.tonePlayable(Note.C5, music.beat(BeatFraction.Sixteenth)), music.PlaybackMode.InBackground)
    } else if (mot == "mal") {
        music.play(music.tonePlayable(Note.C3, music.beat(BeatFraction.Quarter)), music.PlaybackMode.InBackground)
    } else {
        // @highlight
        ecran = mot
    }
})
```

## Affiche l'écran

Une troisième boucle affiche le dernier mot reçu, en permanence.

- :mouse pointer: Dans ``||basic:Base||``, sors un troisième ``||basic:toujours||``.
- :mouse pointer: Dedans, mets ``||basic:afficher texte||`` avec la variable **ecran**.

Afficher prend du temps : presque une seconde pour un chiffre. Dans la boucle d'écoute, ce serait une seconde sans écouter, et les sons arriveraient en retard, puis tous d'un coup. Une boucle par tâche : une envoie tes gestes, une écoute le jeu, une affiche.

```blockconfig.local
let ecran = ""
basic.showString(ecran)
```

```blocks
let ecran = ""
// @highlight
basic.forever(function () {
    // @highlight
    basic.showString(ecran)
})
```

## Télécharge ta manette @showdialog

Clique sur **Télécharger**.

Puis **ferme cet onglet MakeCode** : tant qu'il est ouvert, il garde la liaison série avec ton micro:bit et vole des lettres aux mots de ta manette.

Tu peux maintenant jouer avec ta manette, dans la page relais.
