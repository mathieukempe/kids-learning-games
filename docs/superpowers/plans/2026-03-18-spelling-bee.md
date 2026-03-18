# Spelling Bee Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a bilingual (EN/FR) spelling bee game where the browser speaks a word and the player types the correct spelling, streak-based scoring.

**Architecture:** Single self-contained HTML file with inline CSS and JS. Three screens (start, game, game over) managed by toggling `display` on container divs. Web Speech API for audio. localStorage for best streak persistence.

**Tech Stack:** Vanilla HTML/CSS/JS, Web Speech API, localStorage. No dependencies.

**Spec:** `docs/superpowers/specs/2026-03-18-spelling-bee-design.md`

---

## File Structure

- **Create:** `spelling-bee/index.html` — the entire game (HTML + CSS + JS inline)
- **Modify:** `index.html` — add game card to landing page

---

### Task 1: Scaffold the HTML file with three screens

**Files:**
- Create: `spelling-bee/index.html`

- [ ] **Step 1: Create the HTML skeleton with start, game, and game-over screens**

Create `spelling-bee/index.html` with:
- DOCTYPE, meta charset, viewport, title "Spelling Bee"
- Empty `<style>` block
- Three container divs: `#start-screen`, `#game-screen` (hidden), `#game-over-screen` (hidden)
- Start screen: title "Spelling Bee" with bee emoji, best streak display (`#best-streak-display`), Start button
- Game screen: top bar with streak counters, language flag area, replay button, accent helper row (hidden by default), input field, submit button, hint text
- Game over screen: final streak, best streak, play again button
- Empty `<script>` block

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Spelling Bee</title>
<style>/* Task 2 */</style>
</head>
<body>
  <div id="start-screen">
    <div class="icon">&#x1F41D;</div>
    <h1>Spelling Bee</h1>
    <p class="subtitle">Listen. Spell. Repeat!</p>
    <div id="best-streak-start">Best streak: <span id="best-val-start">0</span></div>
    <div id="speech-warning" style="display:none">Warning: Speech not supported in this browser</div>
    <button id="start-btn">Start</button>
  </div>

  <div id="game-screen" style="display:none">
    <div id="hud">
      <div>Streak: <span id="streak-count">0</span></div>
      <div>Best: <span id="best-val-game">0</span></div>
    </div>
    <div id="lang-flag"></div>
    <button id="replay-btn">Hear Again</button>
    <div id="accent-helper" style="display:none">
      <!-- accent buttons added by JS using safe DOM methods -->
    </div>
    <div id="input-area">
      <input type="text" id="word-input" autocomplete="off" autocapitalize="none" spellcheck="false" placeholder="Type the word...">
      <button id="submit-btn">Submit</button>
    </div>
    <p class="hint">Type what you hear</p>
    <div id="feedback"></div>
  </div>

  <div id="game-over-screen" style="display:none">
    <div class="icon">&#x1F41D;</div>
    <h2>Game Over!</h2>
    <p>Your streak: <span id="final-streak">0</span></p>
    <p>Best streak: <span id="best-val-end">0</span></p>
    <div id="new-record" style="display:none">New Record!</div>
    <button id="play-again-btn">Play Again</button>
  </div>

  <script>/* Tasks 3-7 */</script>
</body>
</html>
```

- [ ] **Step 2: Verify file opens in browser**

Open `spelling-bee/index.html` in a browser. Should see the start screen with the bee icon, title, and start button.

- [ ] **Step 3: Commit**

```bash
git add spelling-bee/index.html
git commit -m "feat(spelling-bee): scaffold HTML with three screens"
```

---

### Task 2: Style the game

**Files:**
- Modify: `spelling-bee/index.html` (the `<style>` block)

- [ ] **Step 1: Add all CSS**

Fill in the `<style>` block with:

- **Base:** dark gradient background (`linear-gradient(135deg, #1a1a2e, #16213e)`), white text, centered layout, `font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif`
- **Start screen:** centered column, large icon (`font-size: 80px`), gradient text on h1, subtitle muted, start button large with warm yellow background (`#ffd32a`), rounded
- **Game screen:** `#hud` flex row with streak/best, `#lang-flag` large text with flag emoji, `#replay-btn` styled as a large circular button, `#accent-helper` as a flex row of small buttons with rounded borders, `#input-area` flex row with large input and submit button, `.hint` muted small text
- **Game over screen:** centered, similar to start
- **Feedback area:** `#feedback` centered, large text for correct/wrong display
- **Confetti animation:** `@keyframes confetti-fall` for particles falling from top. Use `.confetti` class for spawned divs.
- **Shake animation:** `@keyframes shake` — horizontal shake on `body.shake`
- **Input flash:** `.input-correct` green border/shadow, `.input-wrong` red border/shadow
- **Responsive:** input and buttons large enough for tablet use, max-width 600px centered

CSS for animations:

```css
.confetti {
  position: fixed;
  top: -10px;
  width: 10px;
  height: 10px;
  border-radius: 2px;
  animation: confetti-fall linear forwards;
  z-index: 1000;
  pointer-events: none;
}

@keyframes confetti-fall {
  to { top: 100vh; opacity: 0; transform: rotate(720deg); }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-8px); }
  50% { transform: translateX(8px); }
  75% { transform: translateX(-4px); }
}

body.shake { animation: shake 0.4s ease; }

.input-correct { border-color: #7bed9f !important; box-shadow: 0 0 15px rgba(123,237,159,0.5); }
.input-wrong { border-color: #ff6b6b !important; box-shadow: 0 0 15px rgba(255,107,107,0.5); }
```

- [ ] **Step 2: Verify styling in browser**

Open the file — start screen should look polished with dark theme, bee icon, gradient title, yellow start button.

- [ ] **Step 3: Commit**

```bash
git add spelling-bee/index.html
git commit -m "feat(spelling-bee): add CSS styling and animations"
```

---

### Task 3: Add word lists

**Files:**
- Modify: `spelling-bee/index.html` (inside `<script>`)

- [ ] **Step 1: Define the English and French word arrays**

At the top of the `<script>` block, define:

```javascript
const WORDS_EN = [
  "necessary", "separate", "rhythm", "beautiful", "Wednesday",
  "environment", "accommodation", "definitely", "occurrence", "embarrass",
  "conscience", "privilege", "restaurant", "temperature", "immediately",
  "beginning", "calendar", "cemetery", "committee", "experience",
  "government", "guarantee", "independent", "knowledge", "library",
  "lightning", "millennium", "miniature", "miscellaneous", "neighborhood",
  "occasion", "parliament", "perseverance", "possession", "preference",
  "principal", "profession", "questionnaire", "recommend", "reference",
  "refrigerator", "relevant", "sacrifice", "secretary", "sentence",
  "surprise", "thorough", "tomorrow", "unfortunately", "vacuum",
  "vegetable", "weather", "acquaintance", "achievement", "advertise",
  "appreciate", "awkward", "borough", "boundary", "brilliant",
  "broccoli", "business", "category", "challenge", "characteristic",
  "colonel", "communication", "competition", "consequence", "convenience",
  "correspondence", "curiosity", "daughter", "descendant", "dictionary",
  "disappear", "disappointment", "discipline", "enthusiasm", "exaggerate",
  "excellent", "existence", "familiar", "fascinate", "foreign",
  "frequently", "generous", "gorgeous", "gradually", "handkerchief",
  "happened", "height", "imaginary", "intelligence", "interference",
  "interrupt", "island", "jealous", "language", "leisure",
  "maintenance", "marvellous", "medicine", "mischievous", "mysterious"
];

const WORDS_FR = [
  "aujourd'hui", "beaucoup", "biblioth\u00e8que", "papillon", "tranquille",
  "appareil", "conscience", "accueillir", "malheureusement", "n\u00e9cessaire",
  "d\u00e9veloppement", "environnement", "gouvernement", "quelquefois", "maintenant",
  "certainement", "accompagner", "approfondir", "apprentissage", "bouilloire",
  "brouillard", "cueillette", "\u00e9blouissant", "effervescent", "enveloppe",
  "\u00e9quilibre", "exceptionnel", "fr\u00e9quemment", "gargouille", "gentillesse",
  "grammaire", "horizontalement", "hypoth\u00e8se", "imm\u00e9diatement", "inauguration",
  "int\u00e9ressant", "irresponsable", "journaliste", "kilogramme", "laborieusement",
  "l\u00e9opard", "litt\u00e9rature", "magnifique", "malheureux", "math\u00e9matiques",
  "merveilleux", "nonchalamment", "orchestre", "orthographe", "parall\u00e8le",
  "parapluie", "pers\u00e9v\u00e9rance", "philosophie", "photographie", "pourriture",
  "printemps", "psychologie", "quincaillerie", "rassemblement", "r\u00e9frig\u00e9rateur",
  "renseignement", "sauterelle", "silhouette", "soigneusement", "surveillance",
  "sympathique", "temp\u00e9rature", "tournesol", "tourbillon", "transparent",
  "tremblement", "tyrannosaure", "vraisemblable", "abricot", "ambulance",
  "attention", "autoroute", "boulangerie", "camouflage", "catastrophe",
  "chauffeur", "chocolat", "citrouille", "communaut\u00e9", "construction",
  "conversation", "correspondance", "courageux", "crocodile", "dangereux",
  "diff\u00e9rence", "dinosaure", "\u00e9lectricit\u00e9", "enthousiasme", "\u00e9pouvantail",
  "extraordinaire", "fourmi", "grenouille", "h\u00e9risson", "hippopotame"
];
```

- [ ] **Step 2: Verify arrays are valid JS**

Open browser console on the page, type `WORDS_EN.length` and `WORDS_FR.length` — both should return 100.

- [ ] **Step 3: Commit**

```bash
git add spelling-bee/index.html
git commit -m "feat(spelling-bee): add 100 English and 100 French word lists"
```

---

### Task 4: Implement game state and screen management

**Files:**
- Modify: `spelling-bee/index.html` (inside `<script>`, after word lists)

- [ ] **Step 1: Add game state variables and screen-switching functions**

```javascript
// State
let currentWords = [];
let currentIndex = 0;
let streak = 0;
let bestStreak = parseInt(localStorage.getItem('spelling-bee-best-streak')) || 0;

// DOM refs
const startScreen = document.getElementById('start-screen');
const gameScreen = document.getElementById('game-screen');
const gameOverScreen = document.getElementById('game-over-screen');
const streakCount = document.getElementById('streak-count');
const bestValStart = document.getElementById('best-val-start');
const bestValGame = document.getElementById('best-val-game');
const bestValEnd = document.getElementById('best-val-end');
const finalStreak = document.getElementById('final-streak');
const newRecord = document.getElementById('new-record');
const wordInput = document.getElementById('word-input');
const langFlag = document.getElementById('lang-flag');
const accentHelper = document.getElementById('accent-helper');
const feedback = document.getElementById('feedback');
const speechWarning = document.getElementById('speech-warning');

function showScreen(screen) {
  startScreen.style.display = 'none';
  gameScreen.style.display = 'none';
  gameOverScreen.style.display = 'none';
  screen.style.display = 'flex';
}

function updateBestDisplay() {
  bestValStart.textContent = bestStreak;
  bestValGame.textContent = bestStreak;
  bestValEnd.textContent = bestStreak;
}

// Shuffle (Fisher-Yates)
function shuffle(arr) {
  const a = [...arr];
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]];
  }
  return a;
}

// Init
function initGame() {
  const enWords = WORDS_EN.map(w => ({ word: w, lang: 'en' }));
  const frWords = WORDS_FR.map(w => ({ word: w, lang: 'fr' }));
  currentWords = shuffle([...enWords, ...frWords]);
  currentIndex = 0;
  streak = 0;
  streakCount.textContent = '0';
  showScreen(gameScreen);
  nextWord();
}

function endGame() {
  if (streak > bestStreak) {
    bestStreak = streak;
    localStorage.setItem('spelling-bee-best-streak', bestStreak);
    newRecord.style.display = 'block';
  } else {
    newRecord.style.display = 'none';
  }
  finalStreak.textContent = streak;
  updateBestDisplay();
  showScreen(gameOverScreen);
}

// Speech check
if (!window.speechSynthesis) {
  speechWarning.style.display = 'block';
}

updateBestDisplay();
```

- [ ] **Step 2: Verify start screen shows best streak of 0**

Open in browser — start screen should display "Best streak: 0".

- [ ] **Step 3: Commit**

```bash
git add spelling-bee/index.html
git commit -m "feat(spelling-bee): add game state, screen management, shuffle logic"
```

---

### Task 5: Implement speech, accent helper, and core game loop

**Files:**
- Modify: `spelling-bee/index.html` (inside `<script>`, after Task 4 code)

- [ ] **Step 1: Add speech function**

```javascript
function speakWord(word, lang) {
  speechSynthesis.cancel();
  const utterance = new SpeechSynthesisUtterance(word);
  utterance.lang = lang === 'fr' ? 'fr-FR' : 'en-US';
  utterance.rate = 0.9;
  speechSynthesis.speak(utterance);
}
```

- [ ] **Step 2: Add accent helper setup using safe DOM methods**

```javascript
const ACCENT_CHARS = ['\u00e9', '\u00e8', '\u00ea', '\u00eb', '\u00e0', '\u00e2', '\u00e7', '\u00ee', '\u00ef', '\u00f4', '\u00f9', '\u00fb'];

function setupAccentHelper() {
  accentHelper.textContent = '';
  ACCENT_CHARS.forEach(ch => {
    const btn = document.createElement('button');
    btn.textContent = ch;
    btn.className = 'accent-btn';
    btn.addEventListener('click', () => {
      const input = wordInput;
      const start = input.selectionStart;
      const end = input.selectionEnd;
      input.value = input.value.slice(0, start) + ch + input.value.slice(end);
      input.selectionStart = input.selectionEnd = start + 1;
      input.focus();
    });
    accentHelper.appendChild(btn);
  });
}

setupAccentHelper();
```

- [ ] **Step 3: Add nextWord and checkAnswer functions**

```javascript
function nextWord() {
  if (currentIndex >= currentWords.length) {
    currentWords = shuffle(currentWords);
    currentIndex = 0;
  }
  const current = currentWords[currentIndex];
  langFlag.textContent = current.lang === 'fr' ? '\ud83c\uddeb\ud83c\uddf7 Fran\u00e7ais' : '\ud83c\uddec\ud83c\udde7 English';
  accentHelper.style.display = current.lang === 'fr' ? 'flex' : 'none';
  wordInput.value = '';
  wordInput.className = '';
  feedback.textContent = '';
  feedback.className = '';
  wordInput.focus();
  speakWord(current.word, current.lang);
}

function checkAnswer() {
  const current = currentWords[currentIndex];
  const answer = wordInput.value.trim();
  if (!answer) return;

  if (answer === current.word) {
    streak++;
    streakCount.textContent = streak;
    wordInput.className = 'input-correct';
    showConfetti();
    currentIndex++;
    setTimeout(() => nextWord(), 1200);
  } else {
    wordInput.className = 'input-wrong';
    document.body.classList.add('shake');
    feedback.textContent = current.word;
    feedback.className = 'feedback-wrong';
    setTimeout(() => {
      document.body.classList.remove('shake');
      endGame();
    }, 2000);
  }
}
```

- [ ] **Step 4: Add event listeners**

```javascript
document.getElementById('start-btn').addEventListener('click', initGame);
document.getElementById('play-again-btn').addEventListener('click', initGame);
document.getElementById('replay-btn').addEventListener('click', () => {
  const current = currentWords[currentIndex];
  speakWord(current.word, current.lang);
});
document.getElementById('submit-btn').addEventListener('click', checkAnswer);
wordInput.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') checkAnswer();
});
```

- [ ] **Step 5: Verify full game loop works**

Open in browser, click Start, hear a word, type it, submit. Correct should increment streak and play next word. Wrong should shake and show game over.

- [ ] **Step 6: Commit**

```bash
git add spelling-bee/index.html
git commit -m "feat(spelling-bee): implement speech, accent helper, and game loop"
```

---

### Task 6: Add confetti animation

**Files:**
- Modify: `spelling-bee/index.html` (inside `<script>`)

- [ ] **Step 1: Add confetti function**

```javascript
function showConfetti() {
  const colors = ['#ffd32a', '#ff6b6b', '#7bed9f', '#70a1ff', '#ff9ff3'];
  for (let i = 0; i < 30; i++) {
    const confetti = document.createElement('div');
    confetti.className = 'confetti';
    confetti.style.left = Math.random() * 100 + '%';
    confetti.style.backgroundColor = colors[Math.floor(Math.random() * colors.length)];
    confetti.style.animationDuration = (Math.random() * 1 + 0.8) + 's';
    confetti.style.animationDelay = Math.random() * 0.3 + 's';
    document.body.appendChild(confetti);
    confetti.addEventListener('animationend', () => confetti.remove());
  }
}
```

- [ ] **Step 2: Verify confetti CSS is present in style block**

Ensure the `<style>` block (from Task 2) includes the `.confetti` class and `@keyframes confetti-fall`.

- [ ] **Step 3: Test confetti and shake**

In browser: get a word right — confetti should fall. Get a word wrong — screen should shake.

- [ ] **Step 4: Commit**

```bash
git add spelling-bee/index.html
git commit -m "feat(spelling-bee): add confetti and shake animations"
```

---

### Task 7: Add landing page card

**Files:**
- Modify: `index.html` (root landing page)

- [ ] **Step 1: Add the Spelling Bee card to the games grid**

In `index.html`, inside the `.games` div, add after the last game card (Fraction Forge):

```html
<a href="spelling-bee/" class="game-card">
  <div class="icon">&#x1F41D;</div>
  <h2>Spelling Bee</h2>
  <p>Listen to the word and spell it correctly! Bilingual English &amp; French spelling challenge.</p>
  <div class="tags"><span class="tag">Spelling &#x2014; EN/FR</span><span class="age">Age 10</span></div>
</a>
```

- [ ] **Step 2: Verify landing page shows the new card**

Open root `index.html` — should see the bee card alongside the other 10 games.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(spelling-bee): add game card to landing page"
```

---

### Task 8: Final polish and end-to-end test

**Files:**
- Modify: `spelling-bee/index.html` (if needed)

- [ ] **Step 1: End-to-end play test**

1. Open root `index.html`, click Spelling Bee card — should navigate to game
2. Start screen shows, click Start
3. Word is spoken, language flag shown
4. For French word: accent helper buttons visible, click one to insert character
5. Type correct answer — confetti, streak goes to 1
6. Replay button works — word repeats
7. Type wrong answer — shake, correct spelling shown, game over screen
8. Best streak updates if beaten
9. Play Again works — new shuffled words

- [ ] **Step 2: Fix any issues found**

- [ ] **Step 3: Final commit**

```bash
git add spelling-bee/index.html
git commit -m "feat(spelling-bee): final polish"
```
