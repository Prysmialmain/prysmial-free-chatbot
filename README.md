<div align="center">

# Prysmial Chat

**Votre interface de chat IA, avec les meilleurs modèles gratuits. Une seule page, zéro installation.**

Discutez avec des modèles d'IA gratuits (Llama, Qwen, Nemotron, GPT-OSS, Hermes...)
via [OpenRouter](https://openrouter.ai). À utiliser en ligne ou chez vous.

### 👉 [chat.prysmial.com](https://chat.prysmial.com)

[Utilisation](#utilisation) · [Obtenir une clé](#obtenir-une-clé-openrouter) · [Outils de codage](#pour-les-outils-de-codage) · [Confidentialité](#confidentialité)

</div>

---

## Présentation

Une interface de chat (comme ChatGPT) tenant dans **un seul fichier `index.html`**. Vous
choisissez un modèle gratuit par catégorie, et vous discutez. Les réponses s'affichent au
fur et à mesure.

- **Zéro installation** : pas de Python, pas de Node, pas de terminal, pas de build.
- **Zéro dépendance** : HTML, CSS et JavaScript natifs.
- **Gratuit** : uniquement des modèles `:free` d'OpenRouter.
- **Privé** : votre clé et vos conversations restent dans votre navigateur.
- **Portable** : fonctionne sur ordinateur et sur mobile.

## Utilisation

1. Ouvrez **[chat.prysmial.com](https://chat.prysmial.com)** (ou téléchargez `index.html` et ouvrez-le dans votre navigateur).
2. Créez une clé gratuite sur OpenRouter ([voir ci-dessous](#obtenir-une-clé-openrouter)) et collez-la.
3. Choisissez une catégorie et un modèle, puis écrivez votre message.

Pour un premier essai, choisissez *Chatbot général → gpt-oss-20b* : c'est l'un des modèles
gratuits les plus disponibles.

## Fonctionnalités

- Sélection du modèle par **catégorie** : Coding, Chatbot général, Raisonnement, Créatif.
- Réponses en **streaming** et mise en forme **Markdown** (code, gras, listes).
- **Historique de conversations** dans une barre latérale, avec reprise.
- Option **« Conserver l'historique localement »** (sauvegarde dans le navigateur).
- **Dossier optionnel** (Chrome, Edge, Brave) : une conversation = un fichier `.json` sur votre disque.
- Clé mémorisée localement, bouton **Déconnexion**, et fenêtre **Confidentialité** intégrée.

## Obtenir une clé OpenRouter

Un guide illustré est inclus : cliquez sur **« Comment obtenir une clé gratuite ? »** sur
l'écran d'accueil, ou ouvrez `guide.html`. En résumé :

1. Créez un compte sur [openrouter.ai](https://openrouter.ai) (Google, GitHub ou e-mail).
2. Ouvrez votre espace de travail (**Personal → Workspaces**).
3. Allez dans **API Keys**.
4. Cliquez **+ New Key**, donnez un nom, mettez la limite de crédit à **0**, puis **Create**.
5. **Copiez la clé** (affichée une seule fois) et collez-la dans Prysmial Chat.

**Faut-il payer ?** Non. Les modèles gratuits (suffixe `:free`) ne consomment aucun crédit.
Mettre la limite à **0** est recommandé : la clé ne pourra jamais dépenser d'argent.

> Les modèles gratuits ont des quotas quotidiens et sont parfois saturés (erreur 429).
> Dans ce cas, réessayez plus tard ou changez de modèle.

## Pour les outils de codage

Si vous utilisez un assistant de codage (Claude Code, Cursor, etc.), fournissez-lui ce
**prompt unique** pour installer et lancer le chatbot chez vous :

```text
Télécharge ces deux fichiers et place-les dans un même dossier local :
- https://raw.githubusercontent.com/Prysmialmain/prysmial-free-chatbot/main/index.html
- https://raw.githubusercontent.com/Prysmialmain/prysmial-free-chatbot/main/guide.html
Puis ouvre index.html dans mon navigateur par défaut.
C'est une page web autonome (HTML/CSS/JS, aucune dépendance, aucune installation, aucun
build). Ne modifie rien. Je collerai ensuite ma propre clé OpenRouter dans l'interface.
```

## Personnaliser les modèles

Les modèles sont regroupés par catégorie dans une table en haut du `<script>` de
`index.html`. Modifiez-la librement :

```js
const CATEGORIES = {
  coding:       { label: "Coding",              models: ["qwen/qwen3-coder:free", "openai/gpt-oss-120b:free"] },
  chat:         { label: "Chatbot général",     models: ["meta-llama/llama-3.3-70b-instruct:free", "openai/gpt-oss-20b:free"] },
  raisonnement: { label: "Raisonnement",        models: ["nvidia/nemotron-3-ultra-550b-a55b:free", "openai/gpt-oss-120b:free"] },
  creatif:      { label: "Créatif / rédaction", models: ["nousresearch/hermes-3-llama-3.1-405b:free"] },
};
```

La liste à jour des modèles gratuits est sur
[openrouter.ai/models](https://openrouter.ai/models?max_price=0) (identifiant terminé par `:free`).

## Confidentialité

L'application n'a **aucun serveur** et ne stocke **rien** de votre côté hébergeur :

- Votre clé reste dans votre navigateur ; elle n'est envoyée qu'à OpenRouter.
- Vos messages partent directement de votre navigateur vers OpenRouter.
- Vos conversations sont enregistrées chez vous (navigateur, ou dossier de votre choix), et
  l'historique est **désactivé par défaut**.

Le détail complet est accessible dans l'application via le lien **« Comment sont gérées mes
données ? »**.

## Fonctionnement

Appels directs à `POST https://openrouter.ai/api/v1/chat/completions` avec votre clé en
en-tête `Authorization`. L'API OpenRouter est sans état : tout l'historique de la
conversation est renvoyé à chaque message, ce qui maintient le contexte côté navigateur.

## Fichiers

```
index.html     L'application
guide.html     Guide pas à pas pour obtenir une clé OpenRouter
README.md      Ce fichier
docs/          Spécification et plan d'implémentation
```

## Licence

Libre d'utilisation.

---

<div align="center">

Offert et mis à disposition par **[prysmial.com](https://prysmial.com)**

<sub>Conseil, IA et ingénierie pour PME.</sub>

</div>
