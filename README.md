<div align="center">

# Prysmial Chat

**Ton interface de chat IA, avec les meilleurs modèles gratuits. Un seul fichier, zéro installation.**

Discute avec des modèles d'IA gratuits (Llama, Qwen, Nemotron, GPT-OSS, Hermes...)
via [OpenRouter](https://openrouter.ai), depuis une page web que tu ouvres en
double-cliquant. Pas de compte chez nous, pas de serveur, pas de build.

[Démarrer](#-démarrage-en-3-minutes) · [Obtenir une clé](#-obtenir-une-clé-openrouter) · [Personnaliser](#-personnaliser-les-modèles) · [Confidentialité](#-confidentialité-et-sécurité)

</div>

---

## Qu'est-ce que c'est

Une mini application de chat tenant dans **un seul fichier `index.html`**. Tu l'ouvres
dans ton navigateur, tu colles ta propre clé OpenRouter (gratuite), tu choisis un
modèle par catégorie, et tu discutes. Les réponses s'affichent en streaming.

- **Zéro installation** : pas de Python, pas de Node, pas de terminal, pas de build.
- **Zéro dépendance** : HTML, CSS et JavaScript natifs.
- **Gratuit** : uniquement des modèles `:free` d'OpenRouter.
- **Privé** : ta clé reste dans ton navigateur, les conversations ne quittent pas ta machine (sauf l'appel au modèle).
- **Portable** : marche sur Mac, Windows, Linux, et sur mobile.

## Fonctionnalités

- Sélection du modèle par **catégorie** : Coding, Chatbot général, Raisonnement, Créatif.
- **Streaming** des réponses (le texte s'écrit au fur et à mesure).
- **Rendu Markdown léger** : gras, italique, code en ligne, blocs de code, listes.
- Clé API mémorisée dans le navigateur (`localStorage`), avec bouton **Effacer ma clé**.
- Bouton **Nouvelle conversation** pour repartir d'un contexte vide.
- **Guide intégré** pas à pas pour créer sa clé.
- Gestion d'erreurs claire (clé invalide, modèle saturé, réseau).

## 🚀 Démarrage en 3 minutes

1. **Récupère le fichier.** Télécharge `index.html` (et `guide.html` à côté), ou clone le dépôt :
   ```bash
   git clone https://github.com/Prysmialmain/prysmial-free-chatbot.git
   ```
2. **Ouvre `index.html`** dans ton navigateur (double-clic).
3. **Crée une clé gratuite** sur OpenRouter (voir ci-dessous) et colle-la.
4. **Choisis** une catégorie et un modèle, puis écris ton message.

> Astuce : pour ton tout premier essai, prends *Chatbot général → gpt-oss-20b*.
> C'est l'un des modèles gratuits les plus disponibles.

## 🔑 Obtenir une clé OpenRouter

Le guide complet et illustré est inclus dans l'app : ouvre `guide.html`, ou clique
sur **« Comment obtenir une clé gratuite ? »** depuis l'écran d'accueil. En résumé :

1. Crée un compte sur [openrouter.ai](https://openrouter.ai) (Google, GitHub ou e-mail).
2. Ouvre ton espace de travail (profil **Personal → Workspaces**).
3. Va dans le menu **API Keys**.
4. Clique **+ New Key**, donne un nom, mets la limite de crédit à **0**, puis **Create**.
5. **Copie la clé** (affichée une seule fois) et colle-la dans Prysmial Chat.

**Faut-il payer ?** Non. Les modèles gratuits (suffixe `:free`) ne consomment
aucun crédit. Mettre la limite de crédit à **0** est même recommandé : la clé ne
pourra jamais dépenser d'argent par accident.

> Note : les modèles gratuits ont des quotas quotidiens et sont parfois saturés
> (erreur 429). Dans ce cas, réessaie un peu plus tard ou change de modèle.

## 🎛 Personnaliser les modèles

Les modèles sont regroupés par catégorie dans une table en haut du `<script>` de
`index.html`. Édite-la librement :

```js
const CATEGORIES = {
  coding:       { label: "Coding",              models: ["qwen/qwen3-coder:free", "openai/gpt-oss-120b:free"] },
  chat:         { label: "Chatbot général",     models: ["meta-llama/llama-3.3-70b-instruct:free", "openai/gpt-oss-20b:free"] },
  raisonnement: { label: "Raisonnement",        models: ["nvidia/nemotron-3-ultra-550b-a55b:free", "openai/gpt-oss-120b:free"] },
  creatif:      { label: "Créatif / rédaction", models: ["nousresearch/hermes-3-llama-3.1-405b:free"] },
};
```

La liste à jour des modèles gratuits est disponible sur
[openrouter.ai/models](https://openrouter.ai/models?max_price=0). Un modèle est
gratuit quand son identifiant finit par `:free`.

## 🔒 Confidentialité et sécurité

- **Ta clé reste chez toi.** Elle est stockée dans le `localStorage` de ton
  navigateur et n'est envoyée qu'à l'API d'OpenRouter. Aucun serveur intermédiaire.
- **Aucune clé dans ce dépôt.** Chacun utilise la sienne.
- **Conversations non sauvegardées** : elles vivent en mémoire et disparaissent au
  rechargement de la page. (Une option d'historique local est prévue, voir plus bas.)
- **Anti-injection** : les réponses des modèles sont échappées avant rendu, pour
  qu'un contenu malicieux ne puisse pas exécuter de code dans la page.
- Sur un ordinateur partagé, utilise le bouton **Effacer ma clé** en partant.

## 🛠 Comment ça marche

- `index.html` contient tout : structure, style (DA Prysmial), et logique.
- Appels directs à `POST https://openrouter.ai/api/v1/chat/completions` avec
  `Authorization: Bearer <ta-clé>` et `stream: true`.
- L'API OpenRouter est **sans état** : tout l'historique de la conversation est
  renvoyé à chaque message. C'est pour cela que le contexte est « gardé » côté client.

## 🗺 À venir

- [ ] Option **« Conserver l'historique localement »** (case à cocher).
- [ ] **Sidebar des conversations** (comme les grandes IA), avec reprise.
- [ ] Choix du **répertoire** de lecture/écriture pour l'historique.

## Fichiers

```
index.html     L'application
guide.html     Guide pas à pas pour obtenir une clé OpenRouter
README.md      Ce fichier
docs/          Spécification et plan d'implémentation
```

## Licence

Libre d'utilisation. Fais-en ce que tu veux.

---

<div align="center">

Offert et mis à disposition par **[prysmial.com](https://prysmial.com)**

<sub>Conseil, IA et ingénierie pour PME.</sub>

</div>
