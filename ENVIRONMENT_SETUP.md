# Environment Setup — The Unofficial Guide

This combines the course-wide setup material from both Unit 1 ("Building a RAG
System") and Unit 2 ("Testing a RAG System") into one sequence, and checks
every generic claim in that material against this repo's actual state as of
2026-10-31. Where the course material is correct but incomplete for what this
repo specifically needs, that gap is called out below as a **Reality check**.

This project's own setup steps (corpus choice, chunking numbers, cutoff) are
in [PROJECT_WALKTHROUGH.md](PROJECT_WALKTHROUGH.md), Section 8. This file is
only the general, repo-independent setup.

---

## 1. Supported Python

`requirements.txt` pins to **Python 3.11, 3.12, or 3.13** — explicitly not
3.14, because it ships as the default on recent macOS and several pinned
packages don't support it yet.

**Reality check:** the `.venv` already present in this repo was built with
**Python 3.13.7** (`.venv\Scripts\python.exe --version`, run 2026-10-31), which
is inside the supported range.

---

## 2. Create the Virtual Environment, Install Dependencies, Add Your Key

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
cp .env.example .env
```

### Windows (PowerShell)

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
copy .env.example .env
```

Then open `.env` and paste your key into `GEMINI_API_KEY=`. `.env.example`
names the exact variable and points to where to get a free key
(https://aistudio.google.com — create your own project; the free quota is
counted per project).

**Activate the virtual environment every time you open a new terminal.** You
should see `(.venv)` at the start of your prompt. Installing into system
Python instead of the venv is the most common cause of "it worked yesterday."

**Reality check — a `.venv` already exists in this repo.** The steps above
describe setup from zero. This working copy already has a `.venv/` (created
earlier, Python 3.13.7) with the pinned packages from `requirements.txt`
installed — confirmed by running `python test.py` (Section 4 below), which
reported all 7 pinned packages importing cleanly. Re-running `python -m venv
.venv` on top of it is unnecessary; just activate the existing one unless you
specifically want a clean rebuild.

**Reality check — `AI201_CORPUS` in `.env`.** This repo's `.env` has
`# AI201_CORPUS=campus_life` present but **commented out**, so it has no
effect. The corpus actually used is whatever `config.py` defaults to —
`campus_life` (`config.py:23`) — confirmed by running `python app.py corpora`,
which marks `campus_life` with `*` (current default) in real output:

```
  advice_threads
    Question-and-answer threads with several people replying and disagreeing. Uneven lengths; answers spread across replies. (23 documents)

* campus_life
    Short posts about student life. ~88 documents of 1–3 paragraphs. Useful information usually sits in a single sentence. (88 documents)

  city_guides
    Long travel guides divided into labelled sections — nine towns and five guides that cut across them. Information is organised by heading and spread across paragraphs. (14 documents)

  practice
    Not for your project — the small corpus used for the in-class follow-along. (28 documents)

* = current default, set in config.py (AI201_CORPUS in .env wins)
```

---

## 3. What `pip install -r requirements.txt` Actually Installs

`requirements.txt` (31 lines, read in full for this audit) pins exactly these
packages, each with both a lower and upper bound:

| Package | Why |
|---|---|
| `python-dotenv` | loads `GEMINI_API_KEY` out of `.env` |
| `chromadb` | the vector store; also *makes* the embeddings (bundles an ONNX build of `all-MiniLM-L6-v2`, no separate embedding package needed) |
| `posthog` | pinned only because `chromadb` depends on it unpinned, and a current `posthog` prints a harmless-looking fake error on every Chroma call |
| `google-genai` | calls the hosted model that writes answers |
| `rank-bm25` | shipped for Unit 2's optional hybrid-search improvement; unused by default |
| `flask` | Unit 9's HTTP service (`serve.py`); unused in Units 1–2 |
| `gunicorn` | Unit 9's production server; unused in Units 1–2 |

**Reality check — two packages this repo's own code imports are *not* in this
list.** As of 2026-10-31, the currently committed `chunker.py` (the body
actively wired into `split_documents`, see
[PROJECT_WALKTHROUGH.md §5](PROJECT_WALKTHROUGH.md#5-findings-and-lessons-learned),
Finding 2) imports `langchain_text_splitters`, and the committed `scorer.py`
imports `rapidfuzz` at module level. Neither appears in `requirements.txt`.
Verified directly against this repo's own `.venv`:

```
> .venv\Scripts\python.exe -m pip show langchain-text-splitters rapidfuzz
WARNING: Package(s) not found: langchain-text-splitters, rapidfuzz
```

A plain `pip install -r requirements.txt`, by itself, does **not** leave this
environment able to run `python app.py chunks`, `python app.py index`, or
`import scorer`. See Section 5 below and
[PROJECT_WALKTHROUGH.md §5](PROJECT_WALKTHROUGH.md#5-findings-and-lessons-learned)
for what each one does when you try.

If you want to reproduce the state this repo's Milestone 3 and Unit 2 work
actually depends on, install both explicitly (this is what was done to verify
this document):

```powershell
pip install langchain-text-splitters rapidfuzz
```

Neither pin is tested against an upper bound the way the rest of
`requirements.txt` is — installing "whatever's current" is what was done here
(`langchain-text-splitters` resolved to `1.1.3`, `rapidfuzz` to `3.14.6` on
2026-10-31).

---

## 4. Confirm the Environment Works

```bash
python test.py
```

The first run is slow — it downloads the embedding model, about 80 MB, from
Chroma's own CDN (`chroma-onnx-models.s3.amazonaws.com`), not from Hugging
Face. If your network blocks one and not the other, this is the one that has
to get through.

**Reality check — real, just-captured output** (2026-10-31, this repo's own
`.venv`), replacing a generic "it should say PASS" description:

```
AI201 environment check
------------------------------------------------------------
[PASS] Python version
         3.13.7 on Windows
[PASS] Virtual environment
         C:\Users\Angelica M\Documents\CP Prework\ai201-project1-unofficial-guide-starter-v2026\.venv
[PASS] Pinned packages
         all 7 import cleanly
[PASS] Free disk space
         286.4 GB
[PASS] Memory
         15.3 GB
[PASS] Key hygiene
         .env is ignored by git
[PASS] API key
         loaded, 53 characters
[ .. ] Embedding model — first run downloads ~80 MB, this is the slow part
[PASS] Embedding model
         all-MiniLM-L6-v2 loaded, 384-dim vectors
[PASS] Vector store
         Chroma round trip on a cosine collection
[PASS] Model call
         gemini-3.5-flash-lite replied "Ready."
------------------------------------------------------------
10 passed, 0 failed, 0 to look at, 0 skipped

You're set. See you in class.
```

**This 10/10 pass does not mean the whole app runs end to end.** `test.py`
only checks the 7 packages pinned in `requirements.txt`, plus the key, disk,
memory, embedding model, vector store, and one live model call — it does not
import `chunker.py` or `scorer.py`, so it cannot catch the Section 3 gap
above. See
[PROJECT_WALKTHROUGH.md §5, Finding 2](PROJECT_WALKTHROUGH.md#5-findings-and-lessons-learned).

If `test.py` fails on something it passed before, re-run
`pip install -r requirements.txt` inside the activated environment first —
this is Unit 2's own setup instruction, and it's still the right first move.

---

## 5. Build the Index (Unit 2's "Confirm Your Index Is Still There" Step)

Unit 2's setup material says: run `python app.py ask "..."` with one of your
test questions, and if it complains there's no index, run
`python app.py index`.

**Reality check — as of 2026-10-31, `python app.py index` cannot currently
complete for this repo's default corpus (`campus_life`), even after the
Section 3 packages are installed.** Real, captured output:

```
> .venv\Scripts\python.exe app.py index
Corpus: campus_life
  loaded   88 documents, 27,908 characters, ~317 characters per document
ModuleNotFoundError: No module named 'langchain_text_splitters'
```

With `langchain-text-splitters` installed (Section 3), the same command gets
further and fails differently:

```
AttributeError: 'Document' object has no attribute 'page_content'
```

This is not a missing-package problem at that point — it's a real
incompatibility between this project's own `ingest.Document` and the
LangChain splitter `chunker.py::split_documents` currently calls. Full
diagnosis, with the exact traceback, is
[PROJECT_WALKTHROUGH.md §5, Finding 2](PROJECT_WALKTHROUGH.md#5-findings-and-lessons-learned).
This file documents the setup gap; fixing `chunker.py` itself is out of scope
here per the walkthrough's own ground rules.

**Local Chroma artifact note.** This working copy has a pre-existing
`chroma_db/` directory on disk (gitignored — `.gitignore` line
`chroma_db/`, never part of the repo's committed history) left over from
earlier runs. As of 2026-10-31 it holds six collections, none named
`campus_life__default`, and `chromadb.PersistentClient(...).list_collections()`
raises `KeyError: '_type'` while parsing one of their stored configs —
consistent with that data having been written by a different `chromadb`
version than the one currently pinned (`0.6.3`, confirmed installed and
matching `requirements.txt`'s `>=0.6,<0.7`). This is local machine state, not
a defect in the committed code: `store.py::build_index` deletes and recreates
its own named collection on every `index` run (`store.py:153-156`), so once
`chunker.py` can produce chunks again, running `python app.py index` rebuilds
a working collection regardless of what stale ones are sitting next to it. If
`list_collections()`-style calls (used by tooling, not by the app itself) keep
tripping on the old data, the fix is to delete the `chroma_db/` folder and
let `index` recreate it.

---

## 6. Rate Limits, Quota, and Your Key

- **Pacing**: `generate.py` paces requests to stay under the free tier's
  per-minute limit and prints a message when it's waiting — a pause is
  expected, not a bug.
- **Daily quota**: generous; you'd only hit it with a runaway loop. A
  `QuotaGuard` error means stop the program and look for the loop, not that
  your key is broken.
- **Your key is a secret.** It lives in `.env`, which `.gitignore` already
  excludes (confirmed: `.gitignore` line 2, `.env`). Never paste it into code,
  README, or a message. If exposed, say so and replace it — keys are free to
  regenerate.
- Only the generation stage (`generate.py`, Stage 5) calls out to a service.
  Embedding and vector search run entirely on your machine and cost no quota.

---

## 7. Quick Troubleshooting Reference

| Symptom | Meaning |
|---|---|
| `No index called '...'. Run python app.py index first.` | Not indexed yet, or you switched corpus without re-indexing |
| `No GEMINI_API_KEY found` | No `.env`, or the key wasn't pasted in (check it didn't save as `.env.txt` on Windows) |
| `ModuleNotFoundError: No module named 'langchain_text_splitters'` or `'rapidfuzz'` | See Section 3 — install both explicitly; they are not in `requirements.txt` |
| `AttributeError: 'Document' object has no attribute 'page_content'` | The chunker/LangChain incompatibility in Section 5 — not fixed by installing anything |
| `[rate limit] ... Waiting Ns` | Working as intended |

This file intentionally does not repeat `RUNNING.md`, which is the
instructor-provided command reference for this repo and is left as-is per the
course's own instruction not to edit it.
