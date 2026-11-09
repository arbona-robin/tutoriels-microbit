# Tutoriels MakeCode micro:bit

Tutoriels en français pour les ateliers du livre Maker Junior.

| Tutoriel | Séance | Lien |
|---|---|---|
| Ta manette | Jeu d'arcade, séance 3 | https://makecode.microbit.org/?lang=fr#tutorial:https://github.com/arbona-robin/tutoriels-microbit/manette |

Un tutoriel par fichier `.md` à la racine, déclaré dans `files` de `pxt.json`. Aucun fichier `.ts` dans le dépôt : il serait compilé dans chaque tutoriel. Les programmes de référence sont en `.ts.txt` (`manette-s3.ts.txt` : la manette à la fin de la séance 3).

La manette dialogue avec la page relais du jeu d'arcade, dont le contrat est décrit dans le dépôt [tutoriels-arcade](https://github.com/arbona-robin/tutoriels-arcade/blob/main/manette-microbit.md).

## Publier une modification

MakeCode charge le dernier tag du dépôt, pas le dernier commit. À chaque publication : nouvelle version dans `pxt.json`, tag du même numéro (`v0.2.0`…), puis ouvrir ces deux adresses pour vider le cache de MakeCode :

- `https://www.makecode.com/api/gh/arbona-robin/tutoriels-microbit/refs?nocache=1`
- `https://www.makecode.com/api/ghtutorial/arbona-robin/tutoriels-microbit/<tutoriel>?noCache=1`
