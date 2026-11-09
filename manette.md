# Ta manette

## Ta manette @showdialog

Ton micro:bit devient la manette de ton jeu d'arcade. D'abord, il envoie tes gestes : tu l'inclines pour monter et descendre, tu appuies sur A pour tirer.

Ensuite, il écoute ton jeu : un son et une icône quand tu détruis un ennemi ou quand tu es touché, et tes vies qui s'affichent.

## Regarde ton accéléromètre

Avant de choisir quand ta manette monte ou descend, regarde les nombres que donne son accéléromètre.

- :mouse pointer: Dans le ``||basic(noclick):toujours||`` déjà présent, mets ``||serial:série écrire valeur "y" = ||`` avec ``||input:accélération (mg) y||`` de ``||input:Entrées||``. Les blocs ``||serial:série||`` sont dans **Avancé**.
- :mouse pointer: Clique sur **Télécharger**, garde le câble branché, puis clique sur **Afficher données Appareil**.
- :mouse pointer: Incline ta manette vers l'avant, puis vers l'arrière. Note la valeur de **y** à partir de laquelle elle est vraiment penchée.

Ta manette envoie ses mesures par la **liaison série**. C'est par elle aussi qu'elle parlera au jeu.

```blockconfig.local
serial.writeValue("y", input.acceleration(Dimension.Y))
```

```blocks
basic.forever(function () {
    // @highlight
    serial.writeValue("y", input.acceleration(Dimension.Y))
})
```

## Monter et descendre

Ta manette envoie **haut** quand tu l'inclines vers l'avant, **bas** vers l'arrière.

- :mouse pointer: Dans le ``||basic(noclick):toujours||``, remplace ``||serial:série écrire valeur||`` par un ``||logic:si alors sinon si alors||`` de ``||logic:Logique||``.
- :mouse pointer: Conditions : ``||input:accélération (mg) y||`` plus petite que ton seuil avant, puis plus grande que ton seuil arrière.
- :mouse pointer: Dans chaque branche, ``||serial:série écrire ligne||`` avec **haut**, puis **bas**.

Mets les seuils que tu as notés : ici, -300 et 300.

```blockconfig.local
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

## Tirer

Ta manette envoie **A** quand tu appuies sur son bouton A.

- :mouse pointer: Dans le même ``||basic(noclick):toujours||``, sous le premier **si**, ajoute ``||logic:si alors||``.
- :mouse pointer: Condition : ``||input:bouton A est pressé||``. Dans la branche : ``||serial:série écrire ligne||`` avec **A**.
- :mouse pointer: À la fin du **toujours**, ajoute ``||basic:pause (ms) 20||``.

Tant que tu tiens le bouton, la manette répète **A** toutes les 20 ms.

```blockconfig.local
serial.writeLine("A")
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
    if (input.buttonIsPressed(Button.A)) {
        serial.writeLine("A")
    }
    // @highlight
    basic.pause(20)
})
```

## Teste ta manette @showdialog

Clique sur **Télécharger**. Puis clique sur **⋯** à côté de **Télécharger**, et sur **Déconnecter** : la liaison série est libre pour la page relais.

Garde cet onglet ouvert. Va jouer avec ta manette (page de la séance, étape 2), puis fais parler ton jeu à ta manette (étape 3).

Reviens ensuite ici : ta manette va apprendre à écouter ton jeu.

## De la place pour recevoir

Ton jeu envoie maintenant des mots à ta manette. Réserve-leur de la place.

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

## Rejoue @showdialog

Clique sur **Télécharger**, puis sur **⋯** et **Déconnecter**.

Rejoue avec ta manette dans la page relais : elle réagit maintenant à ton jeu.
