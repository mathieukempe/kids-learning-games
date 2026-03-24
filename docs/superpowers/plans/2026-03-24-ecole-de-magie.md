# L'École de Magie Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a magic-school-themed French verb conjugation game (être/avoir, présent tense) with three rotating mechanics and constellation progression.

**Architecture:** Single self-contained HTML file (`ecole-de-magie/index.html`) with inline CSS and JS. No dependencies. Game state managed via a central `state` object, persisted to localStorage. Screen transitions via show/hide CSS classes. Visual effects via CSS animations and a canvas-based particle system.

**Tech Stack:** Vanilla HTML/CSS/JS, CSS animations, Canvas API for particles, localStorage for persistence.

**Spec:** `docs/superpowers/specs/2026-03-24-ecole-de-magie-design.md`

---

### Task 1: Scaffold — HTML structure, CSS base, title screen

**Files:**
- Create: `ecole-de-magie/index.html`

Build the foundational HTML with all screen containers, base CSS (palette, fonts, layout), and a working title screen.

- [ ] **Step 1: Create the HTML file with base structure**

Create `ecole-de-magie/index.html` with:
- `<!DOCTYPE html>`, `<html lang="fr">`, viewport meta
- All 5 screen containers as `<div class="screen" id="X-screen">`: `title`, `round-intro`, `round-play`, `round-summary`, `victory`
- Title screen content: game title "L'École de Magie", starry background, "Jouer" button, "Continuer" button (hidden by default), best streak/crystals display
- Back link `← Accueil` linking to `../` (top-left, fixed)
- Base CSS: dark purple/blue gradient background (`#0a0a1a` to `#1a0a2e`), gold accents (`#ffd866`), purple accents (`#a855f7`), font-family Segoe UI, `.screen` display:none with `.screen.active` display:flex

- [ ] **Step 2: Add starry night background**

CSS-only starry background using:
- Multiple `box-shadow` on pseudo-elements for scattered stars (50+ small white dots at random positions)
- A subtle CSS animation for star twinkle (opacity pulse)
- A castle silhouette at the bottom using CSS shapes or an SVG path inline

- [ ] **Step 3: Add title screen logic**

JS at bottom of file:
- `showScreen(id)` function — removes `.active` from all screens, adds to target
- On load: check localStorage for `ecoleDeMagie_currentRound`. If exists, show "Continuer" button. Display best streak and best crystals from localStorage.
- "Jouer" button: reset game state, call `startGame()`
- "Continuer" button: load saved state, call `startGame()` from saved round

- [ ] **Step 4: Verify in browser and commit**

Open `ecole-de-magie/index.html` in browser. Verify: title screen shows, starry background renders, "Jouer" button is visible, back link works.

```bash
git add ecole-de-magie/index.html
git commit -m "feat(ecole-de-magie): scaffold HTML, base CSS, title screen"
```

---

### Task 2: Game state, conjugation data, and round flow

**Files:**
- Modify: `ecole-de-magie/index.html`

Add the data model, game state management, round sequencing, and screen transitions.

- [ ] **Step 1: Add conjugation data**

JS data structures:

```javascript
const VERBS = {
  être: {
    conjugations: [
      { pronoun: 'je', form: 'suis' },
      { pronoun: 'tu', form: 'es' },
      { pronoun: 'il/elle', form: 'est' },
      { pronoun: 'nous', form: 'sommes' },
      { pronoun: 'vous', form: 'êtes' },
      { pronoun: 'ils/elles', form: 'sont' }
    ],
    sentences: [
      { text: 'Je ___ une sorcière', answer: 'suis' },
      { text: 'Tu ___ très courageux', answer: 'es' },
      { text: 'Elle ___ dans le château', answer: 'est' },
      { text: 'Nous ___ des apprentis', answer: 'sommes' },
      { text: 'Vous ___ prêts pour le sortilège', answer: 'êtes' },
      { text: 'Ils ___ au sommet de la tour', answer: 'sont' },
      { text: 'Je ___ prête pour la magie', answer: 'suis' },
      { text: 'Tu ___ le meilleur élève', answer: 'es' },
      { text: 'Il ___ invisible', answer: 'est' },
      { text: 'Nous ___ dans la forêt enchantée', answer: 'sommes' },
      { text: 'Vous ___ les gardiens du grimoire', answer: 'êtes' },
      { text: 'Elles ___ très puissantes', answer: 'sont' }
    ]
  },
  avoir: {
    conjugations: [
      { pronoun: "j'", form: 'ai' },
      { pronoun: 'tu', form: 'as' },
      { pronoun: 'il/elle', form: 'a' },
      { pronoun: 'nous', form: 'avons' },
      { pronoun: 'vous', form: 'avez' },
      { pronoun: 'ils/elles', form: 'ont' }
    ],
    sentences: [
      { text: "J' ___ un balai magique", answer: 'ai' },
      { text: 'Tu ___ une baguette dorée', answer: 'as' },
      { text: 'Elle ___ un chat noir', answer: 'a' },
      { text: 'Nous ___ des potions secrètes', answer: 'avons' },
      { text: 'Vous ___ le grimoire ancien', answer: 'avez' },
      { text: 'Ils ___ des pouvoirs magiques', answer: 'ont' },
      { text: "J' ___ une étoile brillante", answer: 'ai' },
      { text: 'Tu ___ le chapeau pointu', answer: 'as' },
      { text: 'Il ___ la clé du donjon', answer: 'a' },
      { text: 'Nous ___ des cristaux enchantés', answer: 'avons' },
      { text: 'Vous ___ les ingrédients magiques', answer: 'avez' },
      { text: 'Elles ___ des ailes invisibles', answer: 'ont' }
    ]
  }
};

const ROUNDS = [
  { verb: 'être', mechanic: 'spellCasting' },
  { verb: 'avoir', mechanic: 'spellCasting' },
  { verb: 'être', mechanic: 'enchantedSentences' },
  { verb: 'avoir', mechanic: 'enchantedSentences' },
  { verb: 'être', mechanic: 'runeMatching' },
  { verb: 'avoir', mechanic: 'runeMatching' }
];
```

- [ ] **Step 2: Add game state object**

```javascript
const state = {
  currentRound: 0,
  currentQuestion: 0,
  crystals: 0,
  streak: 0,
  bestStreak: parseInt(localStorage.getItem('ecoleDeMagie_bestStreak')) || 0,
  bestCrystals: parseInt(localStorage.getItem('ecoleDeMagie_bestCrystals')) || 0,
  constellations: { être: 0, avoir: 0 }, // stars lit per verb (0-3)
  firstTry: true // reset per question
};
```

- [ ] **Step 3: Add round flow functions**

```javascript
function startGame() { ... }      // reset state (or load), show round intro
function showRoundIntro() { ... } // display verb + mechanic name, auto-transition after 2s
function startRound() { ... }     // dispatch to correct mechanic function
function endQuestion(correct) {}  // update crystals/streak, advance to next question or end round
function showRoundSummary() { ... } // show stars earned, save to localStorage
function showVictory() { ... }    // celebratory screen, save best scores
function saveProgress() { ... }   // persist to localStorage
function loadProgress() { ... }   // restore from localStorage
```

- [ ] **Step 4: Add round intro screen content**

HTML for `#round-intro-screen`: grimoire icon (CSS-styled book shape), verb name display, mechanic name display. CSS entrance animation (fade-in + scale).

- [ ] **Step 5: Add round summary screen content**

HTML for `#round-summary-screen`: crystals earned this round, constellation preview (which stars are lit), "Continuer" button. CSS for crystal counter animation.

- [ ] **Step 6: Verify round flow and commit**

Open in browser. Click "Jouer" → should see round intro for "être / Sortilège" → after 2s should transition (to empty round play for now). Verify localStorage save/load works.

```bash
git add ecole-de-magie/index.html
git commit -m "feat(ecole-de-magie): add game data, state management, round flow"
```

---

### Task 3: Spell Casting mechanic (multiple choice)

**Files:**
- Modify: `ecole-de-magie/index.html`

Implement the first game mechanic: multiple choice with 4 answer orbs.

- [ ] **Step 1: Add Spell Casting HTML**

Inside `#round-play-screen`, add a container `#spell-casting`:
- `.grimoire` div (styled as an open book with CSS)
- `.prompt` text showing "être → nous ___"
- `.orbs` container with 4 `.orb` buttons (dynamically populated)
- `.wand` element (positioned to the side)

- [ ] **Step 2: Add Spell Casting CSS**

- `.grimoire`: centered, book shape with border-radius, inner glow, subtle float animation
- `.orb`: circular buttons with gradient background, glow effect, hover scale
- `.orb.correct`: green glow + scale-up + fade-out animation
- `.orb.wrong`: red flash + shake animation
- `.wand`: positioned right side, tilt animation on cast
- `.wand-beam`: a line/glow that shoots from wand to correct orb (CSS animation using transform + opacity)

- [ ] **Step 3: Add Spell Casting JS logic**

```javascript
function startSpellCasting(verb) {
  // Get shuffled pronouns for this round
  // For each question: show prompt, generate 4 options (correct + 3 distractors from same verb)
  // On orb click: check answer, play correct/wrong animation, call endQuestion()
}
function generateDistractors(verb, correctForm, count) {
  // Pick `count` other forms from the same verb, excluding correctForm
  // Shuffle and return
}
```

Key details:
- Pronouns are shown in shuffled order across the 6 questions
- For avoir, first pronoun displays as "j'" not "je"
- Distractors come from the same verb's other conjugations
- `state.firstTry` tracks if this is the first attempt (for bonus crystal)

- [ ] **Step 4: Verify and commit**

Open in browser. Play through rounds 1-2 (être + avoir spell casting). Verify: prompts show correctly, j' displays for avoir, correct/wrong animations play, crystals increment, round summary shows after 6 questions.

```bash
git add ecole-de-magie/index.html
git commit -m "feat(ecole-de-magie): implement Spell Casting (multiple choice) mechanic"
```

---

### Task 4: Enchanted Sentences mechanic (fill the gap)

**Files:**
- Modify: `ecole-de-magie/index.html`

Implement the second mechanic: sentences with a blank, 3 answer scrolls.

- [ ] **Step 1: Add Enchanted Sentences HTML**

Inside `#round-play-screen`, add container `#enchanted-sentences`:
- `.sentence-display`: shows the sentence with a glowing blank where the answer goes
- `.scrolls` container with 3 `.scroll` buttons

- [ ] **Step 2: Add Enchanted Sentences CSS**

- `.sentence-display`: large text, centered, the blank (`___`) pulses with a gold glow
- `.scroll`: styled as parchment rectangles with aged-paper gradient, slight rotation, hover lift
- `.scroll.correct`: gold glow + float-up animation (sentence rises off screen)
- `.scroll.wrong`: shake + brief red tint

- [ ] **Step 3: Add Enchanted Sentences JS logic**

```javascript
function startEnchantedSentences(verb) {
  // For each pronoun: pick a random sentence for that pronoun from the verb's sentence bank
  // Show sentence with blank, generate 3 options (correct + 2 distractors)
  // On scroll click: check, animate, call endQuestion()
}
```

Key details:
- Each question picks a sentence that matches the current pronoun
- 2 sentences per pronoun are available in the data — pick one randomly
- 3 options (1 correct + 2 distractors from same verb)

- [ ] **Step 4: Verify and commit**

Play through rounds 3-4. Verify: sentences display with blank, scrolls show correct options, animations work, scoring continues from previous rounds.

```bash
git add ecole-de-magie/index.html
git commit -m "feat(ecole-de-magie): implement Enchanted Sentences (fill-the-gap) mechanic"
```

---

### Task 5: Rune Matching mechanic (drag & drop)

**Files:**
- Modify: `ecole-de-magie/index.html`

Implement the third mechanic: match pronouns to conjugations via drag or tap.

- [ ] **Step 1: Add Rune Matching HTML**

Inside `#round-play-screen`, add container `#rune-matching`:
- `.rune-column.pronouns`: 6 pronoun rune elements on the left
- `.rune-column.conjugations`: 6 conjugation rune elements on the right (shuffled)
- `.match-lines` SVG overlay for drawing connection lines between matched pairs

- [ ] **Step 2: Add Rune Matching CSS**

- `.rune`: hexagonal or rounded stone shape, glowing border, text centered
- `.rune.selected`: bright purple glow (for tap-to-select mode)
- `.rune.matched`: green glow, reduced opacity, non-interactive
- `.rune.wrong`: red shake animation
- Match lines: SVG lines with glow filter between matched pairs

- [ ] **Step 3: Add Rune Matching JS — touch and mouse drag**

Implement drag using pointer events (works for both mouse and touch):

```javascript
function startRuneMatching(verb) {
  // Render 6 pronoun runes (left) and 6 shuffled conjugation runes (right)
  // For avoir: first pronoun shows "j'"
  // Track matched pairs
  // When all 6 matched: round complete
}
```

Two interaction modes:
1. **Drag:** `pointerdown` on a pronoun rune starts drag. `pointermove` updates a visual line from rune to cursor. `pointerup` over a conjugation rune checks the match. Uses `touch-action: none` on rune elements and `setPointerCapture`.
2. **Tap-to-select (fallback):** Tap a pronoun → it glows as selected. Tap a conjugation → check match. Tap another pronoun → deselects first, selects new one.

- [ ] **Step 4: Add match validation and animation**

On correct match:
- Both runes get `.matched` class
- SVG line drawn between them with glow
- Increment crystals (each match = 1 question answered)
- `state.firstTry` per pair

On wrong match:
- Both runes shake briefly
- `state.firstTry = false` for the pronoun's pair

When all 6 matched → call `showRoundSummary()`

- [ ] **Step 5: Verify and commit**

Play through rounds 5-6. Test both drag and tap-to-select on desktop. Verify: runes render, drag works, tap-to-select works, matched pairs show lines, wrong matches shake, round completes when all matched.

```bash
git add ecole-de-magie/index.html
git commit -m "feat(ecole-de-magie): implement Rune Matching (drag-and-drop) mechanic"
```

---

### Task 6: Constellation system and victory screen

**Files:**
- Modify: `ecole-de-magie/index.html`

Add the constellation display, victory screen, and complete the progression loop.

- [ ] **Step 1: Add constellation HTML/CSS**

- `.constellation-display`: visible during round summary and victory screens
- Two constellation shapes (SVG): cat shape for être, wand shape for avoir
- Each has 3 star positions — unlit stars are dim circles, lit stars are bright with glow
- When a star lights up: pulse animation + connecting line to previous star draws in

- [ ] **Step 2: Add constellation JS**

```javascript
function updateConstellations() {
  // Read state.constellations, update SVG star visibility and connecting lines
}
function animateStarUnlock(verb, starIndex) {
  // Animate the star lighting up with a delay and sparkle effect
}
```

Constellation progress updates at end of each round in `showRoundSummary()`.

- [ ] **Step 3: Add victory screen**

HTML for `#victory-screen`:
- "Apprentie Sorcière" title with large sparkle animation
- Both constellations fully lit
- Total crystals + best streak display
- "Rejouer" button (restarts rounds, keeps bests)
- "Nouvelle Partie" button (clears current game state, keeps bests)

Victory triggers when `state.currentRound === 6` (all rounds complete).

- [ ] **Step 4: Add HUD (heads-up display)**

Fixed HUD bar at top during gameplay showing:
- Crystal count (with icon 💎)
- Current streak
- Round X/6 indicator

- [ ] **Step 5: Verify and commit**

Play full game (all 6 rounds). Verify: constellations light up after each round, victory screen shows after round 6, "Rejouer" restarts, "Nouvelle Partie" resets, HUD shows correct values.

```bash
git add ecole-de-magie/index.html
git commit -m "feat(ecole-de-magie): add constellation progression, victory screen, HUD"
```

---

### Task 7: Visual effects and particle system

**Files:**
- Modify: `ecole-de-magie/index.html`

Add the magic visual effects: particles, aurora borealis, and polish.

- [ ] **Step 1: Add canvas-based particle system**

A `<canvas>` overlay behind interactive elements, used for:
- Gold/purple star particles on correct answers (burst from the answer element, float up and fade)
- Crystal collect animation (small crystal flies to HUD counter)

```javascript
const canvas = document.getElementById('particles');
const ctx = canvas.getContext('2d');
const particles = [];

function spawnParticles(x, y, color, count) { ... }
function updateParticles() { ... } // requestAnimationFrame loop
```

Keep it lightweight — max 50 particles at a time, simple circle shapes with size/opacity decay.

- [ ] **Step 2: Add aurora borealis effect**

CSS-only aurora effect (triggered on 3+ streak):
- A full-width div at the top of the screen
- Gradient bands (green, purple, blue) with CSS animation: horizontal wave + opacity pulse
- Triggered by adding `.aurora-active` class to body
- Auto-removes after 3 seconds

- [ ] **Step 3: Add wand beam animation**

CSS animation for spell casting correct answers:
- A `.beam` div stretches from wand position to target orb position
- Uses CSS `transform: scaleX()` animation from 0 to 1
- Glow effect via `box-shadow`
- Duration: 400ms, then fades

- [ ] **Step 4: Add grimoire page-flip animation**

Between questions, the grimoire does a page-turn:
- CSS 3D transform: rotateY from 0 to -180deg on a `.page` overlay
- Duration: 500ms
- Reveals new question content underneath

- [ ] **Step 5: Add victory celebration particles**

On victory screen: continuous gentle particle rain (stars falling slowly) + a big initial burst.

- [ ] **Step 6: Verify all effects and commit**

Play full game, verify: particles spawn on correct answers, aurora triggers on streaks, wand beam animates, page flip works between questions, victory celebration plays.

```bash
git add ecole-de-magie/index.html
git commit -m "feat(ecole-de-magie): add particle system, aurora, wand beam, and visual polish"
```

---

### Task 8: Landing page integration and final polish

**Files:**
- Modify: `ecole-de-magie/index.html`
- Modify: `index.html` (root landing page)

- [ ] **Step 1: Add game card to root landing page**

Add to `index.html` after the last game card (before `</div>` closing `.games`):

```html
<a href="ecole-de-magie/" class="game-card">
  <div class="icon">🪄</div>
  <h2>L'École de Magie</h2>
  <p>Apprends à conjuguer être et avoir dans une école de magie !</p>
  <div class="tags"><span class="tag">Français — Conjugaison</span><span class="age">8 ans</span></div>
</a>
```

- [ ] **Step 2: Responsive polish**

Test and fix layout at:
- Desktop (1200px+): grimoire centered, orbs/scrolls/runes well-spaced
- Tablet (768px): full-width layout, touch-friendly tap targets (min 44px)
- Mobile (375px): stacked layout where needed, rune matching columns may need to be narrower

Key responsive adjustments:
- Font sizes scale down on smaller screens
- Rune matching: reduce rune size on mobile, ensure tap targets stay accessible
- Orbs: 2x2 grid on mobile instead of spread around grimoire

- [ ] **Step 3: Final playthrough and commit**

Complete playthrough on desktop. Verify:
- All 6 rounds work with correct mechanics
- Constellations track properly
- Victory screen shows
- localStorage persists between page reloads
- "Continuer" works on title screen after refresh mid-game
- Back link goes to landing page
- Landing page card shows and links correctly

```bash
git add ecole-de-magie/index.html index.html
git commit -m "feat(ecole-de-magie): add to landing page, responsive polish, final adjustments"
```
