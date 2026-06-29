# OpenRouter Chat Launcher Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a single-file `index.html` OpenRouter chat that anyone can open with zero install, paste their own key, pick a free model by category, and chat with streaming responses.

**Architecture:** One `index.html` containing HTML + CSS + vanilla JS, no dependencies and no build step. The user's API key lives in `localStorage`; conversation history lives in a JS variable (memory only). Calls go directly from the browser to `https://openrouter.ai/api/v1/chat/completions` with SSE streaming.

**Tech Stack:** HTML5, CSS (custom properties for theming), vanilla JS (`fetch` + `ReadableStream` for SSE). No npm, no framework, no build.

## Global Constraints

- Zero runtime dependencies, no build step — everything in one `index.html`.
- No API key in the repo; each user supplies their own at runtime.
- API key stored in `localStorage` under key `or_api_key`; conversations are memory-only (lost on reload).
- All responses streamed (SSE); Markdown rendered lightly with HTML escaped first.
- UI in French; theme follows system (light/dark) via `prefers-color-scheme`.
- Final repo contains only: `index.html`, `README.md`, `.gitignore`.
- Verification is browser-based (no test framework) to honor the zero-dependency constraint.

---

### Task 0: Clean slate + repo scaffold

**Files:**
- Delete: `ask.py`, `compare_models.py`, `config.py`, `models.py`, `list_free_models.py`, `test_nemotron.py`, `requirements.txt`, `.env`, `.env.example`, `.venv/`, `__pycache__/`, `._.venv`, `.___pycache__`
- Create: `.gitignore`, `index.html` (empty placeholder), `README.md` (placeholder)

**Interfaces:**
- Produces: a clean working directory with an empty `index.html` for later tasks.

- [ ] **Step 1: Remove all Python and venv artifacts**

```bash
cd /Volumes/Tai_SSD/OpenRouter
rm -rf ask.py compare_models.py config.py models.py list_free_models.py \
       test_nemotron.py requirements.txt .env .env.example \
       .venv __pycache__ "._.venv" ".___pycache__"
```

- [ ] **Step 2: Write `.gitignore`**

```gitignore
.DS_Store
.env
.venv/
__pycache__/
*.pyc
```

- [ ] **Step 3: Create placeholder `index.html` and `README.md`**

`index.html`:
```html
<!doctype html>
<meta charset="utf-8">
<title>OpenRouter Chat</title>
<p>WIP</p>
```

`README.md`:
```markdown
# OpenRouter Chat

WIP
```

- [ ] **Step 4: Verify the directory is clean**

Run: `ls -A`
Expected: only `.gitignore`, `index.html`, `README.md`, and `docs/` remain.

- [ ] **Step 5: Initialize git and commit**

```bash
git init
git add .gitignore index.html README.md docs
git commit -m "chore: clean slate, scaffold single-file app"
```

---

### Task 1: HTML structure + CSS theme

**Files:**
- Modify: `index.html` (replace placeholder with full skeleton)

**Interfaces:**
- Produces: DOM elements with these exact ids, used by all later tasks:
  `#key-panel`, `#key-input`, `#key-save`, `#app`, `#category-select`,
  `#model-select`, `#new-conv`, `#clear-key`, `#messages`, `#prompt`, `#send`.

- [ ] **Step 1: Replace `index.html` with the full skeleton**

```html
<!doctype html>
<html lang="fr">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>OpenRouter Chat</title>
<style>
  :root {
    --bg: #ffffff; --fg: #1a1a1a; --muted: #666; --border: #ddd;
    --user-bg: #e8f0fe; --assistant-bg: #f4f4f5; --accent: #2563eb;
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #1a1a1a; --fg: #f0f0f0; --muted: #999; --border: #333;
      --user-bg: #1e3a5f; --assistant-bg: #2a2a2e; --accent: #60a5fa;
    }
  }
  * { box-sizing: border-box; }
  body { margin: 0; font: 16px/1.5 system-ui, sans-serif; background: var(--bg); color: var(--fg);
         height: 100vh; display: flex; flex-direction: column; }
  header { display: flex; gap: 8px; align-items: center; flex-wrap: wrap;
           padding: 10px; border-bottom: 1px solid var(--border); }
  select, button, input { font: inherit; padding: 6px 10px; border: 1px solid var(--border);
                          border-radius: 6px; background: var(--bg); color: var(--fg); }
  button { cursor: pointer; }
  button.primary { background: var(--accent); color: #fff; border-color: var(--accent); }
  header .spacer { flex: 1; }
  #messages { flex: 1; overflow-y: auto; padding: 16px; display: flex; flex-direction: column; gap: 12px; }
  .msg { max-width: 80%; padding: 10px 14px; border-radius: 12px; white-space: normal; word-wrap: break-word; }
  .msg.user { align-self: flex-end; background: var(--user-bg); }
  .msg.assistant { align-self: flex-start; background: var(--assistant-bg); }
  .msg pre { background: rgba(127,127,127,.15); padding: 10px; border-radius: 6px; overflow-x: auto; }
  .msg code { background: rgba(127,127,127,.15); padding: 1px 4px; border-radius: 4px; }
  .msg pre code { background: none; padding: 0; }
  .error { color: #dc2626; }
  form#composer { display: flex; gap: 8px; padding: 10px; border-top: 1px solid var(--border); }
  #prompt { flex: 1; resize: none; height: 44px; }
  #key-panel { max-width: 480px; margin: 60px auto; padding: 24px; text-align: center; }
  #key-panel input { width: 100%; margin: 12px 0; }
  .hidden { display: none !important; }
  a { color: var(--accent); }
</style>
</head>
<body>
  <section id="key-panel">
    <h1>OpenRouter Chat</h1>
    <p>Colle ta clé OpenRouter pour commencer.<br>
       <a href="https://openrouter.ai/keys" target="_blank" rel="noopener">Obtenir une clé</a></p>
    <input id="key-input" type="password" placeholder="sk-or-v1-..." autocomplete="off">
    <button id="key-save" class="primary">Enregistrer la clé</button>
  </section>

  <div id="app" class="hidden" style="display:flex; flex-direction:column; height:100%;">
    <header>
      <select id="category-select"></select>
      <select id="model-select"></select>
      <span class="spacer"></span>
      <button id="new-conv">Nouvelle conversation</button>
      <button id="clear-key">Effacer ma clé</button>
    </header>
    <div id="messages"></div>
    <form id="composer">
      <textarea id="prompt" placeholder="Écris ton message… (Entrée = envoyer, Maj+Entrée = saut de ligne)"></textarea>
      <button id="send" class="primary" type="submit">Envoyer</button>
    </form>
  </div>

  <script>
  // Tasks 2–7 fill this in.
  </script>
</body>
</html>
```

- [ ] **Step 2: Verify in browser**

Open `index.html` in a browser.
Expected: the key panel is visible (title, input, "Enregistrer la clé" button, "Obtenir une clé" link). The chat app (`#app`) is hidden. Layout adapts to light/dark system theme.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: HTML structure and CSS theme"
```

---

### Task 2: API key management (localStorage + panel toggle)

**Files:**
- Modify: `index.html` (inside `<script>`)

**Interfaces:**
- Consumes: DOM ids from Task 1.
- Produces: globals `getKey()`, `setKey(k)`, `clearKey()`, `showApp()`, `showKeyPanel()`, and a module-level `apiKey` variable other tasks read.

- [ ] **Step 1: Add key management code inside `<script>`**

```js
const STORAGE_KEY = "or_api_key";
let apiKey = localStorage.getItem(STORAGE_KEY) || "";

const keyPanel = document.getElementById("key-panel");
const app = document.getElementById("app");
const keyInput = document.getElementById("key-input");

function showApp() { keyPanel.classList.add("hidden"); app.classList.remove("hidden"); }
function showKeyPanel() { app.classList.add("hidden"); keyPanel.classList.remove("hidden"); }
function setKey(k) { apiKey = k.trim(); localStorage.setItem(STORAGE_KEY, apiKey); }
function clearKey() { apiKey = ""; localStorage.removeItem(STORAGE_KEY); keyInput.value = ""; showKeyPanel(); }

document.getElementById("key-save").addEventListener("click", () => {
  const v = keyInput.value.trim();
  if (!v) { keyInput.focus(); return; }
  setKey(v);
  showApp();
});
document.getElementById("clear-key").addEventListener("click", clearKey);

// On load: if we already have a key, go straight to the app.
if (apiKey) showApp(); else showKeyPanel();
```

- [ ] **Step 2: Verify in browser**

1. Open `index.html` → key panel shows.
2. Type any text, click "Enregistrer la clé" → app shows, panel hides.
3. Reload the page → app shows directly (key remembered).
4. Click "Effacer ma clé" → returns to key panel.
5. Reload → key panel shows (key forgotten).

Confirm via DevTools → Application → Local Storage that `or_api_key` is set/removed accordingly.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: API key management with localStorage"
```

---

### Task 3: Category and model selectors

**Files:**
- Modify: `index.html` (inside `<script>`)

**Interfaces:**
- Consumes: `#category-select`, `#model-select` from Task 1.
- Produces: global `CATEGORIES` object and `currentModel` variable; populates both selects and keeps `currentModel` in sync.

- [ ] **Step 1: Add the curated model table and selector logic**

```js
const CATEGORIES = {
  coding:       { label: "Coding",              models: ["qwen/qwen3-coder:free", "openai/gpt-oss-120b:free"] },
  chat:         { label: "Chatbot général",     models: ["meta-llama/llama-3.3-70b-instruct:free", "qwen/qwen3-next-80b-a3b-instruct:free", "openai/gpt-oss-20b:free"] },
  raisonnement: { label: "Raisonnement",        models: ["nvidia/nemotron-3-ultra-550b-a55b:free", "nvidia/nemotron-3-super-120b-a12b:free", "openai/gpt-oss-120b:free"] },
  creatif:      { label: "Créatif / rédaction", models: ["nousresearch/hermes-3-llama-3.1-405b:free", "meta-llama/llama-3.3-70b-instruct:free"] },
};

const categorySelect = document.getElementById("category-select");
const modelSelect = document.getElementById("model-select");
let currentModel = "";

function prettyModel(id) { return id.replace(":free", "").split("/").pop(); }

function populateCategories() {
  categorySelect.innerHTML = "";
  for (const [key, cat] of Object.entries(CATEGORIES)) {
    const o = document.createElement("option");
    o.value = key; o.textContent = cat.label;
    categorySelect.appendChild(o);
  }
}
function populateModels(catKey) {
  modelSelect.innerHTML = "";
  for (const id of CATEGORIES[catKey].models) {
    const o = document.createElement("option");
    o.value = id; o.textContent = prettyModel(id);
    modelSelect.appendChild(o);
  }
  currentModel = modelSelect.value;
}

categorySelect.addEventListener("change", () => populateModels(categorySelect.value));
modelSelect.addEventListener("change", () => { currentModel = modelSelect.value; });

populateCategories();
populateModels(categorySelect.value);
```

- [ ] **Step 2: Verify in browser**

1. Enter the app (save a key).
2. Category dropdown shows: Coding, Chatbot général, Raisonnement, Créatif / rédaction.
3. Selecting "Coding" → model dropdown shows `qwen3-coder`, `gpt-oss-120b`.
4. Selecting "Chatbot général" → model dropdown updates to its three models.
5. In DevTools console, type `currentModel` → it matches the selected model id (e.g. `"meta-llama/llama-3.3-70b-instruct:free"`).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: category and model selectors"
```

---

### Task 4: Light Markdown renderer (pure function, escaped)

**Files:**
- Modify: `index.html` (inside `<script>`)

**Interfaces:**
- Produces: global `renderMarkdownLight(text) -> htmlString`. Escapes HTML first, then converts fenced code blocks, inline code, bold, italic, and unordered lists. Other tasks call it to render assistant messages.

- [ ] **Step 1: Add the renderer**

```js
function escapeHtml(s) {
  return s.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
}

function renderMarkdownLight(text) {
  let html = escapeHtml(text);
  // fenced code blocks ```...```
  html = html.replace(/```([\s\S]*?)```/g, (_, code) => `<pre><code>${code.replace(/^\n/, "")}</code></pre>`);
  // inline code `...`
  html = html.replace(/`([^`\n]+)`/g, "<code>$1</code>");
  // bold **...**
  html = html.replace(/\*\*([^*]+)\*\*/g, "<strong>$1</strong>");
  // italic *...*
  html = html.replace(/(^|[^*])\*([^*\n]+)\*/g, "$1<em>$2</em>");
  // unordered list lines starting with "- "
  html = html.replace(/(?:^|\n)((?:- .*(?:\n|$))+)/g, (m) => {
    const items = m.trim().split("\n").map(l => `<li>${l.replace(/^- /, "")}</li>`).join("");
    return `<ul>${items}</ul>`;
  });
  // remaining newlines → <br>, but not inside <pre>
  html = html.replace(/<pre>[\s\S]*?<\/pre>|\n/g, (match) => match.startsWith("<pre>") ? match : "<br>");
  return html;
}
```

- [ ] **Step 2: Verify the pure function in the browser console**

Open `index.html`, then in DevTools console run:

```js
renderMarkdownLight("**gras** et `code`")            // → "<strong>gras</strong> et <code>code</code>"
renderMarkdownLight("```\nx=1\n```")                 // → "<pre><code>x=1\n</code></pre>"
renderMarkdownLight("- a\n- b")                      // → "<ul><li>a</li><li>b</li></ul>"
renderMarkdownLight("<script>alert(1)</script>")     // → escaped, no live tag
```

Expected: each output matches (HTML is escaped; no executable tag is produced for the last one).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: light markdown renderer with HTML escaping"
```

---

### Task 5: Send message + streaming + render

**Files:**
- Modify: `index.html` (inside `<script>`)

**Interfaces:**
- Consumes: `apiKey`, `currentModel`, `renderMarkdownLight`, `#messages`, `#prompt`, `#composer`.
- Produces: globals `messages` (array of `{role, content}`), `addMessage(role, content)`, `sendMessage()`. Calls `handleError(...)` (defined in Task 6) — define a temporary stub now and replace it in Task 6.

- [ ] **Step 1: Add a temporary error stub (replaced in Task 6)**

```js
function handleError(status, detail) {
  addMessage("assistant", "⚠️ Erreur (" + status + "). " + (detail || ""), true);
}
```

- [ ] **Step 2: Add system prompt, history, rendering, and streaming send**

```js
const SYSTEM_PROMPT = "Tu réponds en français. Tu es clair, direct, concret et utile. Quand tu n'es pas sûr, tu le dis.";
const messagesEl = document.getElementById("messages");
const promptEl = document.getElementById("prompt");
let messages = [];

function addMessage(role, content, isError) {
  const div = document.createElement("div");
  div.className = "msg " + role + (isError ? " error" : "");
  div.innerHTML = role === "assistant" && !isError ? renderMarkdownLight(content) : escapeHtml(content);
  messagesEl.appendChild(div);
  messagesEl.scrollTop = messagesEl.scrollHeight;
  return div;
}

async function sendMessage() {
  const text = promptEl.value.trim();
  if (!text) return;
  promptEl.value = "";
  messages.push({ role: "user", content: text });
  addMessage("user", text);

  const assistantDiv = addMessage("assistant", "…");
  let acc = "";
  try {
    const res = await fetch("https://openrouter.ai/api/v1/chat/completions", {
      method: "POST",
      headers: { "Authorization": "Bearer " + apiKey, "Content-Type": "application/json" },
      body: JSON.stringify({
        model: currentModel,
        messages: [{ role: "system", content: SYSTEM_PROMPT }, ...messages],
        stream: true,
      }),
    });
    if (!res.ok) {
      let detail = "";
      try { detail = (await res.json())?.error?.message || ""; } catch {}
      assistantDiv.remove();
      handleError(res.status, detail);
      return;
    }
    const reader = res.body.getReader();
    const decoder = new TextDecoder();
    let buffer = "";
    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      buffer += decoder.decode(value, { stream: true });
      const lines = buffer.split("\n");
      buffer = lines.pop();
      for (const line of lines) {
        const t = line.trim();
        if (!t.startsWith("data:")) continue;
        const data = t.slice(5).trim();
        if (data === "[DONE]") continue;
        try {
          const delta = JSON.parse(data).choices?.[0]?.delta?.content;
          if (delta) { acc += delta; assistantDiv.innerHTML = renderMarkdownLight(acc); messagesEl.scrollTop = messagesEl.scrollHeight; }
        } catch {}
      }
    }
    messages.push({ role: "assistant", content: acc });
  } catch (e) {
    assistantDiv.remove();
    handleError("network", e.message);
  }
}

document.getElementById("composer").addEventListener("submit", (e) => { e.preventDefault(); sendMessage(); });
promptEl.addEventListener("keydown", (e) => {
  if (e.key === "Enter" && !e.shiftKey) { e.preventDefault(); sendMessage(); }
});
```

- [ ] **Step 3: Verify in browser with a real key**

1. Save a valid OpenRouter key.
2. Pick "Chatbot général" → `gpt-oss-20b` (a model confirmed responsive).
3. Send "Dis bonjour en une phrase." → response streams in token by token, formatted.
4. Send a follow-up referencing the previous answer → confirm context is kept (history sent).
5. Send "Donne un exemple de code Python" → fenced code renders in a `<pre>` block.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: streaming chat send and render"
```

---

### Task 6: Robust error handling (401 / 429 / network)

**Files:**
- Modify: `index.html` (replace the Task 5 `handleError` stub)

**Interfaces:**
- Consumes: `addMessage`, `showKeyPanel`.
- Produces: final `handleError(status, detail)` mapping known cases to clear French messages.

- [ ] **Step 1: Replace the stub `handleError` with the full version**

```js
function handleError(status, detail) {
  if (status === 401) {
    addMessage("assistant", "⚠️ Clé invalide. Vérifie ta clé OpenRouter — je te ramène à la saisie.", true);
    showKeyPanel();
    return;
  }
  if (status === 429) {
    addMessage("assistant", "⚠️ Ce modèle gratuit est saturé pour l'instant (limite atteinte). Réessaie dans un moment ou choisis un autre modèle.", true);
    return;
  }
  if (status === "network") {
    addMessage("assistant", "⚠️ Erreur réseau : " + (detail || "connexion impossible") + ". Réessaie.", true);
    return;
  }
  addMessage("assistant", "⚠️ Erreur (" + status + "). " + (detail || ""), true);
}
```

- [ ] **Step 2: Verify each error path in browser**

1. **401:** In DevTools console set `apiKey = "sk-or-v1-invalid"` then send a message → red message "Clé invalide…" appears AND the app returns to the key panel.
2. **429:** Re-enter a valid key, select "Chatbot général" → `llama-3.3-70b` (often rate-limited) and send; if you get the 429 message, confirmed. (If it answers instead, trust the 401 + network checks and the code path.)
3. **Network:** In DevTools Network tab set "Offline", send a message → red "Erreur réseau…" message appears.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: clear error handling for 401/429/network"
```

---

### Task 7: New conversation + README

**Files:**
- Modify: `index.html` (wire `#new-conv`)
- Modify: `README.md`

**Interfaces:**
- Consumes: `messages`, `#messages`, `#new-conv`.

- [ ] **Step 1: Wire the "Nouvelle conversation" button**

```js
document.getElementById("new-conv").addEventListener("click", () => {
  messages = [];
  messagesEl.innerHTML = "";
  promptEl.focus();
});
```

- [ ] **Step 2: Write the README**

```markdown
# OpenRouter Chat

Un mini chat IA dans un seul fichier. Aucune installation.

## Utilisation

1. Télécharge `index.html` (ou ouvre la page hébergée).
2. Ouvre-le dans ton navigateur (double-clic).
3. Crée une clé gratuite sur https://openrouter.ai/keys et colle-la.
4. Choisis une catégorie et un modèle, puis discute.

Ta clé reste **dans ton navigateur** (localStorage) — elle n'est jamais envoyée
ailleurs qu'à OpenRouter. Bouton « Effacer ma clé » pour la retirer.
Les conversations ne sont pas sauvegardées (perdues au rechargement).

## Modèles

Une sélection de modèles **gratuits** d'OpenRouter, classés par catégorie
(Coding, Chatbot général, Raisonnement, Créatif). Édite la table `CATEGORIES`
en haut du `<script>` dans `index.html` pour en ajouter/retirer.

> Note : les modèles gratuits sont parfois saturés (erreur 429). Réessaie ou
> change de modèle.
```

- [ ] **Step 3: Verify end-to-end in browser**

1. Have a conversation with a few exchanges.
2. Click "Nouvelle conversation" → message list clears, input focused.
3. Send a new message that references the prior topic → model has no memory of it (context reset confirmed).
4. Re-read README and confirm steps match actual UI behavior.

- [ ] **Step 4: Commit**

```bash
git add index.html README.md
git commit -m "feat: new conversation reset and README"
```

---

## Self-Review

**Spec coverage:**
- Single-file zero-install → Tasks 0–1. ✓
- Key in localStorage + Effacer → Task 2. ✓
- Categories (4) + curated models → Task 3. ✓
- Markdown light + escaping → Task 4. ✓
- Streaming + history (stateless resend) → Task 5. ✓
- Errors 401/429/network → Task 6. ✓
- New conversation, keep-thread-on-model-switch (no reset wired on model change) → Tasks 5/7. ✓
- Repo = index.html + README + .gitignore; Python deleted → Task 0. ✓
- French UI, system theme → Task 1. ✓

**Placeholder scan:** No TBD/TODO; the only stub (`handleError` in Task 5) is explicitly replaced in Task 6. ✓

**Type consistency:** `apiKey`, `currentModel`, `messages`, `addMessage(role, content, isError)`, `renderMarkdownLight`, `escapeHtml`, `handleError(status, detail)`, `showKeyPanel`/`showApp` are defined once and used consistently across tasks. DOM ids match Task 1 exactly. ✓
