# Spelling Bee — Design Spec

A bilingual (English/French) spelling bee game for a 10-year-old. The game speaks a word aloud, shows which language it's in, and the player types the correct spelling. Streak-based: one wrong answer ends the game.

## Game Structure & Flow

### Start Screen
- Title: "Spelling Bee" with a bee/microphone icon
- "Start" button
- Best streak displayed (loaded from localStorage)

### Game Loop
1. Pick a random word from the combined EN+FR list (no repeats within a session)
2. Show a flag indicator: "🇬🇧 English" or "🇫🇷 Français"
3. Auto-speak the word via Web Speech API
4. "Replay" button allows unlimited re-listens
5. Player types answer in input field, submits with Enter or Submit button
6. **Correct:** confetti animation, streak increments, short delay, next word
7. **Wrong:** screen shake, correct spelling revealed for 2 seconds, game over

### Game Over Screen
- Final streak count
- Best streak (updated if beaten)
- "Play Again" button

## Word List

### English (~100 words)
Common spelling bee words for 10-year-olds — words with tricky patterns: "necessary", "separate", "rhythm", "beautiful", "Wednesday", "environment", "accommodation", "definitely", "occurrence", etc.

### French (~100 words)
Similarly tricky French words: "aujourd'hui", "beaucoup", "bibliothèque", "papillon", "tranquille", "appareil", "conscience", "accueillir", "malheureusement", etc.

### Selection Logic
- Each round, shuffle the combined list (~200 words)
- For each word, display the language flag corresponding to its list
- No repeats until the full list is exhausted (unlikely given streak-based game over)

### Accent Handling
- Input supports accented characters
- Comparison is exact: "francais" is wrong if the word is "français"
- This teaches proper accent usage
- **On-screen accent helper:** When a French word is active, show a row of clickable accent buttons (é, è, ê, ë, à, â, ç, î, ï, ô, ù, û) above the input field. Clicking a button inserts that character at the cursor position. Hidden for English words.

## Audio — Web Speech API

- Use `window.speechSynthesis` with `SpeechSynthesisUtterance`
- Set `utterance.lang = 'en-US'` for English, `utterance.lang = 'fr-FR'` for French
- Auto-play the word when it appears
- "Replay" button triggers the same utterance
- Speech rate: 0.9 (slightly slower for clarity)
- **Fallback:** If `speechSynthesis` is unavailable, show a warning on the start screen. No silent failure.

## UI & Visual Design

### Theme
- Dark background with gradients, consistent with other games in the project
- Spelling bee / stage theme with warm accent colors

### Layout (centered, single column)
- **Top bar:** current streak counter, best streak
- **Middle:** language flag indicator, large Play/Replay button (microphone icon)
- **Below:** text input field, submit button
- **Bottom:** subtle hint text ("Type what you hear")

### Animations
- **Correct:** CSS-only confetti burst (particles falling), green flash on input
- **Wrong:** CSS keyframe screen shake, red flash on input, correct spelling fades in as large text

### Responsive
- Works on desktop and tablet
- Input field large enough for easy typing

### Landing Page Card
- Icon: 🐝
- Title: "Spelling Bee"
- Description: "Listen to the word and spell it correctly! Bilingual English & French spelling challenge."
- Tag: "Spelling — EN/FR"
- Age badge: "Age 10"

## Data & Persistence

- **localStorage key:** `spelling-bee-best-streak`
- **Stored value:** single number, all-time best streak
- **Updated:** only when current streak exceeds stored best at game over
- No other persistence. Word list is hardcoded in the HTML file.

## Technical Constraints

- Single self-contained HTML file: `spelling-bee/index.html`
- No dependencies, no build step, no frameworks
- All CSS and JS inline
- Served as a static file on GitHub Pages
