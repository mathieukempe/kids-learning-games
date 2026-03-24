# L'École de Magie — French Verb Conjugation Game

**Date:** 2026-03-24
**Target audience:** 8-year-old, learning French verb conjugation
**Scope:** Présent tense of être and avoir
**Directory:** `ecole-de-magie/index.html`

## Overview

A magic-school-themed game where the player is a young apprentice witch learning spells by conjugating être and avoir. The game rotates through three mechanics (multiple choice, drag-and-drop matching, fill-the-gap sentences) to keep engagement high. Correct answers trigger wand spells and light up constellations in a starry night sky.

**How this differs from Conjugation Catapult:** Conjugation Catapult is a typing-based game covering many verbs across multiple tenses, aimed at older kids. L'École de Magie is specifically for younger learners (age 8), focuses exclusively on être/avoir in présent, uses tap/drag mechanics (no typing), and has a stronger progression/reward system to encourage repeated practice.

## Theme & Visual Identity

- **Setting:** Starry night sky with a floating castle silhouette in the background
- **Central element:** A floating grimoire (spell book) — pages glow and flip with each new question
- **Wand:** Floats beside the grimoire, shoots spell beams on correct answers (lightning, stars, swirls)
- **Palette:** Dark purple/blue background, gold and purple glowing accents
- **All UI text in French**

### Visual Effects

- **Correct answer:** Wand beam shoots at target, particle sparkles (gold/purple stars floating up)
- **Correct streak (3+):** Aurora borealis effect ripples across the sky
- **Wrong answer:** Gentle screen shake + brief red-tinted flash on the wand (not punishing)
- **Constellation unlock:** Stars light up with a connecting-lines animation
- **Crystal collect:** Small crystal icon floats up to the score counter

### Character

A young apprentice witch — simple illustrated avatar at the bottom of the screen holding a wand. No customization needed for v1.

## Game Flow

### State Machine

1. **Title Screen** — "L'École de Magie" title, starry background, "Jouer" button. If saved progress exists, also show "Continuer" button.
2. **Round Intro** — Brief animation: grimoire opens, text shows the verb and mechanic for this round (e.g., "Sortilège: être" with a wand flourish). 2 seconds, then auto-transitions.
3. **Round Play** — 6 questions per round (one per pronoun). Mechanic and verb determined by round sequence (see below).
4. **Round Summary** — Stars earned, crystals collected this round, constellation progress animation. "Continuer" button to proceed.
5. **Victory Screen** — After all 6 rounds: title "Apprentie Sorcière" with celebratory animation, total crystals, all constellations lit. "Nouvelle Partie" and "Rejouer" buttons.

### Round Sequence (6 rounds total)

| Round | Verb   | Mechanic            |
|-------|--------|---------------------|
| 1     | être   | Spell Casting (MC)  |
| 2     | avoir  | Spell Casting (MC)  |
| 3     | être   | Enchanted Sentences |
| 4     | avoir  | Enchanted Sentences |
| 5     | être   | Rune Matching       |
| 6     | avoir  | Rune Matching       |

This gives a gentle progression: multiple choice first (easiest), then sentences (contextual), then matching (requires knowing all 6 at once). Être before avoir in each pair since it's typically taught first.

## Game Mechanics

The game plays in **rounds of 6 questions** each (one per pronoun: je, tu, il/elle, nous, vous, ils/elles). The round sequence above determines which mechanic and verb are used.

### 1. Spell Casting (Multiple Choice)

- The grimoire displays a prompt: "être → nous ___" (or "avoir → j' ___" — note the elided "j'" form for avoir)
- 4 glowing answer orbs float around the book
- Player taps the correct orb — wand shoots a beam, orb explodes into stars
- Wrong — orb fizzles with a brief effect, player can try again (no penalty, but no first-try bonus)

### 2. Rune Matching (Drag & Drop)

- 6 pronoun runes on the left, 6 conjugation runes on the right (shuffled)
- Player drags a pronoun onto its matching conjugation (or taps pronoun then taps conjugation — tap-to-select alternative for touch devices)
- Touch support: implemented via `touchstart`/`touchmove`/`touchend` events (not HTML5 drag API, which is unreliable on mobile)
- Correct match — both runes glow and a spell beam connects them
- Wrong match — runes bounce back with a gentle shake
- All 6 matched = round complete
- Note: pronoun runes show "j'" (not "je") when the verb is avoir

### 3. Enchanted Sentences (Fill the Gap)

- A magical sentence floats out of the book: "Tu ___ une baguette magique"
- 3 answer choices appear as floating scrolls
- Player taps the correct scroll — sentence glows gold and floats up into the sky
- Wrong — scroll fizzles, player can try again

## Conjugation Data

### Présent — être

| Pronom | Conjugaison |
|--------|-------------|
| je | suis |
| tu | es |
| il/elle | est |
| nous | sommes |
| vous | êtes |
| ils/elles | sont |

### Présent — avoir

| Pronom | Conjugaison |
|--------|-------------|
| j' | ai |
| tu | as |
| il/elle | a |
| nous | avons |
| vous | avez |
| ils/elles | ont |

### Sentences for Enchanted Sentences Mode

Magic-themed, age-appropriate sentences:

**être:**
- "Je ___ une sorcière" (suis)
- "Tu ___ très courageux" (es)
- "Elle ___ dans le château" (est)
- "Nous ___ des apprentis" (sommes)
- "Vous ___ prêts pour le sortilège" (êtes)
- "Ils ___ au sommet de la tour" (sont)
- "Je ___ prête pour la magie" (suis)
- "Tu ___ le meilleur élève" (es)
- "Il ___ invisible" (est)
- "Nous ___ dans la forêt enchantée" (sommes)
- "Vous ___ les gardiens du grimoire" (êtes)
- "Elles ___ très puissantes" (sont)

**avoir:**
- "J' ___ un balai magique" (ai)
- "Tu ___ une baguette dorée" (as)
- "Elle ___ un chat noir" (a)
- "Nous ___ des potions secrètes" (avons)
- "Vous ___ le grimoire ancien" (avez)
- "Ils ___ des pouvoirs magiques" (ont)
- "J' ___ une étoile brillante" (ai)
- "Tu ___ le chapeau pointu" (as)
- "Il ___ la clé du donjon" (a)
- "Nous ___ des cristaux enchantés" (avons)
- "Vous ___ les ingrédients magiques" (avez)
- "Elles ___ des ailes invisibles" (ont)

### Wrong Answer Generation

For multiple choice and sentences, distractors are pulled from the **same verb's other conjugations**. Example: if the answer is "sommes" (être, nous), distractors might be "sont", "êtes", "suis". This reinforces recognition of real forms rather than introducing nonsense.

## Progression & Rewards

### Scoring

- Each correct answer: +1 crystal
- First-try correct: +1 bonus crystal
- 3+ correct streak: aurora borealis animation

### Constellations (Main Progression)

- **2 constellations:** one for être (e.g., a cat shape), one for avoir (e.g., a wand shape)
- **3 stars per constellation** (one per round of that verb — matching the 3 rounds per verb)
- Each completed round lights up one star in the corresponding constellation
- Stars connect with glowing lines as they light up
- Completing both constellations (all 6 stars) earns the title **"Apprentie Sorcière"** with a celebratory animation (big sparkle burst, floating text)
- After victory, "Rejouer" restarts all rounds; "Nouvelle Partie" resets everything

### Persistence (localStorage)

Key prefix: `ecoleDeMagie_`

- `ecoleDeMagie_bestStreak` — longest correct streak across all games
- `ecoleDeMagie_bestCrystals` — highest crystal total in a single game
- `ecoleDeMagie_constellations` — current game constellation progress (stars lit per verb)
- `ecoleDeMagie_currentRound` — current round index (for "Continuer" on title screen)
- `ecoleDeMagie_crystals` — current game crystal count

"Nouvelle Partie" clears current game state but preserves best streak and best crystals.

### No Punishment Philosophy

Wrong answers don't cost anything — the player just doesn't earn the crystal. The game encourages retry and learning, not perfection anxiety.

## Landing Page Card

```
Icon: 🪄
Title: L'École de Magie
Description: Apprends à conjuguer être et avoir
Tag: Français
Age: 8 ans
```

## Audio

No audio for v1. The game relies on visual feedback only. Audio (chimes, spell sounds) can be added in a future iteration.

## Technical Constraints

- Single self-contained HTML file (HTML + CSS + JS inline)
- No dependencies, no frameworks, no build step
- Works as a static file served by GitHub Pages
- localStorage for persistence
- Responsive — works on tablet and desktop
- Back link to root landing page (← Accueil)
- Must be added to root `index.html` game list
