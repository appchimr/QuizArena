# QuizArena — Installation (Firebase + hébergement)

Ce dossier contient une version autonome de QuizArena (`index.html`) qui ne dépend plus de
l'environnement Claude : elle utilise **Firebase Realtime Database** pour synchroniser l'hôte
et les joueurs, et peut être hébergée n'importe où (GitHub Pages, Vercel, Netlify...).

Aucune étape de build n'est nécessaire : React et Babel sont chargés depuis un CDN, le JSX est
transformé directement dans le navigateur.

## 1. Créer un projet Firebase (gratuit)

1. Va sur https://console.firebase.google.com et clique sur **"Ajouter un projet"**.
2. Donne-lui un nom (ex. `quizarena`), continue avec les options par défaut.
3. Une fois le projet créé, dans le menu de gauche, va dans **Compilation > Realtime Database**.
4. Clique sur **"Créer une base de données"**.
   - Choisis une région (Europe de préférence si tu es en France).
   - Pour démarrer rapidement, choisis le mode **test** (règles ouvertes 30 jours). Tu pourras
     resserrer les règles ensuite (voir section 4).
5. Toujours dans la console, va dans **Paramètres du projet** (icône ⚙️) > **Général**.
6. Descends jusqu'à **"Vos applications"**, clique sur l'icône **Web `</>`**, donne un nom à
   l'application, puis clique sur **"Enregistrer l'application"**.
7. Firebase affiche un bloc de configuration qui ressemble à ceci :

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "quizarena-xxxxx.firebaseapp.com",
  databaseURL: "https://quizarena-xxxxx-default-rtdb.firebaseio.com",
  projectId: "quizarena-xxxxx",
  ...
};
```

Copie ces 4 valeurs (`apiKey`, `authDomain`, `databaseURL`, `projectId`).

## 2. Coller la config dans le code

Ouvre `index.html`, cherche le bloc `FIREBASE_CONFIG` tout en haut du `<script type="text/babel">`,
et remplace les valeurs `"VOTRE_..."` par celles copiées à l'étape précédente :

```js
const FIREBASE_CONFIG = {
  apiKey: "AIzaSy...",
  authDomain: "quizarena-xxxxx.firebaseapp.com",
  databaseURL: "https://quizarena-xxxxx-default-rtdb.firebaseio.com",
  projectId: "quizarena-xxxxx",
};
```

Enregistre le fichier.

## 3. Héberger sur GitHub Pages

1. Crée un nouveau repository GitHub (public ou privé).
2. Mets `index.html` à la racine du repo (commit + push).
3. Dans le repo, va dans **Settings > Pages**.
4. Sous "Build and deployment", choisis **Source : Deploy from a branch**, branche `main`,
   dossier `/ (root)`.
5. Après une minute ou deux, ton site est en ligne à une adresse du type :
   `https://tonpseudo.github.io/nom-du-repo/`

C'est cette URL que tu partages avec tes joueurs (et celle que le QR code encode automatiquement).

## 4. Sécuriser les règles Firebase (recommandé avant un usage répété)

Les règles "mode test" laissent n'importe qui lire/écrire toute la base, sans limite de temps
après 30 jours (et c'est risqué dès le premier jour si l'URL de la base venait à fuiter).

Dans la console Firebase > Realtime Database > onglet **Règles**, un réglage simple et
suffisant pour ce projet (tout le monde peut lire/écrire uniquement sous `rooms/`) :

```json
{
  "rules": {
    "rooms": {
      ".read": true,
      ".write": true
    },
    "library": {
      ".read": true,
      ".write": true
    }
  }
}
```

Le chemin `library/` contient tous les quiz enregistrés par les organisateurs (visibles par
quiconque connaît le code organisateur, puisqu'il n'y a pas de compte utilisateur séparé).

Cela limite l'accès à la partie du jeu et bloque le reste de la base par défaut. Une vraie
authentification (comptes utilisateurs) serait plus sûre mais demande plus de travail ; pour un
usage entre collègues/amis, cette règle est un compromis raisonnable.

## 5. Changer le code organisateur

Le code qui débloque l'éditeur de quiz (`HOST_PIN`) est en clair dans le fichier — pratique pour
toi, mais visible par quiconque inspecte le code source. Pour le changer, cherche cette ligne :

```js
const HOST_PIN = "Qual2026*";
```

et remplace la valeur.

## Limites à connaître

- **Bibliothèque de quiz** : enregistrée dans Firebase, donc accessible depuis n'importe quel
  appareil dès lors qu'on connaît le code organisateur (pas de compte séparé par personne — tous
  les organisateurs partagent la même bibliothèque).
- **Plan gratuit Firebase (Spark)** : largement suffisant pour des quiz ponctuels entre
  collègues — la limite porte sur le volume de données transférées par jour, très généreuse
  pour ce type d'usage.
- **QR code** : il encode l'URL de la page actuelle + le code de la salle. Assure-toi de tester
  le lien une fois le site en ligne (et pas seulement en local) avant un évènement important.
