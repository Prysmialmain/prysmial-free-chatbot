# Design — OpenRouter Chat Launcher (single-file `index.html`)

Date: 2026-06-29

## 1. Objectif

Un mini chat OpenRouter **lançable bêtement** : un repo public où n'importe qui ouvre
un seul fichier `index.html` (ou un lien GitHub Pages), colle **sa propre** clé
OpenRouter, choisit un modèle gratuit par catégorie, et discute. Aucune
installation, pas de Python, pas de terminal, pas de build. Marche sur Mac,
Windows et mobile.

## 2. Contraintes & décisions

- **Zéro dépendance, zéro build** : HTML + CSS + JS vanilla dans un seul fichier.
- **Pas de clé dans le repo** : chaque utilisateur fournit la sienne au runtime.
- **Clé conservée dans `localStorage`** du navigateur de l'utilisateur (pour ne pas
  la retaper), avec un bouton **« Effacer ma clé »**.
- **Pas de persistance des conversations** : l'historique vit en mémoire (variable JS)
  et est perdu au rechargement. L'API OpenRouter est *stateless* — on renvoie tout
  l'historique à chaque appel.
- **Réponses en streaming** (SSE) + **rendu Markdown léger** (gras, italique, code
  inline, blocs de code, listes), sans librairie externe.
- **UI en français**, sobre, responsive, thème suivant le système (clair/sombre).
- **Liste de modèles curée en dur** (IDs vérifiés contre l'API le 2026-06-29).

## 3. Catégories & modèles (curés)

Table JS éditable en haut du fichier. IDs vérifiés existants et `:free` :

```js
const CATEGORIES = {
  coding: {
    label: "Coding",
    models: [
      "qwen/qwen3-coder:free",
      "openai/gpt-oss-120b:free",
    ],
  },
  chat: {
    label: "Chatbot général",
    models: [
      "meta-llama/llama-3.3-70b-instruct:free",
      "qwen/qwen3-next-80b-a3b-instruct:free",
      "openai/gpt-oss-20b:free",
    ],
  },
  raisonnement: {
    label: "Raisonnement",
    models: [
      "nvidia/nemotron-3-ultra-550b-a55b:free",
      "nvidia/nemotron-3-super-120b-a12b:free",
      "openai/gpt-oss-120b:free",
    ],
  },
  creatif: {
    label: "Créatif / rédaction",
    models: [
      "nousresearch/hermes-3-llama-3.1-405b:free",
      "meta-llama/llama-3.3-70b-instruct:free",
    ],
  },
};
```

Note : certains modèles gratuits sont régulièrement **rate-limités upstream (429)** ;
c'est pourquoi chaque catégorie propose plusieurs modèles de repli.

## 4. Écrans & flux utilisateur

1. **Ouverture** : si aucune clé en `localStorage`, afficher un panneau de saisie de
   clé avec un lien vers `https://openrouter.ai/keys`. Sinon, aller au chat.
2. **Sélection** : menu **Catégorie** → menu **Modèle** (filtré sur la catégorie).
3. **Chat** : liste de messages (utilisateur / assistant) + champ de saisie + bouton
   Envoyer (Entrée pour envoyer, Maj+Entrée pour saut de ligne).
4. **Streaming** : la réponse de l'assistant s'affiche au fur et à mesure.
5. **Nouvelle conversation** : bouton qui vide l'historique en mémoire.
6. **Changement de modèle/catégorie en cours** : **on garde le fil** ; seul le bouton
   « Nouvelle conversation » réinitialise le contexte.
7. **Effacer ma clé** : bouton qui supprime la clé du `localStorage` et revient à
   l'écran de saisie.

## 5. Appels API

- Endpoint : `POST https://openrouter.ai/api/v1/chat/completions`
- Header : `Authorization: Bearer <clé>`, `Content-Type: application/json`
- Body : `{ model, messages, stream: true }` où `messages` inclut un message system
  en français + tout l'historique.
- Streaming : lecture du flux SSE (`data: {...}` lignes, `[DONE]` en fin), concaténation
  des `choices[0].delta.content`.

### Gestion d'erreurs (explicite)

| Cas | Détection | Message UI |
| --- | --- | --- |
| Clé invalide / absente | HTTP 401 (`User not found`) | « Clé invalide — vérifie ta clé OpenRouter. » + bouton revenir à la saisie |
| Modèle saturé | HTTP 429 | « Ce modèle gratuit est saturé pour l'instant. Réessaie ou choisis un autre modèle. » |
| Réseau / autre | exception / autre code | « Erreur réseau. Réessaie. » + détail |

## 6. Structure du fichier `index.html`

Un seul fichier, organisé en sections :

- `<style>` : thème (variables CSS clair/sombre), layout (header de sélection, zone
  messages, barre de saisie).
- HTML : panneau clé, header (catégorie + modèle + boutons Nouvelle conv / Effacer clé),
  liste messages, barre de saisie.
- `<script>` :
  - `CATEGORIES` (table curée).
  - État : `apiKey`, `currentModel`, `messages` (en mémoire).
  - `localStorage` : get/set/clear de la clé.
  - `renderMarkdownLight(text)` : conversion Markdown minimale → HTML échappé.
  - `sendMessage()` : construit le body, appelle l'API en streaming, met à jour l'UI.
  - `handleError(status, payload)` : mappe les cas du tableau ci-dessus.
  - Câblage des événements (selects, boutons, Entrée).

Unités à responsabilité unique, testables/raisonnables isolément :
gestion clé, rendu Markdown, appel streaming, gestion d'erreurs, rendu UI.

## 7. Sécurité

- Le rendu Markdown **échappe le HTML** avant transformation (pas d'injection via les
  réponses du modèle).
- La clé n'est jamais envoyée ailleurs qu'à `openrouter.ai`.
- `.env` (clé locale de test) reste **gitignoré** ; aucune clé dans le repo public.

## 8. Repo

App la plus légère possible : un fichier livrable + un README + un .gitignore.

```
/
├── index.html          # le livrable (tout dedans)
├── README.md           # "ouvre index.html, colle ta clé, choisis un modèle, chatte"
└── .gitignore          # .DS_Store (+ .env/.venv résiduels ignorés par sécurité)
```

Tout le code Python actuel est **supprimé** (`ask.py`, `compare_models.py`,
`config.py`, `models.py`, `list_free_models.py`, `test_nemotron.py`,
`requirements.txt`, `.env`, `.env.example`, `.venv/`, `__pycache__/`) : il ne sert
plus le but. Le repo ne contient que ce qui fait tourner l'app.

## 8b. Ajout v1.1 — Historique local (livré)

- Sidebar de conversations (toujours visible ; tiroir sur mobile), reprise au clic, suppression, titre auto.
- Case « Conserver l'historique localement » : persistance navigateur via **IndexedDB**.
- Bouton « Lier un dossier » (File System Access API, Chromium) : un fichier `.json`
  lisible par conversation ; repli **Export/Import** `.json` sur les autres navigateurs.
- Page guide `guide.html` (DA Prysmial) liée depuis l'écran d'accueil pour obtenir une clé.
- Crédit « Offert et mis à disposition par prysmial.com » en pied de page.

## 9. Hors périmètre (YAGNI v1)

- ~~Persistance des conversations~~ → livrée en v1.1 (voir 8b).
- Récupération live des modèles depuis l'API (on reste curé en dur).
- Multi-conversations / onglets.
- Réglages avancés (température, max tokens, system prompt éditable).
- Exécutable double-clic / GitHub Pages auto (le fichier marche déjà en local ;
  l'hébergement est un simple bonus de déploiement, pas du code).

## 10. Critères de succès

- Ouvrir `index.html` dans un navigateur, coller une clé valide, choisir une
  catégorie + un modèle, envoyer un message, voir une réponse en streaming.
- Recharger la page : la clé est mémorisée, l'historique est vide.
- Clé invalide → message clair. Modèle 429 → message clair invitant à changer.
- « Nouvelle conversation » vide le contexte ; « Effacer ma clé » revient à la saisie.
