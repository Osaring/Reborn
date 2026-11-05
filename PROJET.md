# Reborn — carnet de bord partagé

> **À lire par toute IA (Claude, ChatGPT…) avant de travailler sur le projet, et à mettre à jour à la fin de chaque session.**
> Le propriétaire du projet (Valentin) ne code pas : explique-lui les choses simplement, en français.

---

## 1. Le projet en bref

**Reborn** est un simulateur de vie pour iPhone, inspiré de BitLife. Le joueur vit une vie année par année : événements à choix, études, carrière, relations, objectifs annuels, trophées, et des étoiles ★ qui se gardent d'une vie à l'autre pour acheter des atouts de naissance.

- **Site en ligne :** https://osaring.github.io/Reborn/
- Le jeu est un **site web** ajouté à l'écran d'accueil de l'iPhone (Safari → Partager → Sur l'écran d'accueil). Ce n'est pas une app de l'App Store.
- **Langue du jeu :** français. **Cible :** iPhone, en portrait.

## 2. Où est le code (important)

| Élément | Détail |
|---|---|
| Dépôt GitHub | `Osaring/Reborn` |
| **Branche publiée** | **`claude/site`** ← c'est elle que GitHub Pages affiche |
| Branche `main` | Ancienne version, **pas à jour**. Ne pas travailler dessus. |

➡️ **Toujours partir de `claude/site` et y envoyer les modifications.**

## 3. Les fichiers

| Fichier | Rôle |
|---|---|
| `index.html` | **Tout le jeu** : HTML, CSS et JavaScript dans un seul fichier |
| `version.json` | Numéro de la dernière version, ex. `{"v":10}` |
| `manifest.webmanifest` | Nom et icônes de l'app pour l'écran d'accueil |
| `apple-touch-icon.png`, `icon-192.png`, `icon-512.png` | Icônes (étoile dorée sur fond sombre) |
| `PROJET.md` | Ce carnet de bord |

## 4. Organisation de `index.html` (dans l'ordre du fichier)

1. **CSS** : les couleurs sont des variables (`--bg`, `--accent`, `--bon`, `--san`, `--int`, `--cha`…) avec un thème clair et un thème sombre. Polices : *Bricolage Grotesque* (titres) et *Figtree* (texte).
2. **Données et logique**
   - `S` : la vie en cours (âge, stats, argent, métier, proches, journal…). `M` : la progression permanente (étoiles, trophées, record).
   - `STATS` : bonheur, santé, intelligence, charme (de 0 à 100).
   - `JOBS` : les métiers. `EDU` / `STUDY` : diplômes et études.
   - `PERKS` : atouts de naissance. `TROPH` : trophées et défis. `OBJ` : objectifs de l'année.
   - `EV` : **événements classiques**, déclarés avec `E(id, âgeMin, âgeMax, probabilité, {icon, title, text, c:[choix]})`.
   - `ACTS` : activités (1 éclair d'énergie chacune, 4 par an).
   - `newLife()`, `newYear()`, `ageUp()`, `checkDeath()`, `die()`, `checkTrophies()`.
3. **Icônes et affichage** : `I` (icônes SVG), `render()`, puis un panneau par onglet : `pJournal`, `pActs`, `pCareer`, `pRel`, `pDefis`.
4. **IA** (voir section 6).
5. **Fenêtres** : `runQueue()` (file d'événements), `choose()`, `showDeath()`, `optionsModal()`.
6. **Accueil** (`renderStart`), **gestion des clics** (un seul écouteur `document.addEventListener('click', …)`), puis **mises à jour** et **démarrage** (`boot`).

### Ajouter un événement classique (exemple)
```js
E('monid', 18, 60, .15, {icon:'🎸', title:'Concert surprise', text:()=>'Ton groupe préféré passe en ville.', c:[
  C('Y aller', ()=>(fx({bonheur:6, money:-80}), 'Soirée inoubliable.')),
  C('Rester chez toi', ()=>'Tu regardes la vidéo le lendemain.'),
]});
```
`fx({...})` applique les effets (`bonheur`, `sante`, `intel`, `charme`, `money`). Le texte renvoyé s'affiche comme résultat.

## 5. Règles à respecter

- **Sauvegarde :** la partie est stockée dans `localStorage`, sous la clé `destinees_v1`. **Ne jamais renommer cette clé**, sinon tous les joueurs perdent leur partie.
- **Version :** à chaque modification publiée, **augmenter de 1** `APP_VERSION` dans `index.html` **et** `v` dans `version.json`, avec le même nombre. C'est ce qui déclenche la mise à jour automatique sur le téléphone.
- **Aucune clé d'API dans le code.** Le dépôt est public. La clé Gemini est collée par le joueur dans *Options* et reste dans son téléphone (`localStorage` : `rb_gkey`).
- **Écran iPhone :** le jeu doit tenir sur un seul écran, sans que la page défile (seule la zone centrale défile). Garder `apple-mobile-web-app-status-bar-style` sur `default` : le mode `black-translucent` provoque un bug d'iOS qui coupe le bas de l'écran.
- **Style :** visuel, peu de texte, icônes plutôt que longues phrases. Ne pas ajouter de librairies lourdes.
- **Contenu :** pas de romance ni de contenu sexuel pour les personnages mineurs.
- **Tester** avant de publier : vérifier au minimum que le JavaScript n'a pas d'erreur de syntaxe.

## 6. Le système d'IA (Gemini, offre gratuite)

- Le joueur colle une clé Google AI Studio (gratuite) dans *Options*.
- `aiModels()` demande à Google la liste des modèles disponibles et choisit un modèle *Flash-Lite* (rapide et économique). Le choix est mémorisé dans `rb_models`.
- `aiGenerate(âge, 4)` crée **4 événements d'un coup**, au format JSON. `aiPrefetch()` garde une **réserve** (`S.aiStock`) d'au moins 3 événements, préparée en arrière-plan.
- **Le joueur n'attend jamais :** si la réserve est vide, le jeu affiche un événement classique à la place (`aiLoad`).
- Les effets proposés par l'IA sont limités par `sanitizeAI()` (stats entre −15 et +15, argent plafonné).

## 7. Journal des modifications

| Version | Qui | Changement |
|---|---|---|
| v1–v2 | Claude | Première version du jeu, renommé « Reborn », mis en ligne sur GitHub Pages |
| v3 | Claude | Nouveau design plein écran : anneaux de stats, tuiles, barre d'onglets |
| v3 | Claude | Mise à jour automatique + bouton « Vérifier les mises à jour » |
| v4 | Claude | Bouton Rafraîchir toujours visible en haut |
| v5–v6 | Claude | Correction du bas de l'écran coupé sur iPhone (barre d'état en mode `default`) |
| v7 | Claude | Événements générés par IA (Gemini) + réglages dans Options |
| v8 | Claude | Choix automatique du modèle Gemini + affichage des erreurs |
| v9 | Claude | Événement IA préparé à l'avance, réflexion désactivée |
| v10 | Claude | Réserve de 4 événements IA, plus jamais d'attente |

## 8. Tâches

> Avant de commencer une tâche, écrire son nom à côté (« en cours — ChatGPT ») pour éviter que deux IA fassent la même chose.

**À faire**
- [ ] Acheter des biens (logement, voiture) avec un loyer ou un crédit
- [ ] Plus d'événements classiques à l'âge adulte et à la vieillesse
- [ ] Équilibrage : l'argent devient vite négatif sans emploi

**En cours**
- (rien)

**Terminé**
- [x] Voir le journal des modifications ci-dessus
