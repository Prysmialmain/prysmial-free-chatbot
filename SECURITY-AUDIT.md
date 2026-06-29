# Security Audit — Prysmial Chat

**Date:** 2026-06-29
**Scope:** whole repo (`index.html`, `guide.html`; `docs/`, `README.md`, `.gitignore`)
**Mode:** deep (dégradé, sans CodeQL)
**Tools run:** Semgrep (p/xss, p/javascript, p/secrets, p/owasp-top-ten), revue manuelle insecure-defaults + sharp-edges, variant-analysis (balayage des sinks DOM), recherche de secrets commités
**Tools skipped:** CodeQL — binaire absent de la machine (`command -v codeql` → introuvable)

## Executive summary

Application front-end statique (HTML/CSS/JS vanilla, aucun backend, aucune
dépendance JS). La surface d'attaque est minimale par conception : pas de serveur,
clé apportée par l'utilisateur, appels directs vers OpenRouter. **Aucune faille
critique, haute ou moyenne.** Semgrep ne remonte rien (108 règles, 0 finding). La
revue manuelle relève **2 points bas (durcissement)** et **2 informatifs**, dont
un seul touche le code : une chaîne externe (nom du dossier choisi par
l'utilisateur) insérée via `innerHTML` (auto-XSS, probabilité faible). Aucun secret
n'est présent dans le dépôt. Posture globale : saine.

| Severity | Count |
|---|---|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 2 |
| Info | 2 |

## Confirmed findings

### [LOW] Nom de dossier inséré dans `innerHTML` sans échappement (auto-XSS)

- **Statut :** ✅ corrigé le 2026-06-29 (le nom est désormais inséré via
  `folderState.append(...)`, c.-à-d. un nœud texte, donc aucun HTML injecté).
- **Location:** `index.html:690`
- **Class:** DOM XSS (self-XSS)
- **Found by:** revue manuelle (sharp-edges) ; semgrep n'a pas couvert ce sink
- **Risk:** `folderState.innerHTML = '... ' + (dirHandle.name || "ok")` insère le nom
  du dossier lié directement en HTML. Sous macOS, un dossier peut contenir des
  caractères `<` et `>` (ex. `<img src=x onerror=alert(1)>`). Si un utilisateur lie
  un dossier au nom malveillant, le script s'exécute dans la page. C'est un
  **auto-XSS** : la victime doit elle-même créer puis lier un dossier piégé, et il
  n'y a ni session ni cookie à voler (la clé est déjà dans son propre navigateur).
  Impact réel donc très limité, mais c'est le seul sink non échappé du code.
- **Fix:** ne pas construire ce contenu par `innerHTML`. Mettre le libellé statique
  en HTML et le nom via `textContent` :
  ```js
  // remplacer la ligne 690 par :
  folderState.textContent = "Dossier lié : " + (dirHandle.name || "ok");
  // (et garder la pastille .ok comme élément séparé si voulu)
  ```
  ou échapper : `... + escapeHtml(dirHandle.name)`.

### [LOW] En-têtes de sécurité absents sur le déploiement hébergé

- **Statut :** ✅ corrigé le 2026-06-29 (fichier `_headers` ajouté : CSP stricte +
  `X-Frame-Options: DENY` + `X-Content-Type-Options: nosniff` + `Referrer-Policy` +
  `Permissions-Policy`. Vérifié en ligne sur chat.prysmial.com).
- **Location:** déploiement Cloudflare Pages (pas de fichier `public/_headers` / `_headers`)
- **Class:** durcissement / insecure default (manque de défense en profondeur)
- **Found by:** revue manuelle (insecure-defaults)
- **Risk:** aucune CSP, ni `X-Frame-Options`, ni `X-Content-Type-Options`. Pas de
  vol de session possible (l'app n'en a pas), mais une CSP réduirait fortement
  l'impact d'un éventuel XSS (cf. finding précédent) et empêcherait l'embarquement
  en iframe (clickjacking).
- **Fix:** ajouter un fichier `_headers` à la racine du dépôt (servi par Cloudflare
  Pages), avec une CSP adaptée aux besoins réels de l'app (OpenRouter + polices) :
  ```
  /*
    X-Content-Type-Options: nosniff
    X-Frame-Options: DENY
    Referrer-Policy: strict-origin-when-cross-origin
    Content-Security-Policy: default-src 'self'; base-uri 'self'; frame-ancestors 'none'; object-src 'none'; img-src 'self' data:; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src https://fonts.gstatic.com; script-src 'self' 'unsafe-inline'; connect-src https://openrouter.ai
  ```
  Note : `script-src 'unsafe-inline'` reste nécessaire tant que le JS est inline.
  Pour s'en passer (CSP plus stricte), externaliser le `<script>` dans un fichier
  `app.js` servi depuis `'self'`.

## Worth a look (lower confidence)

- **[INFO] ✅ corrigé — Polices désormais auto-hébergées.** Auparavant chargées
  depuis `fonts.googleapis.com` / `fonts.gstatic.com` (l'IP du visiteur était
  transmise à Google). Les `.woff2` Hanken Grotesk et JetBrains Mono (latin +
  latin-ext) sont maintenant servis depuis `fonts/` ; plus aucun appel tiers
  (vérifié : 0 référence Google dans le HTML en ligne). Repli `system-ui` si le
  dossier `fonts/` est absent.
- **[INFO] Clé API en `localStorage`.** Par conception : la clé reste sur la
  machine de l'utilisateur, envoyée uniquement à OpenRouter en HTTPS, jamais
  journalisée. Persistante jusqu'à « Déconnexion ». Acceptable pour une app
  100 % client ; à signaler aux utilisateurs d'ordinateurs partagés (déjà couvert
  par le bouton Déconnexion et la fenêtre Confidentialité).

## Coverage & limitations

- **Couvert :** scan de patterns Semgrep (XSS, JS, secrets, OWASP) sur le JS inline
  extrait + les deux HTML ; revue manuelle de tous les sinks DOM (`innerHTML`),
  de la gestion de la clé, de l'API fichiers (traversée de chemin via `safeName`),
  des liens `target="_blank"` (tous en `rel="noopener"`), de l'absence d'`eval` /
  `Function` / `document.write` ; recherche de secrets commités (aucun).
- **Non couvert :** analyse de flux interprocédurale (CodeQL absent) ; tests
  dynamiques (DAST) dans un vrai navigateur au-delà des vérifications manuelles
  déjà faites ; sécurité des comptes/infra Cloudflare ; conditions d'utilisation et
  traitement des données côté OpenRouter (tiers).
- **Suivi manuel recommandé :** revalider le rendu Markdown si on l'enrichit un
  jour (ajout de liens, d'images, de tables) — c'est l'endroit où un futur XSS
  pourrait apparaître. Garder `escapeHtml` en première étape de tout rendu.

## Recommended next steps

Les trois points relevés ont été corrigés le 2026-06-29 :

1. ✅ `index.html` : nom de dossier inséré via `append()` (nœud texte) au lieu d'`innerHTML`.
2. ✅ Fichier `_headers` ajouté (CSP stricte + en-têtes de sécurité), vérifié en ligne.
3. ✅ Polices auto-hébergées, plus aucun appel à Google Fonts.

Il ne reste aucune action de sécurité ouverte. Suivi recommandé : revalider le rendu
Markdown si on l'enrichit (liens, images, tables) — c'est le seul endroit où un futur
XSS pourrait apparaître.

> Un scan propre signifie que les outils n'ont rien trouvé, pas qu'il n'y a rien.
> Cela dit, pour une app sans backend, sans dépendance et à clé apportée par
> l'utilisateur, la surface résiduelle est faible et les points ci-dessus sont du
> durcissement, pas des failles exploitables à distance.
