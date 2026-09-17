# CAMUSAT TALK

**CAMUSAT TALK** est une application interne développée pour **Camusat Sénégal**. Elle sert de plateforme au programme du même nom : un rendez-vous de prise de parole, de partage d'expérience et de développement des compétences, dans lequel les collaborateurs proposent et présentent un sujet, accompagnés d'un mentor.

> Share. Learn. Inspire.

L'application permet à un collaborateur de proposer un sujet en moins de deux minutes, à tout le monde de consulter les présentations en cours (filtrées par thématique et par statut), et à l'équipe RH / Direction de suivre et piloter l'ensemble des propositions depuis un tableau de bord dédié.

## Fonctionnalités

- **Page d'accueil** : présentation du programme, statistiques en direct, accès rapide aux deux actions principales.
- **Formulaire de proposition** : informations du présentateur, sujet, thématique, mentor accompagnateur, objectif, durée et date souhaitée — avec validation des champs et message de confirmation.
- **Page Présentateurs** : les propositions sous forme de cartes, avec filtres par thématique et par statut, et une recherche libre.
- **Dashboard administrateur** (protégé par un code d'accès) : vue d'ensemble chiffrée, modification du statut et du mentor de chaque proposition, export des données en CSV et en Excel (.xlsx).
- Identité visuelle Camusat (bleu foncé `#203261`, rouge accent), interface responsive (mobile / tablette / desktop), thème clair et sombre.

## Technologies utilisées

- HTML5 / CSS3 (variables CSS, grille, media queries) — aucun framework CSS, styles écrits à la main.
- JavaScript (ES5/ES6, vanilla — sans framework front-end).
- Police [Google Fonts](https://fonts.google.com/) : Sora (titres) et Inter (texte).
- [SheetJS / xlsx](https://github.com/SheetJS/sheetjs) (chargé depuis un CDN, uniquement au moment de l'export Excel).
- **Stockage des données** : base de données partagée fournie par le runtime **Claude Artifact** (capacité `db`) — voir la section suivante, c'est le point le plus important à comprendre avant de continuer le développement.

## Comment ça marche : une application pensée pour Claude Artifact

Ce projet a été construit et publié comme un **Claude Artifact** (une page web hébergée sur claude.ai). Concrètement, `index.html` est un fichier autonome, mais il appelle à l'exécution des fonctions spéciales exposées par la plateforme qui l'héberge :

| Fonction | Rôle dans l'application |
|---|---|
| `window.claude.use("db")` | Stocke et synchronise en temps réel les propositions entre tous les collaborateurs qui ouvrent la page. |
| `window.claude.use("downloads")` | Permet d'enregistrer les exports CSV / Excel sur l'ordinateur de la personne connectée. |
| `window.claude.use("user")` | Détermine qui a les droits d'administration (`canEdit` / `isOwner`) sur l'artifact. |

**Ces trois fonctions n'existent que lorsque la page est ouverte via une session Claude.** Si tu ouvres `index.html` directement dans un navigateur (double-clic), ou si tu le déploies sur GitHub Pages, Netlify, Vercel, etc., `window.claude` n'existe pas : le code le détecte et bascule sur un stockage local vide en mémoire (`LOCAL_FALLBACK` dans le script — volontairement sans aucune proposition d'exemple). Tu peux donc toujours consulter et faire évoluer l'interface, les formulaires et le design localement, avec les pages vides ("Aucun sujet pour le moment") — mais sans persistance réelle des données ni export fonctionnel.

Ce dépôt a pour rôle de **conserver et versionner le code source**. Pour une utilisation en production avec plusieurs collaborateurs, deux options :

1. **Rester sur Claude Artifact** : republier ce fichier via l'outil Artifact d'une session Claude (c'est ce qui a été fait initialement) — aucune configuration de base de données à gérer.
2. **Migrer vers un vrai backend** (si l'application doit vivre en dehors de Claude) : remplacer les trois appels `window.claude.use(...)` par un service équivalent (par exemple [Supabase](https://supabase.com/) pour la base de données et l'authentification), et reconstruire l'interface en React / Next.js si tu veux repartir sur la stack initialement envisagée. C'est un chantier à part entière, distinct de la sauvegarde du code actuel.

## Installation locale

Aucune dépendance ni build n'est nécessaire : le projet est un unique fichier HTML autonome.

```bash
git clone https://github.com/<ton-compte>/camusat-talk.git
cd camusat-talk
```

## Lancer l'application

**Option 1 — ouverture directe**
Double-clique sur `index.html`, ou ouvre-le depuis ton navigateur (`Fichier > Ouvrir`). Tu verras l'interface avec des listes vides (aucune donnée d'exemple n'est incluse) ; l'enregistrement des propositions et les exports ne seront pas fonctionnels dans ce mode.

**Option 2 — via un petit serveur local** (recommandé pour éviter les restrictions de sécurité de certains navigateurs sur les fichiers locaux) :

```bash
# Avec Python 3 (déjà installé sur la plupart des systèmes)
python3 -m http.server 8000
# puis ouvrir http://localhost:8000 dans le navigateur

# ou, avec Node.js installé
npx serve .
```

**Option 3 — fonctionnalités complètes (base de données, export, administration)**
Republie `index.html` comme Artifact depuis une session Claude (Claude App / claude.ai) avec les capacités `db`, `downloads` et `user` déclarées — c'est l'environnement pour lequel l'application a été conçue.

## Structure du projet

```
camusat-talk/
├── index.html              # Application complète (HTML + CSS + JS)
├── README.md                # Ce fichier
├── .gitignore                # Adapté à un usage React/Next.js si le projet évolue
└── docs/
    └── screenshots/          # Captures d'écran de référence (accueil, formulaire, présentateurs, dashboard)
```

## Compte administrateur (Dashboard)

Le Dashboard est protégé par un code d'accès défini dans `index.html` (constante `ADMIN_CODE`, valeur par défaut : `CAMUSAT2026`). Ce n'est **pas** un mécanisme de sécurité réel — comme tout code présent dans une page web, il est visible par quiconque consulte le source. Avant de partager largement l'application ou ce dépôt :

- change la valeur de `ADMIN_CODE`, et
- privilégie, quand c'est possible, les droits de partage natifs de l'Artifact Claude (« Peut modifier ») plutôt que ce code, qui n'est qu'une commodité d'interface.

Aucune clé API, jeton ou identifiant réel n'est présent dans ce dépôt, et aucune donnée d'exemple n'est plus embarquée dans le code : l'application démarre sur une base vide.

## Limites connues / pistes d'évolution

- Pas de suite de tests automatisés.
- Pas de vraie authentification par utilisateur (l'identité du présentateur est simplement saisie dans le formulaire).
- Le contrôle d'accès au Dashboard repose sur un code partagé, pas sur des comptes individuels.
- Pour une utilisation à grande échelle en dehors de Claude Artifact, une migration vers un backend dédié (Supabase, Firebase, API interne Camusat…) est recommandée.

## Captures d'écran

| Accueil | Proposer un sujet |
|---|---|
| ![Accueil](docs/screenshots/accueil.png) | ![Formulaire](docs/screenshots/proposer-un-sujet.png) |

| Présentateurs | Dashboard |
|---|---|
| ![Présentateurs](docs/screenshots/presentateurs.png) | ![Dashboard](docs/screenshots/dashboard.png) |

---

Projet interne — Camusat Sénégal, Bureau d'Études.
