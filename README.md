# Good Maps

Application web **mobile-first** permettant aux personnes en situation de handicap de trouver des lieux accessibles autour d'elles, avec filtrage par type de besoin et visualisation cartographique.

**Dépôt :** [github.com/Jeff-Leote/Good-Maps](https://github.com/Jeff-Leote/Good-Maps)

---

## Sommaire

- [Présentation](#présentation)
- [Fonctionnalités](#fonctionnalités)
- [Parcours utilisateur](#parcours-utilisateur)
- [Stack technique](#stack-technique)
- [Architecture du projet](#architecture-du-projet)
- [Installation et lancement](#installation-et-lancement)
- [Scripts disponibles](#scripts-disponibles)
- [Légende des marqueurs](#légende-des-marqueurs)
- [Données OpenStreetMap](#données-openstreetmap)
- [Sécurité](#sécurité)
- [Maintenabilité](#maintenabilité)
- [Méthodologie IA](#méthodologie-ia)
- [Licence](#licence)

---

## Présentation

Good Maps répond à une problématique concrète : les personnes en situation de handicap manquent d'un outil simple pour identifier les lieux accessibles à leurs besoins spécifiques (fauteuil roulant, malvoyant, malentendant, etc.).

L'application permet de :

- Déclarer ses **besoins d'accessibilité** (sélection multiple)
- Indiquer une **ville de recherche** ou utiliser la **géolocalisation**
- Visualiser sur une **carte interactive** les lieux proches filtrés par catégorie
- Consulter pour chaque lieu un **statut d'accessibilité détaillé** (PMR, boucle magnétique, guidage sonore, etc.)
- **Réserver**, **appeler** ou **consulter l'itinéraire** directement depuis la fiche lieu

> Aucune clé API requise. Toutes les données cartographiques proviennent d'[OpenStreetMap](https://www.openstreetmap.org/) (licence ODbL).

---

## Fonctionnalités

### Écran d'accueil (Splash)

- Logo animé Good Maps affiché **2,5 secondes** avant redirection automatique vers le formulaire de profil.

### Profil utilisateur

- **Ville de recherche** (optionnelle) — la carte s'ouvre directement sur cette ville ; sans ville, la géolocalisation ou Paris est utilisé en fallback.
- **Besoins d'accessibilité** (obligatoire, sélection multiple) :
  - Fauteuil roulant (accès PMR, rampes, ascenseurs)
  - Malvoyant / Aveugle (guidage sonore, braille)
  - Malentendant / Sourd (boucle magnétique, LSF)
  - Handicap cognitif (signalétique simplifiée)
  - Mobilité réduite (espaces larges, pas de marches)
  - Toilettes adaptées (WC accessibles PMR)
- Bouton **Carte** pour accéder à la carte une fois au moins un besoin sélectionné.

### Carte interactive

- Tuiles **OpenStreetMap** via Leaflet, chargé dynamiquement côté client (`ssr: false`).
- **12 catégories** filtrables : Tout, Restos, Cafés, Bars, Hôtels, Musées, Cinémas, Théâtres, Pharmacies, Toilettes, Supermarchés, Hôpitaux.
- **Marqueurs colorés** selon l'accessibilité du lieu par rapport aux besoins déclarés (voir [légende](#légende-des-marqueurs)).
- **Barre de recherche** par ville ou adresse.
- Bouton **Suggestions** pour lancer une recherche autour de la position GPS ou de la ville du profil.
- Bouton retour vers le formulaire de profil pour modifier ses besoins.

### Fiche lieu

- Nom, adresse, type de lieu et statut ouvert/fermé.
- **Badges d'accessibilité** contextuels selon les besoins sélectionnés.
- Actions rapides : **Réserver** (site web), **Appeler** (téléphone), **Carte** (itinéraire OpenStreetMap).
- Mention des données manquantes en orange (non renseigné).

### Interface

- Design **mobile-first** (cadre 430 px sur desktop).
- Thème principal **vert** (`#16a34a`) pour les boutons, sélections et éléments d'action.
- Libellés de boutons courts et explicites pour une navigation claire.

---

## Parcours utilisateur

```
SplashScreen (2,5 s auto)
        ↓
ProfileForm (ville + besoins d'accessibilité)
        ↓  [Carte]
     MapView (carte + filtres + recherche)
        ↑
        └── onBack → retour au profil
```

---

## Stack technique

| Technologie | Version | Rôle |
|---|---|---|
| [Next.js](https://nextjs.org/) | 14 (App Router) | Framework React, routing, SSR |
| [TypeScript](https://www.typescriptlang.org/) | 5 | Typage statique |
| [Tailwind CSS](https://tailwindcss.com/) | 3.4 | Stylisation utilitaire responsive |
| [Leaflet](https://leafletjs.com/) | 1.9 | Carte interactive |
| [Nominatim API](https://nominatim.org/) | — | Géocodage et recherche de POI (OSM) |

---

## Architecture du projet

```
Good-Maps/
├── app/
│   ├── components/
│   │   ├── GoodMapsLogo.tsx    # Logo SVG réutilisable (3 tailles : sm, md, lg)
│   │   ├── SplashScreen.tsx    # Écran de démarrage (2,5 s)
│   │   ├── ProfileForm.tsx     # Formulaire profil (ville + besoins)
│   │   ├── MapView.tsx         # Carte Leaflet, marqueurs, filtres, recherche
│   │   └── PlaceDetail.tsx     # Fiche détail d'un lieu (bottom sheet)
│   ├── lib/
│   │   └── overpass.ts         # Appels Nominatim, parsing OSM, catégories
│   ├── types.ts                # Interface AccessiblePlace
│   ├── page.tsx                # Orchestration des 3 écrans
│   ├── layout.tsx              # Layout racine, métadonnées, viewport mobile
│   └── globals.css             # Variables CSS, styles Leaflet, conteneur app
├── next.config.mjs
├── tailwind.config.ts          # Couleur primary (#16a34a), police Inter
├── tsconfig.json
├── postcss.config.mjs
├── METHODOLOGY.md              # Documentation méthodologie IA (EPSI)
└── package.json
```

### Fichiers clés

| Fichier | Responsabilité |
|---|---|
| `app/page.tsx` | Gestion de l'état global (`splash` / `profile` / `map`) et du profil utilisateur |
| `app/lib/overpass.ts` | Requêtes Nominatim, parsing des tags OSM (`wheelchair`, `hearing_loop`, etc.) |
| `app/components/MapView.tsx` | Initialisation Leaflet, calcul couleur des marqueurs, géolocalisation |
| `app/components/PlaceDetail.tsx` | Badges d'accessibilité contextuels et actions (réservation, appel, itinéraire) |
| `app/components/ProfileForm.tsx` | Sélection des besoins et ville, soumission du profil |

---

## Installation et lancement

### Prérequis

- **Node.js** ≥ 18.16
- **npm** (inclus avec Node.js)
- Aucune variable d'environnement ni clé API requise

### Étapes

```bash
# Cloner le dépôt
git clone https://github.com/Jeff-Leote/Good-Maps.git
cd Good-Maps

# Installer les dépendances
npm install

# Lancer le serveur de développement
npm run dev
```

Ouvrir [http://localhost:3000](http://localhost:3000) dans un navigateur (de préférence en mode mobile ou avec les outils de développement).

### Build de production

```bash
npm run build
npm start
```

L'application sera disponible sur [http://localhost:3000](http://localhost:3000).

---

## Scripts disponibles

| Commande | Description |
|---|---|
| `npm run dev` | Serveur de développement avec rechargement à chaud |
| `npm run build` | Compilation optimisée pour la production |
| `npm start` | Lance le serveur de production (après `build`) |
| `npm run lint` | Vérification ESLint (config Next.js) |

---

## Légende des marqueurs

Les couleurs des marqueurs sont calculées dynamiquement en fonction des besoins d'accessibilité sélectionnés et des tags OpenStreetMap du lieu :

| Couleur | Signification |
|---|---|
| Vert | Tous les critères satisfaits |
| Orange | Partiellement accessible |
| Gris | Non accessible |
| Rouge | Données manquantes / non renseigné |

---

## Données OpenStreetMap

Les données d'accessibilité proviennent des tags OSM contribués par la communauté :

- `wheelchair` — accès fauteuil roulant
- `tactile_paving` — guidage au sol pour malvoyants
- `hearing_loop` / `induction_loop` — boucle magnétique
- `toilets:wheelchair` — toilettes PMR
- `opening_hours`, `phone`, `website` — informations pratiques

La qualité et la complétude de ces données **varient selon les villes et les contributeurs**. Vérifiez les informations importantes avant tout déplacement.

> Contribuez à [OpenStreetMap](https://www.openstreetmap.org/) pour améliorer les données d'accessibilité de votre ville !

---

## Sécurité

- **Aucune clé API** : Nominatim est public, aucun secret à protéger
- **Pas de base de données** : application stateless, aucune donnée utilisateur persistée côté serveur
- **Sanitisation des entrées** : toutes les saisies passent par `encodeURIComponent` avant les appels fetch
- **Liens externes sécurisés** : `rel="noopener noreferrer"` sur tous les `<a target="_blank">`
- **Pas d'injection HTML** : les variables injectées dans les SVG Leaflet proviennent uniquement du code interne (couleurs calculées), jamais de l'utilisateur

---

## Maintenabilité

- **Séparation des responsabilités** : `lib/` (API), `components/` (UI), `types.ts` (interfaces)
- **TypeScript strict** : props et données API entièrement typées
- **Ajout d'une catégorie** : modifier le tableau `CATEGORIES` dans `app/lib/overpass.ts`
- **Ajout d'un besoin d'accessibilité** : ajouter l'option dans `ProfileForm.tsx`, la logique badge dans `PlaceDetail.tsx` et le score dans `MapView.tsx`
- **Personnalisation du thème** : modifier `primary` et `primary-dark` dans `tailwind.config.ts` et `app/globals.css`
- **Zéro dépendance propriétaire** : Next.js, Tailwind, Leaflet — toutes open-source

---

## Méthodologie IA

Ce projet a été développé avec l'aide de l'IA générative dans le cadre du cours **"Coder avec l'IA Générative"** à l'EPSI.

Consultez [METHODOLOGY.md](./METHODOLOGY.md) pour le détail du préprompt, des outils utilisés et des décisions d'architecture.

---

## Licence

Projet réalisé dans le cadre du cours **"Coder avec l'IA Générative"** à l'EPSI.

Données cartographiques : © [OpenStreetMap contributors](https://www.openstreetmap.org/copyright) (ODbL).
