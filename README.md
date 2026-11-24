# Up or Out · La Tour

Jeu 2D cartoon en vue de côté, jouable dans le navigateur. Tu es un jeune consultant qui passe son premier jour dans la Tour Franklin, à La Défense : chaque étage est un mini-jeu, et chaque mission réussie te rapporte de l'XP pour monter en grade.

▶ **Jouer en ligne** : https://faustineavril.github.io/up-or-out/ (une fois GitHub Pages activé sur le dépôt)

Analyst → Consultant → Senior Consultant → Manager → Partner

## Lancer le jeu

Ouvre `index.html` dans un navigateur, par un double-clic. Il n'y a ni installation ni serveur. Ta progression est sauvegardée dans le navigateur.

## Mini-jeu 1 · Lobby : Le Ballet des Ascenseurs

- Glisse-dépose chaque client sur le bon groupe d'ascenseurs :
  - **A** : étages 1–12
  - **B** : étages 13–21
  - **C** : étages 22–32
- Tu as 10 secondes par client, 7 secondes pour un VIP.
- La partie dure **1 minute**. La difficulté monte au fil du temps :
  - les clients arrivent de plus en plus vite ;
  - plusieurs clients attendent en même temps, avec des arrivées groupées après 25 s ;
  - des VIP apparaissent à partir de 12 s : ils passent devant les autres et rapportent le double ;
  - des pannes surviennent à partir de 18 s : le groupe concerné refuse tout le monde pendant 5 s, et le chrono de ses clients est gelé.
- Points :
  - 100 par client, plus un bonus de rapidité (jusqu'à +50) et un bonus de série ;
  - −50 et −8 % de satisfaction pour un mauvais groupe ;
  - −10 % de satisfaction pour un client qui repart (−18 % pour un VIP).
- Victoire : **1 800 points**. Si la satisfaction tombe à 0 %, tu es renvoyé.
- Raccourci : `Échap` ou `P` met le jeu en pause.

## Mini-jeu 2 · Étage 1 : La Machine à Café

1. L'ascenseur monte d'un étage. Le consultant traverse l'open space tout seul jusqu'à la machine. Tu peux aller plus vite avec `→`, ou reculer avec `←`.
2. Devant la machine, la caméra zoome en douceur sur son écran.
3. Tu choisis un café. Plus il est corsé, plus le calcul est difficile, et plus il rapporte :

   | Café | Difficulté | Exemple de calcul |
   |---|---|---|
   | Espresso | facile | 7 + 12, 4 × 6 |
   | Allongé | tranquille | 38 + 21, 8 × 5 |
   | Cappuccino | moyen | 67 + 45, 7 × 8 |
   | Latte | costaud | 85 + 76, 11 × 7 |
   | Mocaccino | difficile | 187 + 64, 13 × 7 |

4. Un calcul s'affiche (addition, soustraction ou multiplication), avec un chrono de **10 secondes**.
5. Tu tapes ta réponse sur le pavé de la machine à la souris, ou au clavier : chiffres, `Retour arrière` pour effacer, `Entrée` pour valider.
6. Tu n'as **qu'une seule chance** :
   - bonne réponse : la tasse se remplit et la machine affiche « Victoire » ;
   - mauvaise réponse ou temps écoulé : « PAS DE CAFÉ POUR TOI », puis « Défaite ».
7. **Rejouer** tire un nouveau calcul. **Changer de café** revient au menu. **Retour au bureau** ouvre la Performance Review, basée sur ton dernier essai.

Score d'une bonne réponse : 200 + 100 × niveau du café + 30 par seconde restante. Il faut au moins 300 points pour réussir.

## Mini-jeu 3 · Étage 8 : Raccourcis Hero (PowerPoint)

Ce mini-jeu existe en deux versions, construites à partir des mêmes sources :

- **dans la tour**, à l'étage 8 (Open Space). Il faut 6 000 points pour réussir la mission ;
- **en fichier HTML unique et autonome** : [`ppt-raccourcis.html`](ppt-raccourcis.html), à ouvrir directement dans le navigateur.

Une partie compte 10 actions PowerPoint :

| Questions | Niveau | Coefficient | Type de raccourci |
|---|---|---|---|
| 1 à 3 | facile | ×1 | Ctrl + une lettre |
| 4 à 7 | intermédiaire | ×1,5 | trois touches et touches de fonction |
| 8 à 10 | difficile | ×2 | enchaînement de deux raccourcis |

Pour chaque question :

1. **Phase 1 : à l'aveugle.** Tu as 10 s pour taper le raccourci de mémoire. Une réussite rapporte 1 000 points, plus un bonus de vitesse jusqu'à +500.
2. **Phase 2 : raccourci révélé.** Les touches descendent vers une ligne de validation, comme dans un jeu de rythme. Tu as encore 5 s. Une réussite rapporte 200 points, plus un bonus jusqu'à +200.
3. Si la phase 2 expire aussi, la question rapporte 0 point. La réponse reste affichée 1,5 s.

Règles complémentaires :

- **Coefficient** : les points sont multipliés par le coefficient du niveau.
- **Combo** : il augmente de +0,25 à chaque réussite en phase 1, jusqu'à ×2. Il retombe à ×1 après une réussite en phase 2 ou un échec.
- **Mauvaise combinaison** : la question n'est pas perdue. Le jeu secoue la diapositive et flashe en rouge, puis tu peux réessayer tant que le temps n'est pas écoulé.
- **Enchaînements** : une erreur ramène au début de la séquence.
- **Touches seules** : Ctrl, Maj ou Alt seuls sont ignorés.
- **Pause** : `Échap` met en pause. C'est la seule touche réservée.

### Architecture

```
js/floors/ppt/shortcuts-config.js   BANQUE DE RACCOURCIS (seul fichier à modifier pour les questions)
js/floors/ppt/ppt-game.js           module autonome : window.PptShortcuts.createGame(container, options)
css/floors/ppt.css                  styles (toutes les classes sont préfixées pptg-)
js/floors/open-space.js             adaptateur pour la tour (sons, Quitter, Performance Review)
tools/build-standalone.js           génère ppt-raccourcis.html (un seul fichier, tout est inline)
```

Interface du module :

- Point d'entrée : `PptShortcuts.createGame(container, { config, onGameEnd, sfx, onQuit, endActions })`. C'est le seul nom global exposé.
- Méthodes du jeu : `start()` lance une partie, `destroy()` retire tout.
- Fin de partie : `onGameEnd({ score, details })` est appelé. `details` contient, pour chaque question : action, raccourci, résultat (`phase1`, `phase2` ou `missed`) et points.

Format d'une entrée de la banque :

```js
{ label: 'Dupliquer la diapositive, puis lancer le diaporama depuis celle-ci', combos: ['Ctrl+D', 'Maj+F5'], level: 'difficile' }
```

Au chargement, le jeu exclut automatiquement toute entrée invalide ou non interceptable par le navigateur : Ctrl+N, Ctrl+W, Ctrl+T, Ctrl+Maj+T, Ctrl+Maj+N, Alt+F4, Alt+Tab, la touche Windows… La liste est modifiable dans `forbidden`. Les exclusions sont affichées dans la console.

Après avoir modifié la banque, régénère le fichier autonome :

```powershell
node tools/build-standalone.js
```

### Neutralisation du clavier

- **Pendant la partie** :
  - `keydown`, `keyup` et `keypress` sont capturés sur `window` en phase de capture, avec `preventDefault()` et `stopPropagation()`, pour toutes les touches : F1–F12, Alt seul, mauvaises combinaisons ;
  - la zone de jeu n'a ni menu contextuel, ni sélection, ni champ de saisie.
- **Sur l'accueil, l'écran de fin et la tour** : le navigateur retrouve son comportement normal.
- **Mode facultatif « Plein écran + verrouillage clavier »** : sur Chrome et Edge, il utilise `navigator.keyboard.lock()`.
- **AZERTY et QWERTY** : les lettres sont comparées sur `event.key`. Alt Gr compte comme Ctrl+Alt, donc Alt Gr+V = Ctrl+Alt+V. Cmd compte comme Ctrl sur Mac.
- **Test par navigateur** : l'écran d'accueil propose **« Tester les raccourcis sur ce navigateur »**. Il liste chaque combinaison de la banque et la passe au vert quand le jeu l'a bien interceptée. Si le navigateur réagit quand même, retire la combinaison de la banque. Ce test est à faire à la main sur Chrome, Edge et Firefox.

## Mini-jeu 4 · Étage 11 : Feuille de temps Express

Ce mini-jeu existe aussi en deux versions : **dans la tour**, à l'étage 11 (Contrôle de gestion), où il faut 7 000 points pour réussir, et en **fichier HTML unique** : [`feuille-de-temps.html`](feuille-de-temps.html). Il se joue à la souris, au clavier ou au toucher, sur ordinateur comme sur mobile.

**Règle de conversion : 1 heure imputée = 1 seconde réelle.**

Chaque manche suit le même déroulé :

1. Une consigne s'affiche, par exemple « Tu dois imputer 5 h 00 sur « Projet Alpha – Cadrage IA » ».
2. **Start** lance le chronomètre, affiché en `HH:MM` d'heures imputées.
3. **Imputer**, ou la barre d'espace, l'arrête. L'objectif est de s'arrêter pile sur la valeur demandée.
4. Le jeu affiche le temps imputé, l'écart en minutes (« 12 minutes imputées en trop », « 3 minutes oubliées »), les points et un tampon (Validé, À revoir, Refusé).
5. Si tu ne fais rien 3 s après la cible, la manche s'arrête : 0 point et « Feuille de temps non soumise ».

| Manches | Niveau | Coefficient | Valeurs cibles | Chronomètre |
|---|---|---|---|---|
| 1 à 3 | facile | ×1 | heures pleines de 2 h à 5 h | visible |
| 4 à 7 | intermédiaire | ×1,5 | demi-heures et quarts d'heure de 3 h à 8 h | masqué au bout de 2 s |
| 8 à 10 | difficile | ×2 | à la minute près de 5 h à 10 h | masqué dès le départ |

**Points par manche :** 1 000 × max(0, 1 − écart en ms / 1 000), puis × coefficient du niveau. Un écart de 20 ms ou moins ajoute un bonus de 250 points.

| Écart | Palier |
|---|---|
| ≤ 20 ms | Parfait – validé sans relecture |
| ≤ 100 ms | Excellent |
| ≤ 250 ms | Correct |
| ≤ 500 ms | Approximatif – le manager fronce les sourcils |
| > 500 ms | Refusé par le contrôle de gestion |

### Architecture

```
js/floors/timesheet/timesheet-config.js   CONFIGURATION : projets, niveaux, paliers, durées
js/floors/timesheet/timesheet-game.js     module autonome : window.TimesheetGame.createGame(container, options)
css/floors/timesheet.css                  styles (classes préfixées tsg-), responsive par container queries
js/floors/timesheet.js                    adaptateur pour la tour (sons, Quitter, Performance Review)
```

L'interface est la même que pour Raccourcis Hero :

- `TimesheetGame.createGame(container, { config, onGameEnd, sfx, onQuit, endActions })` renvoie `{ start(), destroy() }`. C'est le seul nom global exposé.
- `onGameEnd({ score, details })` est appelé en fin de partie. `details` contient une ligne par manche : jour, projet, cible, temps imputé, écart en ms et en minutes, palier, points.

**Précision du chronométrage :**

- **Horodatage** : le départ et l'arrêt sont relevés avec `performance.now()` en première ligne des gestionnaires `pointerdown` et `keydown` (pas `click`). Le score ne dépend que de ces deux horodatages.
- **Affichage** : `requestAnimationFrame` ne sert qu'à l'œil. Une partie de test où l'affichage était complètement bloqué pendant toute l'attente a donné un écart mesuré de 1 ms.
- **Barre d'espace** : `preventDefault()` (pas de défilement ni de double action), et les répétitions automatiques (`event.repeat`) sont ignorées.
- **Saisies parasites** : toute saisie est ignorée 500 ms après un arrêt. Un arrêt dans les 300 ms qui suivent le départ est aussi ignoré, pour qu'un double-clic sur Start n'arrête pas le chrono aussitôt.

## Régénérer les fichiers HTML autonomes

Les fichiers `ppt-raccourcis.html` et `feuille-de-temps.html` sont générés à partir des mêmes sources que la tour. Après une modification d'une configuration ou d'un module :

```powershell
node tools/build-standalone.js
```

## Structure

```
index.html              charge le noyau, puis les étages, puis app.js
css/style.css           styles communs (tour, écrans, review)
css/floors/lobby.css    styles du mini-jeu Lobby
css/floors/coffee.css   styles du mini-jeu Machine à café
js/core/engine.js       registre des étages : TowerGame.registerFloor()
js/core/state.js        XP, grades, déblocages, records (localStorage)
js/core/character.js    personnages SVG partagés (le consultant, les clients)
js/core/review.js       Performance Review (paliers, citations, XP)
js/core/sfx.js          sons synthétisés
js/core/app.js          écran titre, vue de la tour, lancement des étages, review
js/floors/lobby.js      mini-jeu 1 (RDC)
js/floors/coffee.js     mini-jeu 2 (Étage 1)
js/floors/open-space.js mini-jeu 3 (Étage 8), qui branche le module js/floors/ppt/ sur la tour
ppt-raccourcis.html     mini-jeu 3 en fichier HTML autonome (généré)
js/floors/timesheet.js  mini-jeu 4 (Étage 11), qui branche le module js/floors/timesheet/ sur la tour
feuille-de-temps.html   mini-jeu 4 en fichier HTML autonome (généré)
tools/build-standalone.js  génère les deux fichiers HTML autonomes
js/floors/coming-soon.js  étages à venir (affichés COMING SOON)
js/floors/_TEMPLATE.js  modèle pour créer un nouvel étage
```

## Ajouter un étage (un nouveau mini-jeu)

1. Copie `js/floors/_TEMPLATE.js` vers, par exemple, `js/floors/open-space.js`.
2. Garde le même `id` qu'un étage de `coming-soon.js` pour le remplacer à sa place dans la tour, ou choisis un nouvel `id` et un nouvel `order`.
3. Écris ton jeu dans `start(root, ctx)` :
   - `root` est une zone de 1280×720 px, mise à l'échelle automatiquement ;
   - appelle `ctx.finish({ score, target, success, stats })` à la fin de la partie ;
   - renvoie `{ destroy() }` pour arrêter proprement tes boucles et tes timers.
4. Ajoute une ligne `<script src="js/floors/open-space.js"></script>` dans `index.html`, avant `app.js`. Ajoute aussi une feuille CSS si besoin.

Le noyau se charge tout seul du reste :

- l'XP, gagnée seulement en cas de réussite. Elle vaut 100 XP quand l'objectif est tout juste atteint, et monte jusqu'à 250 XP à 2,5 fois l'objectif, plus un bonus de palier. Elle ne dépend donc pas de l'échelle des scores de chaque mini-jeu ;
- les promotions ;
- le déblocage de l'étage suivant ;
- les records ;
- la Performance Review. Ta fonction optionnelle `reviewComments(result)` peut y ajouter des commentaires, et `reviewQuotes` peut remplacer les citations du manager pour chaque palier.

L'ordre des étages dans la tour vient de `order`. Un étage s'ouvre dès que celui du dessous a été réussi une fois. Tu peux donc insérer un étage entre deux autres sans casser les sauvegardes existantes.
