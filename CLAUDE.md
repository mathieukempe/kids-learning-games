# Kids Learning Games

Educational browser games for kids, deployed on GitHub Pages.

## Project Structure

```
kids-learning-games/
├── index.html                      # Landing page linking to all games
├── multiplication-drop/
│   └── index.html                  # Multiplication tables game (ages 10+)
├── .github/workflows/pages.yml     # GitHub Pages deployment workflow
└── docs/superpowers/specs/         # Design specs for each game
```

## Adding a New Game

1. Create a new directory: `<game-name>/index.html`
2. The game must be a **single self-contained HTML file** (HTML + CSS + JS inline)
3. **No dependencies** — no npm, no frameworks, no build step
4. Add a link to the game in the root `index.html` landing page
5. Commit and push to `main` — GitHub Actions deploys automatically

## Constraints

- All games must work as **static files** served by GitHub Pages
- No server-side code — everything runs in the browser
- Use `localStorage` for persistence (high scores, progress)
- Keep each game in its own directory with an `index.html`

## Deployment

- **URL:** https://mathieukempe.github.io/kids-learning-games/
- **Method:** GitHub Actions workflow (`.github/workflows/pages.yml`)
- **Trigger:** Every push to `main` auto-deploys
- **No build step** — files are uploaded as-is

## Testing

Open any `index.html` directly in a browser for local testing. No dev server needed.
