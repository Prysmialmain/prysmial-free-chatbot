# Prysmial Chat

Un mini chat IA dans **un seul fichier**. Aucune installation, aucun compte à créer chez nous.
Tu utilises des modèles **gratuits** via [OpenRouter](https://openrouter.ai), avec ta propre clé.

## Utilisation

1. Télécharge `index.html` (ou ouvre la page hébergée).
2. Ouvre-le dans ton navigateur (double-clic).
3. Crée une clé gratuite sur https://openrouter.ai/keys et colle-la.
4. Choisis une catégorie et un modèle, puis discute.

Ta clé reste **dans ton navigateur** (localStorage). Elle n'est jamais envoyée
ailleurs qu'à OpenRouter. Le bouton « Effacer ma clé » la retire.
Les conversations ne sont pas sauvegardées : elles disparaissent au rechargement.

## Catégories

Une sélection de modèles gratuits, classés en quatre catégories :

- **Coding** : qwen3-coder, gpt-oss-120b
- **Chatbot général** : llama-3.3-70b, qwen3-next, gpt-oss-20b
- **Raisonnement** : nemotron ultra, nemotron super, gpt-oss-120b
- **Créatif / rédaction** : hermes-3-405b, llama-3.3-70b

Pour ajouter ou retirer des modèles, édite la table `CATEGORIES` en haut du
`<script>` dans `index.html`.

> Les modèles gratuits sont parfois saturés (erreur 429). Réessaie dans un
> moment ou choisis un autre modèle.

## Technique

HTML + CSS + JavaScript vanilla, zéro dépendance, zéro build. Les réponses sont
affichées en streaming, avec un rendu Markdown léger. L'API OpenRouter est sans
état : tout l'historique de la conversation est renvoyé à chaque message.

---

Offert et mis à disposition par [prysmial.com](https://prysmial.com).
