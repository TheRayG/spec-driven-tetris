# Tetris Game — Implementation Specification

Generate a complete, playable Tetris game in a **single `index.html` file** using vanilla HTML, CSS, and JavaScript. No build tools. External libraries are allowed only via CDN `<link>` or `<script>` tags.

---

## 1. Technical Constraints

- **Single file**: All HTML, CSS, and JavaScript in one `index.html`
- **Font**: 'Press Start 2P' from Google Fonts, fallback to `cursive`
- **No frameworks**: Vanilla JS only (ES6+)
- **Persistence**: `localStorage` for high scores and AI weights; silently fall back to defaults on failure

---

## 2. Visual Design

**Color scheme**: Dark background (`#222`), cyan accents (`#00FFFF`), white text, green highlights (`#00FF00`).

**Layout**: Three-column flexbox (`#game-layout`), 40px gap, centered, flex-start alignment:

| Column | Content | Width |
|--------|---------|-------|
| Left | Controls list + Statistics | 180–220px |
| Center | Game board + Next/Hold previews | Fixed board (250×500px) + sidebar |
| Right | AI Visualization + AI Tuning | 180–240px |

**Right column constraint**: Combined height of AI Visualization and AI Tuning panels must not exceed the game board wrapper height (~514px). Use compact spacing.

**Responsive** (`@media max-width: 799px`): Stack vertically, center-aligned, 20px gap. Game container reorders to top via `order: -1`. Side panels become full width.

**Glow effects**: Cyan box-shadow (`0 0 20px rgba(0,255,255,0.5)`) on the board wrapper.

---

## 3. Game Board & Pieces

### Board
- 10 columns × 20 rows of 25px cells
- 3px solid cyan border, 4px internal padding
- Empty cells: `border: 1px solid rgba(0,0,0,0.1)`
- Filled cells: `border: 1px solid rgba(255,255,255,0.3)`, `box-shadow: inset 0 0 5px rgba(0,0,0,0.5)`

### Seven Tetrominos (I, O, T, S, Z, J, L)
Defined as 2D arrays. Colors in order: `cyan, yellow, purple, green, red, blue, orange`.

### Preview Grids (Next & Hold)
- Each preview wrapped in a container with the shared panel style: `background: rgba(0,0,0,0.5)`, `border: 1px solid rgba(0,255,255,0.3)`, 8px padding. Uppercase cyan label (8px) above the grid.
- 4×4 grid, 20px per cell (80×80px total)
- Subtle cell borders: `0.5px solid rgba(255,255,255,0.05)`
- Filled cells match main board styling
- Pieces placed at natural 0,0 coordinates (no centering logic)

### Ghost Piece
- Shows hard-drop landing position
- Styling: `background: transparent`, `border: 1px dashed white`, `opacity: 0.3`, no box-shadow
- Toggle with `G` key
- Only rendered when ghost Y differs from current Y

---

## 4. Core Game Mechanics

### Piece Spawning
- Spawn position: horizontally centered at Y=0 (top of visible board)
- If the spawned piece collides with existing blocks at Y=0 → game over
- Random bag: simple `Math.random() * 7` for each piece (no 7-bag)

### Gravity & Drop Speed
- Formula: `max(50, 1000 - (level - 1) × 25)` milliseconds
- Continuous interval timer; not paused during lock delay

### Movement
- Left/Right: Move 1 cell if no collision
- Soft Drop: Move down 1 cell, +1 point per cell. If collision → lock piece immediately
- Hard Drop: Instantly drop to lowest position, +2 points per cell, lock immediately

### Rotation (Clockwise Only)
- Matrix transformation: transpose + reverse rows
- Wall kick offsets tried in order: `[0, -1, 1, -2, 2]` (horizontal only, no Y adjustment)
- For each offset: check bounds first, then collision. First success wins.
- Rotation does not affect lock delay timer

### Lock Delay
- Triggered when `autoDown` detects collision and no lock delay timer exists
- Duration = current drop speed
- **Not reset** by movement or rotation; runs to completion
- Only one timer active at a time
- Hard drop and soft drop bypass lock delay (lock immediately)
- Cleared when: piece locks, piece holds, game pauses, game over

### Hold Piece
- Swap current piece with held piece (or next piece if hold is empty)
- Can only hold once per piece drop (`canHold` flag, reset on lock)
- Clears lock delay timer
- Re-spawns the swapped piece at center, Y=0
- Triggers autoplay recalculation if autoplay is enabled

### Line Clearing
- Scan bottom-to-top; when a full row is found, splice it and unshift an empty row
- Re-check the same row index after splice (`y++`)
- Scoring: `[0, 100, 300, 500, 800][linesCleared] × level`

### Level Progression
Lines required to advance per level range:

| Level Range | Lines Per Level |
|-------------|----------------|
| 1–9 | 10 |
| 10–19 | 12 |
| 20–29 | 14 |
| 30–39 | 16 |
| 40–49 | 18 |
| 50+ | 20 |

A `levelLines` counter tracks lines since last level-up. When it reaches the threshold, subtract the threshold and increment the level (can multi-level-up from a Tetris).

### Game Over Detection (Two Paths)
1. **At spawn**: New piece at Y=0 collides with existing blocks → immediate game over
2. **At lock**: Any block of the locked piece has Y < 0 (above visible board) → immediate game over

---

## 5. Game States & Overlay Screens

All overlays are `position: absolute` children of `#board-wrapper`, covering 100% width/height with `background: rgba(0,0,0,0.85)`, centered flex column, `z-index: 10`.

| State | Trigger | Overlay | Active Keys |
|-------|---------|---------|-------------|
| **Title Screen** | Page load | "TETRIS" title, "Press Start to Play", Start Game button | Start button, A (autoplay), V (viz toggle) |
| **Active Gameplay** | Start Game clicked | None | All controls |
| **Paused** | P or Escape | "Paused" + Resume button | P/Escape (resume), S (restart), A, G, V |
| **Game Over** | Piece locks/spawns illegally | "Game Over!" + final stats + Restart button | S (restart), A, V |

- Game does **not** auto-restart on game over, even with autoplay enabled
- Pausing clears the drop interval and all autoplay timers
- Resuming restarts the drop interval and re-schedules autoplay if enabled

---

## 6. Controls & Input Handling

All keyboard input is **case-insensitive** (convert to lowercase).

### System Keys (always active, highest priority)

| Key | Action |
|-----|--------|
| `A` | Toggle AI autoplay |
| `V` | Toggle visualization (all 3 layers) |
| `G` | Toggle ghost piece (only during active gameplay) |
| `P` / `Escape` | Pause/Resume (only during active gameplay) |
| `S` | Restart game — no confirmation on title/game over screen; confirmation prompt during active/paused gameplay |

### Game Controls (blocked during autoplay, pause, game over)

| Key | Action |
|-----|--------|
| `ArrowLeft` | Move left |
| `ArrowRight` | Move right |
| `ArrowDown` | Soft drop |
| `ArrowUp` / `X` | Rotate clockwise |
| `Space` | Hard drop |
| `C` / `Shift` | Hold piece |

---

## 7. UI Components

### Left Panel — Controls, Statistics & High Score

All three sections share a consistent container style matching the right-panel AI panels: semi-transparent dark background (`rgba(0,0,0,0.5)`), subtle cyan border (`1px solid rgba(0,255,255,0.3)`), 8px padding, 6px bottom margin. Each has an uppercase cyan heading (8px).

**Controls section**: Heading "CONTROLS". Each key label in cyan, 7px font, 2.2 line-height.

**Statistics section**: Heading "STATISTICS". Label (gray) / Value (green) rows at 8px font:
- Score, Level, Lines, Speed (in ms), AI Score (1 decimal place, e.g. `-28.0`; "N/A" when autoplay off)

**High Score section**: Heading "HIGH SCORE". Same label/value row styling as Statistics (8px font). Shows Score, Lines, Level loaded from localStorage. Includes "Reset High Score" button with confirmation prompt.

### Center — Game Board + Previews
- Board wrapper with cyan border and glow
- Next and Hold preview grids in a sidebar column, 15px gap between them

### Right Panel — AI Visualization + AI Tuning

**AI Visualization container** (above AI Tuning):
- Semi-transparent dark background with subtle cyan border, 6px padding
- Heading: "AI VIZUALIZATION" (8px, cyan, uppercase) with "Viz ON"/"Viz OFF" badge right-aligned
- Two heatmap grids (Current Piece, Hold Piece) with labels
- Score breakdown bar chart below heatmaps
- Tooltip (absolute positioned, on hover)
- Margin-bottom: 6px

**AI Tuning panel**:
- Heading: "AI Tuning" (8px, cyan, uppercase) with "AI ON"/"AI OFF" badge right-aligned
- Autoplay Delay dropdown (full width): 20ms, 100ms, 200ms (default), 500ms, 1000ms
- Weight sliders in 2-column CSS grid (`grid-template-columns: 1fr 1fr`, `gap: 0 8px`):

| Label | Key | Default | Min | Max | Step |
|-------|-----|---------|-----|-----|------|
| Lines | lines | 2.0 | 0 | 2 | 0.1 |
| Holes | holes | -2.0 | -2 | 0 | 0.1 |
| Bump | bumpiness | -1.0 | -1 | 0 | 0.1 |
| Height | height | -0.01 | -0.1 | 0 | 0.01 |
| Valley | valley | -1.0 | -1 | 0 | 0.1 |
| Top N | overlayTopN | 3 | 1 | 5 | 1 |

- Each slider item has a label row with the label (left) and current value (right, green) on the same line (flexbox `space-between`), with the range input below. Example: `Lines  2.0` above the slider.
- Value formatting: Lines, Holes, Bump, Valley display with 1 decimal place zero-padded (e.g. `2.0`, `-1.0`). Height displays with 2 decimal places zero-padded (e.g. `-0.01`). Top N displays as integer. Apply this formatting consistently on slider input, reset defaults, and initial sync from localStorage.
- Slider labels: 6px, `white-space: nowrap`, abbreviated names
- "Reset Defaults" button restores all weights and Top N to defaults
- All weights persist to localStorage on change
- Padding: 8px, no bottom margin

### Badge Styling
- `.ai-on`: green (`#00FF00`)
- `.ai-off`: red (`#FF4444`)
- 7px font, 2px 6px padding, 3px border-radius

---

## 8. AI Autoplay

### Overview
Toggle with `A` key. Works from any game state (including title screen — the game will auto-play when started). When active, the AI evaluates all possible placements and executes the best move automatically.

### Best Move Calculation (`calculateBestMove`)
For a given piece type and board state:
1. Iterate all 4 rotations
2. For each rotation, try every valid X position (-1 to BOARD_WIDTH)
3. Bounds-check all blocks; skip out-of-bounds positions (mark as invalid)
4. Simulate hard-drop landing position
5. Evaluate the resulting board and record placement with score and breakdown
6. Return: best move + all placements array

**Hold consideration**: If `canHold` is true, also evaluate the hold piece (or next piece if hold is empty). Use whichever yields the higher score. **Important**: All hold-piece placements must be tagged with `hold: true` *unconditionally* (before comparing scores), so that visualization can distinguish current-piece vs hold-piece placements. Only the `bestMove` selection depends on the score comparison.

### Board Evaluation (`evaluateBoardDetailed`)
Five metrics calculated on the post-placement board:

| Metric | How Calculated |
|--------|---------------|
| **Holes** | Count empty cells below any filled cell in each column |
| **Column Heights** | For each column, distance from bottom to highest filled cell |
| **Aggregate Height** | Sum of all column heights |
| **Bumpiness** | Sum of absolute differences between adjacent column heights |
| **Valley Depth** | For each column, if shorter than both neighbors, add the difference to the taller neighbor |
| **Lines Cleared** | Count of complete rows |

**Scoring formula**:
```
score = (linesCleared × lineWeight × 100)
      + (holes × holeWeight × 5)
      + (bumpiness × bumpinessWeight)
      + (aggregateHeight × heightWeight)
      + (valleyDepth × valleyWeight)
      + (gameOverPenalty: -100000 if top row has any filled cell)
```

### Autoplay Move Scheduling (`scheduleAutoplayMove`)
1. Guard: return if autoplay disabled, game not active, paused, or game over
2. Clear all pending autoplay timers
3. **Reset piece** to base rotation and spawn X position (ensures consistency with evaluation)
4. Calculate best move for current piece
5. If canHold, calculate best move for hold/next piece; compare scores
6. Store evaluation data globally for visualization
7. Update AI Score display and render visualization
8. Execute the best move

### Autoplay Move Execution (`executeAutoplayMove`)
Actions are queued and executed sequentially with `autoplayDelay` between each:
1. Hold piece (if using hold)
2. Each rotation step individually
3. Movement chain: recursive left/right moves toward target X, each with `autoplayDelay`
4. Hard drop when at target position

Every action checks `gameActive && !gamePaused && !gameOver && autoplayEnabled` before executing.

### Timer Management
- All autoplay timers stored in an array
- `clearAutoplayTimers()` clears all pending timers
- Called on: pause, game over, restart, autoplay toggle off, before each new move schedule

---

## 9. AI Decision Visualization

Three visualization layers, all toggled together by the `V` key. Default state: **on** (`vizEnabled = true`).

### Data Foundation
The `calculateBestMove` return value is stored globally as `lastAIEvaluation`:
```
{
  bestMove: { rotation, x, hold, score },
  allPlacements: [ { rotation, x, hold, score, valid, breakdown }, ... ],
  currentPieceType, holdPieceType
}
```

**Data lifecycle**:
- Generated each time autoplay schedules a move
- Cleared (set to null + placeholder UI) on: autoplay disable, game restart, game over
- Preserved through: pause/resume, viz toggle off

**Viz toggle off**: Shows placeholder UI but does **not** null the stored data. Toggling back on immediately renders the preserved data.

### Layer 1: Placement Heatmap Matrix

Two grids rendered in the AI Visualization container — one for current piece, one for hold piece.

**Grid layout**: Rows = rotations (R0–R3), Columns = valid X positions. Each cell is 16×16px, 1px gap, 2px border-radius, pointer cursor.

**Cell coloring** (HSL hue interpolation):
- Score range calculated from valid, non-game-over placements
- Normalized linearly to hue 0° (red/worst) through 120° (green/best)
- Saturation: 100%, Lightness: 40%
- If all scores are equal → yellow (hue 60°)

**Special cells**:
- Invalid (out-of-bounds): dark gray (`#333`), class `invalid-cell`
- Game over: black background, `1px solid #FF4444` border, class `game-over-cell`
- Best move: `2px solid #00FFFF` outline, class `best`

**Hold grid**: Always visible. Shows "Hold (N/A)" with placeholder (4×10 gray cells) when no hold piece exists.

**Tooltip** (on hover): Absolute positioned near cursor. Dark opaque background (`rgba(0,0,0,0.95)`), cyan border, 7px monospace font, max-width 160px, `pointer-events: none`. Shows: rotation, X position, (Hold) flag, composite score, and all 5 factor scores.

**Click interaction**: Clicking a valid heatmap cell updates the score breakdown to show that placement's scores.

**Placeholder state**: 4 rows × 10 columns of gray cells with R0–R3 labels. Shown when viz is off or no evaluation data exists.

### Layer 2: Board Overlay — Top N Placements

Rendered during `drawBoard()` when viz is on and evaluation data exists.

**Rendering**:
1. Filter valid, non-game-over placements, sort by score descending
2. Slice to top N (controlled by "Top N" slider, default 3)
3. For each placement, calculate hard-drop landing position
4. Render in reverse rank order (worst first, best last → best on top)
5. Only apply overlay styling to cells not already filled

**Rank styling**:

| Rank | Background | Border |
|------|-----------|--------|
| 1 | `rgba(0,255,0, 0.15)` | `2px solid rgba(0,255,0, 0.8)` |
| 2 | `rgba(200,255,0, 0.10)` | `1px solid rgba(200,255,0, 0.6)` |
| 3 | `rgba(255,255,0, 0.08)` | `1px solid rgba(255,255,0, 0.45)` |
| 4 | `rgba(255,170,0, 0.05)` | `1px dashed rgba(255,170,0, 0.35)` |
| 5 | `rgba(255,68,68, 0.03)` | `1px dashed rgba(255,68,68, 0.25)` |

All overlay cells: `box-shadow: none`.

### Layer 3: Score Breakdown Bar Chart

Displayed below the heatmap grids. Auto-shows the best (highest-scoring) placement's breakdown when evaluation renders.

**Layout**: 5 rows (Lines, Holes, Bump, Height, Valley). Each row: label (50px, gray), bar container (flex, 8px height), value (55px, right-aligned).

**Bars**: Extend from center line — positive (green `#00FF00`) extends right, negative (red `#FF4444`) extends left. Width proportional to `|value| / maxAbsValue × 50%`. Minimum width: 2px.

**Header**: Shows `R{rotation} X{position} = {score}` in cyan 8px font. Defaults to "Best Move" in placeholder state.

**Placeholder state**: All 5 rows with labels, empty bar containers, "--" values in gray (`#666`).

### Board Rendering Order (in `drawBoard`)
1. Locked blocks (board array) — filled cell styling with piece colors
2. Overlay placements (rank N → rank 1, so best renders on top)
3. Ghost piece (if enabled and at different Y than current)
4. Current piece (always on top)

---

## 10. Data Persistence

### localStorage Keys

| Key | Value | Defaults |
|-----|-------|----------|
| `tetrisHighScore` | JSON: `{ score, lines, level }` | `{ score: 0, lines: 0, level: 1 }` |
| `tetrisAIWeights` | JSON: 5 weight properties | Default weights (see AI Tuning table) |

### Error Handling
- Wrap all localStorage access in try/catch
- On failure: log to console, fall back to defaults silently
- No user-visible error alerts

---

## 11. Initialization Sequence

On page load:
1. Initialize board array (20 rows of 10 zeros)
2. Create board DOM (200 cells)
3. Create preview grid DOMs (16 cells each)
4. Load AI weights from localStorage, sync sliders
5. Display high score from localStorage
6. Update statistics display
7. Render visualization placeholder state
8. Draw empty board
9. Title screen overlay is visible — game waits for Start Game click
