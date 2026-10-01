# Marvel Movies

A static "premium streaming" landing page — full-viewport hero, a frosted-glass info card, and a horizontally scrolling strip of movie posters pulled from the TMDb API.

No build step, no dependencies, no framework. Open it in a browser and it runs.

---

## Quick start

Because the page fetches from an API, it needs to be served over HTTP (not opened as a `file://` path). Any static server works:

```bash
# from the repo root
python3 -m http.server 8080 --directory marvel-test
```

Then open <http://localhost:8080>.

---

## Configuration (TMDb API key)

The page loads **live trending movies from TMDb**, which requires an API key. The key is **not stored in this repository** — it is read at runtime from a local `config.js` file that git ignores.

**To enable live data:**

1. Get a free API key at <https://www.themoviedb.org/settings/api>
2. Copy the template:
   ```bash
   cp marvel-test/config.example.js marvel-test/config.js
   ```
3. Open `marvel-test/config.js` and replace `YOUR_TMDB_API_KEY_HERE` with your key
4. Reload the page

**If you skip this,** the page still works — it detects the missing key, skips the network request, and renders the built-in sample data instead.

### Why it's set up this way

`config.js` is listed in `.gitignore`, so your key can never be committed or pushed. Only `config.example.js` — which contains a fake placeholder — is tracked by git.

**One caveat worth understanding:** any key used in browser-side JavaScript is visible to anyone who opens DevTools, no matter how it's stored. Keeping it out of git stops it leaking *through the repository*; it does not make it secret. For a public deployment you should also **restrict the key to your domain** in your TMDb account settings, so it's useless if copied elsewhere.

For a genuinely secret key you'd need a small server-side proxy to hold it — out of scope for a static site.

---

## Project structure

```
.
├── index.html                  ← placeholder at repo root (see note below)
└── marvel-test/                ← the actual site
    ├── index.html              main page: hero, info card, poster strip
    ├── movie-hero-card.html    standalone portrait card — design reference, not linked
    ├── style.css               all styles for index.html
    ├── config.example.js       config template (committed)
    └── config.js               your local key (gitignored, you create it)
```

**Note on the root `index.html`:** it is currently an empty 0-byte file, and the real site lives one level down in `marvel-test/`. Most static hosts serve the repo root, so deploying as-is produces a blank page. Either move the site files up to the root, or replace the root file with a redirect — see the open issue.

---

## Files at a glance

| File | Purpose |
|---|---|
| `marvel-test/index.html` | The application. Fetches `trending/movie/day` from TMDb, builds up to 20 poster cards, and swaps the hero backdrop + info card when a poster is clicked. |
| `marvel-test/style.css` | Glassmorphism info card, hero layering, scroll strip, responsive breakpoints. |
| `marvel-test/movie-hero-card.html` | A self-contained 320×460 portrait card (its own inline CSS, no JS). Looks like the ancestor of the current poster-only design and is kept as a reference. |
| `marvel-test/config.example.js` | Copy this to `config.js` to add your API key. |

---

## Known issues

Tracked in the project review; the notable ones:

- `IMAGE_BASE_URL` in `index.html` is missing the `/t/` segment (`image.tmdb.org/p/w500` instead of `image.tmdb.org/t/p/w500`), so poster URLs 404 and fall through to a fallback image service that is no longer reliable. **Posters do not currently load for this reason.**
- Card markup is built via `innerHTML` with unescaped TMDb data — an overview containing a `"` can break the markup.
- The horizontal poster cards are not keyboard-focusable (`<article>` + click listener, no `tabindex`/key handling).
- The PLAY and Watchlist buttons and the nav links are inert.
- `movie-hero-card.html` is not linked from anywhere.
