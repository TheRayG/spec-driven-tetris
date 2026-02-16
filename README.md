# Spec-Driven Tetris

A requirements-first approach to using AI code generation to build a single-file application.

A complete, fully playable Tetris game built entirely in a single `index.html` file using vanilla HTML, CSS, and JavaScript. Features a heuristic AI autoplay mode with real-time decision visualization and tunable parameters.

## Play

Open `index.html` in any modern browser. No build tools, no dependencies, no server required.

![Gameplay screenshot](TetrisGameplay.png)

## Features

### Core Gameplay
- Standard 10×20 Tetris board with all 7 tetromino shapes (I, O, T, S, Z, J, L)
- Gravity with level-based speed: `max(50, 1000 - (level - 1) × 25)` ms
- Clockwise rotation with wall kick offsets `[0, -1, 1, -2, 2]`
- Lock delay, soft drop (+1 pt/cell), and hard drop (+2 pts/cell)
- Ghost piece showing landing position (toggleable)
- Hold piece and next piece preview
- Line clearing with scoring: 100/300/500/800 × level
- Level progression with tiered line thresholds (10–20 lines per level)

### AI Autoplay
- Heuristic-based AI that evaluates all possible placements across all rotations and positions
- Five weighted scoring factors: lines cleared, holes, bumpiness, aggregate height, valley depth
- Considers hold piece swaps — evaluates both current and hold piece, picks the higher score
- Tunable weights via slider panel with localStorage persistence
- Configurable move execution speed (20ms–1000ms)
- Toggle on/off with the `A` key from any game state

### AI Decision Visualization
Three visualization layers, all toggled together with the `V` key:

- **Placement Heatmap**: Two rotation×position grids (current piece and hold piece) color-coded red-to-green by score. Cyan outline marks the best move. Click any cell to inspect its breakdown.
- **Board Overlay**: Semi-transparent outlines on the game board showing where the top N ranked placements would land (N configurable 1–5 via slider)
- **Score Breakdown**: Horizontal bar chart showing the 5 weighted factor contributions. Bars extend left (red/negative) or right (green/positive) from a center line.
- Placeholder state (gray grids, "--" values) shown when AI is off or no evaluation data exists

### UI
- Retro aesthetic with Press Start 2P font, dark background, cyan accents, green highlights
- Three-column responsive layout (stacks vertically below 800px)
- All panels share a consistent container style (dark background, cyan border)
- Title screen, pause overlay, and game over screen with final stats
- Real-time statistics: Score, Level, Lines, Speed, AI Score
- High score persistence via localStorage

## Controls

### System Keys (always active)
| Key | Action |
|-----|--------|
| A | Toggle AI autoplay |
| V | Toggle AI visualization |
| G | Toggle ghost piece |
| P / Escape | Pause / Resume |
| S | Restart (confirms during active play) |

### Game Controls (blocked during autoplay, pause, game over)
| Key | Action |
|-----|--------|
| Left / Right Arrow | Move piece |
| Up Arrow / X | Rotate clockwise |
| Down Arrow | Soft drop |
| Space | Hard drop |
| C / Shift | Hold piece |

## How It Was Built

This project was generated using AI code generation (Claude Code with Anthropic Opus 4.6) from a detailed requirements specification.

1. **Requirements-first approach**: A comprehensive specification ([TetrisSpec.md](TetrisSpec.md)) was written covering every game mechanic, UI element, timing detail, and edge case
2. **Single-prompt generation**: The `index.html` was generated from the spec with the prompt: *"Generate a Tetris Game in a single HTML file using the requirements given in TetrisSpec.md"*
3. **Iterative refinement**: The spec was updated as bugs were found and UI improvements were made, creating a living document that stays in sync with the implementation

The spec serves as both the build instructions and the living documentation for the game's behavior.

## Project Structure

```
index.html       # The complete game (HTML + CSS + JS)
TetrisSpec.md    # Detailed implementation specification
README.md        # This file
```

## Browser Support

Chrome 70+, Firefox 65+, Safari 12+, Edge 79+. Requires ES6+ and localStorage.

## License

MIT License — Copyright (c) 2026 TheRayG
