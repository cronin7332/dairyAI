# RAGDAIRY — Vocalkisan Dairy AI Document Assistant

A toy RAG (Retrieval-Augmented Generation) demo built to show the dairy sector
(NDDB, private dairies, etc.) how their existing website — **viksitdairy2047.in**
— can offer an AI assistant grounded in their own project documents (PDFs,
Excel models), via either:

- **Option A — Open-source / self-hosted:** [Ollama](https://ollama.com) running
  `llama3.2:8b` locally or on your own server. Full data control, no per-query
  cost, but needs your own compute and cannot run on free public cloud hosts.
- **Option B — Claude (hosted):** Anthropic's `claude-haiku-4-5` via API.
  Deployable for free on Streamlit Community Cloud, better answer quality,
  small per-query cost.

Both options share the same retrieval pipeline (chunking → embeddings →
ChromaDB vector search) — only the final "LLM answer generation" step differs.
Switch between them from the sidebar radio button in the app.

---

## 1. Local setup

```bash
git clone https://github.com/FGCONSULT/RAGDAIRY.git
cd RAGDAIRY

# reuse or create a conda/venv environment
conda create -n ragdairy python=3.11 -y
conda activate ragdairy

pip install -r requirements.txt
```

Add a couple of sample documents to `data/` (see `data/README.md`).

### To test with Claude (Option B) locally
```bash
cp .streamlit/secrets.toml.example .streamlit/secrets.toml
# edit .streamlit/secrets.toml and paste your real ANTHROPIC_API_KEY
streamlit run app.py
```

### To test with Ollama (Option A) locally
```bash
ollama pull llama3.2:8b
ollama pull nomic-embed-text   # not required by this app (we use a local
                                # HuggingFace embedding model instead), but
                                # useful if you extend the app later
streamlit run app.py
# In the sidebar, choose "Open-source (Ollama, self-hosted only)"
```

Open the local URL Streamlit prints (usually `http://localhost:8501`), upload
a PDF/Excel file (or tick "use preloaded sample documents"), click
**Build / Rebuild Index**, then ask a question in the chat box.

---

## 2. Deploy the free, publicly-hosted version (Option B — Claude)

Streamlit Community Cloud cannot run Ollama, so the public demo link should
use the Claude backend.

1. Push this repo to GitHub (already at `FGCONSULT/RAGDAIRY`) — see §4 below
   if you're pushing local changes.
2. Go to **[share.streamlit.io](https://share.streamlit.io)** → sign in with
   GitHub → **New app**.
3. Repository: `FGCONSULT/RAGDAIRY`, branch: `main`, main file: `app.py`.
4. Click **Advanced settings → Secrets** and paste:
   ```toml
   ANTHROPIC_API_KEY = "sk-ant-xxxxxxxxxxxxxxxxxxxxxxxx"
   ```
5. Deploy. You'll get a public URL such as:
   `https://ragdairy.streamlit.app`

### Deploying Option A (Ollama) for a persistent demo
Streamlit Cloud won't work for this — instead run it on an in-house server or
VPS that has Ollama installed, then either:
- expose it via a subdomain (e.g. `rag.vocalkisan.in`) behind Nginx, or
- for a one-off live demo from your laptop, run `ngrok http 8501` to get a
  temporary public URL.

---

## 3. Add it to viksitdairy2047.in (Wix)

**Method 1 — Menu link (same pattern as your existing "Dairy Forecast App" link):**
Wix Editor → Menu → Add Item → paste your Streamlit/subdomain URL → save.
Zero coding, matches what's already live on the site.

**Method 2 — Inline embed (stays on your domain, feels native):**
1. Wix Editor → **Add Elements → Embed → Embed a Widget** (may show as
   "Custom Element" / "HTML iframe" depending on your Wix plan).
2. Paste:
   ```html
   <iframe src="https://ragdairy.streamlit.app/?embed=true"
           style="width:100%; height:800px; border:none;">
   </iframe>
   ```
3. Place it on a new page, e.g. "Dairy AI Assistant," alongside your
   existing Portfolio/Blog pages.

---

## 4. Pushing local changes to `FGCONSULT/RAGDAIRY`

```bash
git init                      # only if this folder isn't already a git repo
git remote add origin https://github.com/FGCONSULT/RAGDAIRY.git   # if not already set
git add .
git commit -m "Add toy RAG demo app (Claude + Ollama backends)"
git branch -M main
git push -u origin main
```

If the repo already has content (e.g. an initial README from GitHub), pull
first to avoid conflicts:
```bash
git pull origin main --allow-unrelated-histories
# resolve any conflicts, then
git push origin main
```

---

## 5. What this demo is meant to show

| | Option A (Open-source) | Option B (Claude) |
|---|---|---|
| Where the model runs | Your own server/laptop | Anthropic's cloud (API) |
| Data leaves your infrastructure? | No | Only the final prompt + retrieved chunks, per query |
| Cost | Free (your compute) | Small per-query cost (Haiku-class pricing) |
| Setup effort | Higher (GPU/server, Ollama) | Lower (just an API key) |
| Best pitch to | Orgs wanting full data sovereignty (e.g. NDDB internal) | Orgs wanting a fast, low-maintenance pilot |

Both point to the same conclusion for the audience: **the dairy sector can
stand up its own in-house RAG assistant** — either fully self-hosted, or as a
lightweight hosted pilot — using only their own project documents.

---

## Notes / caveats for this toy demo

- Documents uploaded via the app are processed **in-memory for that browser
  session only** — nothing is permanently stored, and the vector index is not
  shared between visitors.
- The preloaded `data/` folder should contain only non-sensitive sample
  files if this repo stays public.
- The guardrail system prompt instructs the model to say *"I don't have this
  information in the uploaded documents"* rather than guess — worth
  demonstrating live by asking an out-of-scope question.
