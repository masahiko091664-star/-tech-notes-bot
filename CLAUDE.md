# CLAUDE.md

Guidance for AI assistants (Claude Code and others) working in this repository.

## What this project is

**tech-notes-bot** is a single-file, client-side RAG (retrieval-augmented
generation) chatbot. It answers questions *only* from a curated set of Markdown
"notes" hosted on GitHub, using the Anthropic API directly from the browser.

- The entire application is **`index.html`** — HTML, CSS, and vanilla JS in one
  file. There is no build step, no framework, no bundler, and no `package.json`.
- The UI is in **Japanese**. Keep user-facing strings and code comments in
  Japanese to match the existing style unless asked otherwise.
- It is designed to be deployed as a static site (Netlify) — just push and it
  redeploys.

## Repository layout

```
index.html              The whole app (UI + logic). Edit this for behavior/UI.
notes/
  index.json            Manifest the deployed app reads (path "notes").
  notes/
    index.json          Second manifest (nested copy).
    claude-code.md      A knowledge note.
    netlify.md          A knowledge note.
```

### ⚠ Note the nested `notes/notes/` directory

There is a structural quirk: the Markdown notes currently live in
**`notes/notes/`**, but the app's default configuration fetches from the
**`notes/`** path (see `ghPath` default `"notes"` in `index.html`). So
`notes/index.json` lists `claude-code.md` and `netlify.md`, yet those files
physically sit one level deeper in `notes/notes/`.

When adding or moving notes, be deliberate about which directory the deployed
app actually reads. The reference path is whatever is configured in the app's
settings (default `notes`), and `<path>/index.json` must list files that exist
at `<path>/<filename>`. If you consolidate the layout, update both the file
locations and the `index.json` `files` arrays together, and verify against the
default `ghPath`.

## How the app works (data flow)

1. User opens the page and enters, in the ⚙ settings panel:
   - Anthropic **API key** (stored only in `localStorage`, key `techNotesCfg`).
   - GitHub **user / repo / branch / path** pointing at the notes.
2. `reloadNotes()` fetches `https://raw.githubusercontent.com/<user>/<repo>/<branch>/<path>/index.json`,
   reads its `files` array, and fetches each Markdown file from the same raw base.
3. On each question, `selectContext()` does lightweight local retrieval:
   - `chunkize()` splits each note on Markdown headings (`#`..`######`).
   - `tokenize()` extracts ASCII tokens and Japanese (hiragana/katakana/kanji)
     runs of length ≥ 2.
   - Chunks are scored by token overlap with the question; the top chunks are
     concatenated up to `maxChars` (14000). If nothing matches, the first 6
     chunks are sent as a fallback.
4. The selected context is injected into a Japanese system prompt that
   instructs the model to answer **only** from the provided knowledge and to say
   it found nothing rather than inventing an answer. The request goes to
   `POST https://api.anthropic.com/v1/messages` with
   `anthropic-dangerous-direct-browser-access: true`.
5. The response is rendered with a minimal Markdown renderer (`renderMd()` —
   only fenced code blocks and inline code) and source note filenames are shown.

## Common tasks

### Add or edit a knowledge note
1. Add/edit a `.md` file in the notes directory that the app reads (see the
   nested-directory caveat above).
2. Add its filename to the `files` array in the corresponding `index.json`.
3. Commit and push — the app reads notes live from GitHub raw at runtime, so no
   redeploy of `index.html` is needed for note changes (the browser refetches on
   reload).

### Change the model
The model id is hard-coded in `index.html` in the `fetch` body
(`model: "claude-opus-4-20250514"`). Update it there. Prefer the latest capable
Claude model when changing this; current model ids include
`claude-opus-4-8` (Opus 4.8) and `claude-sonnet-4-6` (Sonnet 4.6).

### Change retrieval behavior
Edit `selectContext()`, `chunkize()`, `tokenize()`, and the `maxChars` budget in
`index.html`.

### Change the system prompt / answer policy
Edit the `system` array built inside `send()` in `index.html`.

## Conventions & constraints

- **No build / no dependencies.** Do not introduce a bundler, `npm`, or a
  framework. Keep everything self-contained in `index.html` unless the user
  explicitly asks to restructure.
- **No tests / no linters** are configured. There is nothing to run; verify by
  opening `index.html` in a browser (or via `netlify dev`, per `netlify.md`).
- **Secrets stay client-side.** The API key lives only in `localStorage` and is
  sent directly to Anthropic from the browser. Do not commit keys, and do not
  add code that ships keys anywhere else. If a server-side proxy is ever added,
  use a Netlify Function (`netlify/functions/*.js`) with the key in an env var
  (`ANTHROPIC_API_KEY`), per `notes/notes/netlify.md`.
- **Match the existing style:** vanilla DOM APIs, the `$ = id => getElementById`
  helper, Japanese comments, the dark GitHub-like theme variables in `:root`.
- Keep `index.html` self-contained and dependency-free so it deploys as a plain
  static file.

## Deployment

Static-site deploy (Netlify). Pushing `index.html` to the deployed branch
redeploys automatically. See `notes/notes/netlify.md` for Functions and env-var
details if backend logic is ever introduced.

## Git workflow

- Commit with clear, descriptive messages.
- Only open a pull request when the user explicitly asks for one.
