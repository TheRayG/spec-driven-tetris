# Spec-Driven Tetris

A requirements-first approach to using AI code generation to build a simple application in a single-pass prompt.

A complete, fully playable classic Tetris game built entirely in a single `index.html` file using vanilla HTML, CSS, and JavaScript. Features a heuristic AI autoplay mode with tunable parameters.

## Play

Open `index.html` in any modern browser. No build tools, no dependencies, no server required.

## Features

### Core Gameplay
- Standard 10x20 Tetris board with all 7 tetromino shapes
- Automatic gravity with level-based speed progression
- Wall kick rotation system
- Lock delay, soft drop, and hard drop mechanics
- Ghost piece showing landing position
- Hold piece and next piece preview
- Line clearing with scoring (100/300/500/800 x level)

### AI Autoplay
- Heuristic-based AI that evaluates all possible placements
- Weighted scoring: lines cleared, holes, bumpiness, aggregate height, valley depth
- Considers hold piece swaps for better moves
- Tunable weights via slider panel with localStorage persistence
- Configurable move execution speed (20ms - 1000ms)
- Toggle on/off with the `A` key at any time

### UI
- Retro aesthetic with Press Start 2P font
- Three-column responsive layout (stacks vertically below 800px)
- Title screen, pause overlay, and game over screen
- Real-time statistics: score, level, lines, speed, AI evaluation score
- High score persistence via localStorage

## Controls

| Key | Action |
|-----|--------|
| Left/Right Arrow | Move piece |
| Up Arrow / X | Rotate clockwise |
| Down Arrow | Soft drop (+1 pt/cell) |
| Space | Hard drop (+2 pts/cell) |
| C / Shift | Hold piece |
| P / Escape | Pause/Resume |
| R | Restart game |
| G | Toggle ghost piece |
| A | Toggle AI autoplay |

## How It Was Built

This project was generated using AI code generation (Claude Code with Anthropic Opus 4.6) from a detailed requirements specification.

1. **Requirements-first approach**: A comprehensive prompt document ([TetrisPrompt.md](TetrisPrompt.md)) was written specifying every game mechanic, UI element, timing detail, and edge case
2. **Single-prompt generation**: The final `index.html` was generated from the refined spec in a single pass using the prompt, "Generate a Tetris Game in a single HTML file using the requirements given in [TetrisPrompt.md](TetrisPrompt.md)"

The spec document serves as both the build instructions and the living documentation for the game's behavior.

## Project Structure

```
index.html       # The complete game (HTML + CSS + JS)
TetrisPrompt.md  # Detailed requirements specification
```

## Browser Support

Chrome 70+, Firefox 65+, Safari 12+, Edge 79+. Requires ES6+ and localStorage.

## License

MIT
