# Generating a Classic Tetris Game in a Single HTML File

Generate a complete and fully playable classic Tetris game using only HTML, JavaScript, and CSS. All code (HTML structure, CSS styling, and JavaScript logic) must be contained within a single `index.html` file. The game should faithfully replicate the core mechanics and features of a classic Tetris experience, suitable for demonstration to a non-technical audience.

## IMPLEMENTATION PRIORITY GUIDE

**For AI Code Generation - Read This First:**

This document contains specifications for a COMPLETE Tetris game with advanced features. For successful implementation, follow this priority order:

**PHASE 1 - CORE:**
- Board rendering with 10×20 grid
- 7 tetromino shapes with colors
- Piece spawning (centered, off-screen start)
- Automatic gravity (pieces fall)
- Left/right movement
- Basic rotation
- Piece locking
- Line clearing
- Basic scoring

**PHASE 2 - ENHANCED:**
- Wall kick rotation
- Lock delay timer
- Soft drop / hard drop
- Next piece preview
- Level progression

**PHASE 3 - ADVANCED:**
- Ghost piece
- Hold functionality
- High score persistence
- Title/pause/game over screens

**PHASE 4 - AI AUTOPLAY:**
- Board evaluation algorithm
- Move calculation
- AI tuning panel
- Autoplay execution

**IMPORTANT:** Implement phases sequentially. Each phase should result in a working game with increasing functionality.

## Game Concepts

This section defines key Tetris terminology and gameplay concepts used throughout this document.

### **Core Terminology:**

- **Tetromino (Piece)**: A geometric shape composed of four square blocks connected orthogonally. There are seven standard tetromino shapes, each named after a letter they resemble: I, O, T, S, Z, J, and L. Throughout this document, "tetromino," "piece," and "falling piece" are used interchangeably.

- **Board (Matrix/Playing Field)**: The rectangular grid where gameplay occurs, typically 10 blocks wide by 20 blocks high. Also referred to as "game board," "matrix," or "playing field." The board is represented internally as a 2D array where each cell can be empty (0) or contain a locked block from a previously placed piece (1-7).

- **Block (Cell)**: A single square unit that makes up both tetrominoes and the board grid. Empty cells on the board are displayed with subtle borders for visibility. Filled cells display the color of the tetromino that placed them.

### **Spatial Concepts:**

- **Spawn Position**: The location where new tetrominoes first appear. In this implementation, pieces spawn **fully off-screen** above the visible board (at negative Y coordinates), then descend into view. Horizontal spawn position is centered on the board.

- **Visible Board**: The 10×20 grid that players can see. The coordinate system uses Y=0 for the top row and Y=19 for the bottom row.

- **Off-Screen Area**: The region above the visible board (negative Y coordinates) where pieces initially spawn. Pieces in this area are not visible until they descend onto the board.

- **Collision**: A state where a tetromino's blocks would overlap with the board boundaries or existing locked blocks. Collision detection prevents invalid piece positions.

### **Gameplay Mechanics:**

- **Automatic Descent (Gravity)**: The continuous downward movement of the active tetromino at regular intervals determined by the current level. This is the primary game loop mechanism.

- **Active Piece (Current Piece)**: The tetromino currently falling and under player control (or AI control during autoplay).

- **Piece Locking**: The process of permanently placing the active piece's blocks onto the board when it can no longer move downward. After locking, a new piece spawns.

- **Lock Delay**: A grace period (equal to the current drop speed) that begins when a piece touches the ground. During this time, the player can still move or rotate the piece, but the timer is **not** reset by these actions. Once the timer expires, the piece locks regardless of any movement or rotation performed during the delay. This gives players a fixed window to adjust placement.

- **Soft Drop**: Manual downward movement triggered by holding the Down arrow key. Awards +1 point per cell moved. When collision is detected during soft drop, the piece locks immediately (bypassing lock delay).

- **Hard Drop**: Instant placement of the active piece to its lowest possible position, triggered by the Spacebar. Awards +2 points per cell dropped. The piece locks immediately upon landing (bypassing lock delay).

- **Rotation**: Turning the active piece 90 degrees clockwise. Uses matrix transformation (transpose + reverse rows) to calculate the new shape.

- **Wall Kick**: A rotation assistance system that tries alternative horizontal positions [0, -1, +1, -2, +2] when rotation at the current position would cause a collision. This allows pieces to rotate near walls and obstacles.

### **Scoring and Progression:**

- **Line Clearing (Line Completion)**: When a horizontal row is completely filled with blocks (no gaps), that line is removed from the board. All blocks above the cleared line(s) shift downward to fill the empty space.

- **Score**: Points awarded for clearing lines (100/300/500/800 × level for 1/2/3/4 lines) and for drop actions (+1 for soft drop, +2 for hard drop per cell).

- **Level**: A difficulty indicator that increases after clearing a threshold number of lines. Higher levels increase the automatic descent speed (gravity), making the game progressively more challenging.

- **High Score**: The best score achieved across all game sessions, persisted in browser localStorage along with the lines cleared and level reached during that high score run.

### **Advanced Features:**

- **Next Piece Preview**: A 4×4 grid display showing which tetromino will spawn after the current piece locks. Allows players to plan ahead.

- **Hold Piece**: A storage slot that allows the player to save one tetromino for later use. Pressing the Hold key swaps the active piece with the held piece. This can only be done once per piece (resets when a piece locks).

- **Ghost Piece**: A transparent outline showing where the active piece would land if hard-dropped immediately. Serves as a visual aid for precise placement. Can be toggled on/off with the 'G' key.

- **Autoplay Mode (AI)**: An automated gameplay mode where an algorithm evaluates all possible placements and executes the optimal move based on weighted scoring criteria (lines cleared, holes created, surface bumpiness, stack height, valley depth). Can be toggled with the 'A' key.

### **Game States:**

- **Title Screen**: The initial state when the page loads, displaying the game title and a "Start Game" button. No gameplay occurs until this button is clicked.

- **Active Gameplay**: The main game state where pieces fall, players provide input, and the game loop runs.

- **Paused**: A suspended state triggered by the 'P' or Escape key. All timers are paused, and a "Paused" overlay is displayed. Press 'P' or Escape again to resume.

- **Game Over**: The terminal state reached when a piece locks with any blocks above the visible board (negative Y coordinate). Displays final statistics and offers a restart option.

### **Technical Concepts:**

- **Drop Interval Timer**: A repeating timer (setInterval) that calls the automatic downward movement function at regular intervals based on the current game speed.

- **Drop Speed**: The time interval between automatic downward movements, calculated as max(50, 1000 - (level-1) × 25) milliseconds. Ranges from 1000ms at level 1 to a minimum of 50ms at level 39+.

- **Accelerated Initial Descent**: A special mechanic where newly spawned pieces fall at 100ms intervals for their first 3 drops, then transition to normal level-based speed. This reduces perceived delay when pieces spawn off-screen.

- **Drop Counter**: A counter that tracks how many automatic drops have occurred for the current piece, used to implement the accelerated initial descent feature.

- **localStorage**: Browser API for persistent data storage. Used to save high scores and AI tuning parameters across browser sessions.

### **Coordinate System:**

- **X Coordinate**: Horizontal position on the board (0 = leftmost column, 9 = rightmost column for standard 10-wide board)
- **Y Coordinate**: Vertical position on the board (0 = top row, 19 = bottom row for standard 20-tall board, negative values = off-screen above board)
- **Position**: Refers to the top-left corner of the tetromino's bounding box relative to the board grid

### **AI Terminology:**

- **Board Evaluation**: The process of analyzing a hypothetical board state and assigning a numerical score based on weighted criteria.

- **Holes**: Empty cells with filled blocks directly above them. Holes are difficult to clear and heavily penalized by the AI.

- **Bumpiness**: A measure of surface unevenness, calculated as the sum of absolute height differences between adjacent columns.

- **Aggregate Height**: The sum of all column heights (where height = number of blocks from bottom to topmost filled cell in each column).

- **Valley Depth**: A measure of how much lower each column is compared to its adjacent neighbors. Deep valleys are hard to fill efficiently.

- **Best Move**: The piece placement (rotation + position + optional hold) that produces the highest evaluation score according to the AI's weighted criteria.

## Initial Page Load State:

When the page first loads, the game should be in the following state:

*   **Title Screen Displayed:** A title screen overlay is visible, positioned over the game board, displaying:
    *   Game title: "TETRIS"
    *   Subtitle: "Press Start to Play"
    *   A "Start Game" button (the only interactive element at this stage)
*   **Game Board Rendered:** The game board is visible beneath the title screen overlay:
    *   All cells are empty (no tetrominoes placed)
    *   Grid borders are visible on empty cells: `border: 1px solid rgba(0, 0, 0, 0.1)`
    *   Board dimensions: 10 blocks wide × 20 blocks high
*   **UI Panels Visible:** All control panels and information displays are rendered:
    *   Left panel: Controls list and game statistics area
    *   Center: Game board with Next/Hold preview boxes
    *   Right panel: AI tuning controls
*   **Initial Statistics Display:**
    *   Score: 0
    *   Level: 1
    *   Lines: 0
    *   Speed: 1000ms (level 1 default)
    *   AI Score: "N/A" (no autoplay active)
    *   High Score: Loaded from localStorage and displayed (or 0 if no saved data)
*   **Preview Boxes Empty:**
    *   Next piece preview: Empty 4×4 grid
    *   Hold piece preview: Empty 4×4 grid
*   **No Active Timers:** 
    *   No game loop running
    *   No drop interval timer active
    *   No autoplay timers active
*   **Interactivity Restrictions:**
    *   Only the "Start Game" button responds to clicks
    *   'A' key (autoplay toggle) is functional even on title screen
    *   All other keyboard controls are inactive until game starts
    *   UI controls (sliders, dropdowns) are visible but game-related inputs are blocked

Once the user clicks "Start Game", the title screen overlay is hidden, the first tetromino spawns off-screen, game timers begin, and full keyboard controls become active.

## Game Loop Architecture:

This section describes how the various game mechanics integrate into a cohesive game loop, clarifying the execution order and timer interactions.

### **Main Drop Interval Timer:**

*   Implemented via a repeating interval timer
*   Calls the automatic downward movement function at intervals determined by current game speed (level-based speed or 100ms for first 3 drops)
*   Runs continuously during active gameplay
*   Paused when game is paused or game over
*   Cleared and restarted when speed changes (level up) or when drop count transitions from fast (100ms) to normal

### **Automatic Downward Movement Flow:**

1. Attempt to move the current piece down one cell by checking for collisions at the new position
2. **If no collision (piece can move down):**
   - Update piece position to new location
   - Clear any existing lock delay timer (piece is still falling freely)
   - Increment drop counter (tracks fast drop phase)
   - Redraw the board to show new position
   - Return success
3. **If collision detected (piece cannot move down):**
   - Start lock delay timer with duration equal to current drop speed (e.g., 1000ms at level 1)
   - Lock delay timer will finalize piece placement when it expires
   - During lock delay, player can still move/rotate the piece
   - Return collision status

### **Lock Delay Timer Management:**

*   **Purpose:** Gives players time to adjust piece position after it lands
*   **Duration:** Equal to current drop speed
*   **Activation:** Started when automatic downward movement detects collision
*   **No Reset on Movement:** Horizontal movement and rotation do **not** reset or restart the lock delay timer. The timer runs for its full duration regardless of player input.
*   **Expiration:** When timer completes, the piece is locked to the board
*   **Exceptions that bypass lock delay:**
   - Hard drop (Spacebar): Immediately locks piece without starting lock delay timer
   - Soft drop (Down Arrow): When collision detected, immediately locks piece without starting lock delay timer

### **Piece Locking Flow:**

1. Copy all blocks from current tetromino to the board array at current position
2. **Game Over Check:** Iterate through the piece's blocks and check if any are positioned above the visible board (negative Y coordinate or row index below 0)
   - If yes: Trigger game over sequence and return immediately (do not spawn next piece)
   - If no: Continue to next step
3. Check for and remove any completed horizontal lines
4. Reset hold availability flag (allow hold function for next piece)
5. Reset drop counter (reset fast drop counter for next piece)
6. Spawn the next falling tetromino

### **Piece Spawning Flow:**

1. If a "next piece" exists in queue, use it as current piece; otherwise generate random piece
2. Generate new "next piece" for preview display
3. Calculate spawn position:
   - Horizontal (X): Centered horizontally on the board
   - Vertical (Y): Fully off-screen above board (negative Y position equal to piece height)
4. Set current piece state variables
5. Update next piece preview display
6. If autoplay is enabled, schedule AI move calculation
7. Restart drop interval timer

**No collision check needed at spawn** - pieces always spawn at negative Y coordinates where collision detection doesn't validate. Game over detection happens only during piece locking (see Piece Locking Flow).

### **Line Clearing Flow:**

1. Scan board array from bottom to top, identifying rows where all cells are filled (no gaps)
2. Remove completed lines from the board array
3. Add empty rows to top of board for each removed row
4. Calculate score bonus based on number of lines cleared (100/300/500/800 × level)
5. Update score and total lines cleared counters
6. Update level progress counter (lines cleared since last level up)
7. **Check for Level Up:** If level progress threshold reached:
   - Increment level
   - Reset level progress counter
   - Update drop speed for new level
8. Refresh score/high score display
9. Refresh statistics panel
10. Redraw entire board to show cleared lines and shifted blocks

### **Timer Interaction Summary:**

*   **Drop Interval Timer** (runs continuously): Calls automatic downward movement at regular intervals
*   **Lock Delay Timer** (conditional): Started by downward movement on collision, locks piece on expiration
*   **Autoplay Timers** (when enabled): Schedule AI move execution steps with configurable delay between actions
*   **Critical Rule:** Only one lock delay timer should be active at a time. Movement/rotation during lock delay does **not** reset the timer; it runs to completion.

### **State Transitions:**

```
Title Screen → [Start Game clicked] → Active Gameplay
Active Gameplay → [P/Escape pressed] → Paused
Paused → [P/Escape pressed] → Active Gameplay
Active Gameplay → [Piece locks with blocks above board] → Game Over
Game Over → [R pressed or Restart button] → Active Gameplay (new game starts immediately)
```

### **Input Priority During Active Gameplay:**

1. **System Keys** (highest priority): P/Escape (pause), R (restart), A (autoplay toggle), G (ghost toggle)
2. **Game Controls** (blocked during autoplay): Arrow keys, Spacebar, X (rotate), C/Shift (hold)
3. **Lock Delay Interaction:** Movement/rotation inputs are allowed during lock delay but do **not** reset the lock delay timer. The piece locks when the timer expires regardless of player actions.

## Core Game Features:

1.  **Game Board (Matrix):**
    *   A standard Tetris playing field, typically 10 blocks wide and 20 blocks high.
    *   Visually represent individual blocks on the grid with subtle cell borders: empty cells have `border: 1px solid rgba(0, 0, 0, 0.1)` for grid visibility.

2.  **Tetrominoes (Falling Pieces):**
    *   Include all seven standard Tetris shapes (I, O, T, S, Z, J, L).
    *   Each tetromino should be composed of four square blocks.
    *   Each type of tetromino should have a distinct color using CSS color names:
        *   I-piece (line): 'cyan'
        *   O-piece (square): 'yellow'
        *   T-piece: 'purple'
        *   S-piece: 'green'
        *   Z-piece: 'red'
        *   J-piece: 'blue'
        *   L-piece: 'orange'

3.  **Piece Movement and Rotation:**
    *   **Automatic Descent:** Tetrominoes should automatically fall from the top of the game board. This automatic descent is crucial because new pieces initially spawn off-screen, above the visible playing area, and need to descend into view.
    *   **Accelerated Initial Descent:** To reduce the perceived delay when a new tetromino spawns off-screen, it should initially fall at exactly 100ms intervals for the first 3 drops before reverting to the normal level-based speed.
    *   **Left/Right Movement:** Players must be able to move the falling tetromino horizontally left and right within the game board boundaries.
    *   **Soft Drop:** Implement a "soft drop" mechanism that moves piece down one cell immediately, awards +1 point per cell moved, and returns success/failure status. Soft drop is activated by pressing the Down Arrow key. Holding the Down Arrow key continuously triggers repeated soft drops via the browser's native key repeat, allowing rapid descent.
    *   **Hard Drop:** Implement a "hard drop" mechanism, where pressing a key instantly drops the current piece to the lowest possible position on the board.
    *   **Rotation:** Tetrominoes must be rotatable by 90-degree increments (clockwise) while falling. The rotation should respect the game board boundaries and existing stacked blocks. All 7 pieces use the same rotation logic; the O-piece (2×2 square) is symmetric, so rotation produces no visual change but the same code path executes. Implement wall kick logic that attempts rotation at multiple horizontal offsets in order: [0, -1, +1, -2, +2]. **Wall Kick Edge Case Handling:** If a kick attempt would place any part of the tetromino outside the board boundaries (x < 0 or x >= BOARD_WIDTH), that kick is automatically rejected and the next kick in sequence is attempted. Pieces cannot be kicked to positions that would place blocks beyond board edges. Wall kicks only adjust horizontal position; the Y coordinate remains unchanged during rotation attempts.
    
    **Wall Kick Algorithm:**
    
    The wall kick system allows pieces to rotate even when initially blocked, by testing alternative horizontal positions. The algorithm works as follows:
    
    1. **Calculate Rotated Shape:** Apply 90-degree clockwise matrix transformation to the piece (transpose matrix and reverse each row)
    2. **Try Multiple Offsets:** Test rotation at horizontal offsets in this order: current position (0), left 1 (-1), right 1 (+1), left 2 (-2), right 2 (+2)
    3. **Bounds Validation:** For each offset:
       - Check if all blocks of the rotated piece would be within horizontal board boundaries
       - If any block would be outside boundaries (x < 0 or x >= board width), reject this offset and try next
    4. **Collision Check:** For offsets that pass bounds validation:
       - Check if the rotated piece at the offset position collides with existing blocks on the board
       - If no collision, accept the rotation and apply the position adjustment
       - If collision, try next offset
    5. **Lock Delay Unchanged:** If rotation succeeds while a lock delay timer is active, the timer continues running and is **not** reset. The piece will still lock when the timer expires.
    6. **Failure:** If all offsets fail (all produce either bounds violations or collisions), reject the rotation entirely
    
    **Key Points:**
    
    - The offset value adjusts the piece's horizontal position: 0 = current position, -1 = one cell left, +1 = one cell right, etc.
    - Bounds checking happens before collision checking to prevent array index errors
    - Y coordinate never changes - only horizontal adjustments are made
    - First successful kick wins - algorithm stops as soon as valid rotation found
    - Lock delay is **not** reset on successful rotation; the timer continues running

4.  **Line Clearing:**
    *   When a complete horizontal line of blocks is formed without any gaps, that line should disappear.
    *   Blocks above the cleared line(s) should fall down to fill the empty space.
    *   **Line Clearing Animation:** No flash animation is required. When complete lines are detected, they are immediately removed and empty rows added to the top, followed by immediate visual update.

5.  **Scoring System:**
    *   Award points for clearing lines using this exact scoring system:
        *   Single line clear: 100 × current level
        *   Double line clear: 300 × current level
        *   Triple line clear: 500 × current level
        *   Tetris (4 lines): 800 × current level
    *   Display the current score prominently.
    *   Points should also be awarded for drops using this system:
        *   Soft drop: +1 point per cell moved down
        *   Hard drop: +2 points per cell moved down
    *   **High Score Tracking:** Implement a high score feature that tracks and displays the highest score achieved across multiple game sessions, along with the total lines cleared and the level achieved during that high score session. All three values should be persisted using `localStorage` in the user's browser. The high score display should show the score, lines cleared, and level achieved (e.g., "1500" for score, "12" for lines, "3" for level). When a new high score is achieved, the score, current session's total lines cleared, and current level are all saved together.
    
    **High Score Logic:**
    
    When updating the score display:
    - Check if current score exceeds stored high score
    - If yes, update high score along with current total lines cleared and current level
    - Persist all three values to localStorage as a snapshot
    - Update high score display
    
    This check happens every time the score changes (after line clearing, after drop points, after level changes).
    
    *   **Reset High Score:** Provide a reset button to clear the stored high score (and associated lines cleared and level) with confirmation dialog: "Are you sure you want to reset the high score? This action cannot be undone."

6.  **Game Over Condition:**
    *   The game ends when a piece locks with any block positioned above the visible board (negative Y coordinate). This is detected when finalizing piece placement by checking if any block has a Y position below 0. Note: The collision check at spawn time also exists but effectively never triggers because pieces spawn at fully negative Y coordinates where collision checks only validate visible board cells. The primary and reliable game-over mechanism is the piece locking above-board check.
    *   Display a "Game Over!" message with final score, total lines cleared, and level achieved, then offer a restart button labeled "Restart Game".
    *   **Title Screen:** On initial page load, a title screen overlay is displayed with the game title "TETRIS", subtitle "Press Start to Play", and a "Start Game" button. The game does not begin until the user clicks this button. The title screen is an overlay positioned over the game board.

7.  **Game Progression (Levels/Speed):**
    *   The speed at which tetrominoes fall should gradually increase using this system:
        *   Level progression: Line-based advancement. Track lines cleared since the last level-up. When the threshold is reached for the current level, advance to the next level and reset the counter.
        *   **Lines required per level (based on current level):**
            *   Level 1–9: 10 lines cleared per level
            *   Level 10–19: 12 lines cleared per level
            *   Level 20–29: 14 lines cleared per level
            *   Level 30–39: 16 lines cleared per level
            *   Level 40–49: 18 lines cleared per level
            *   Level 50+: 20 lines cleared per level
        *   Drop speed calculation: max(50, 1000 - (level-1) × 25) milliseconds
        *   Level 1: 1000ms, Level 2: 975ms, Level 3: 950ms, etc., minimum 50ms
        *   **Speed reaches minimum (50ms) at level 39:** All levels 39+ remain at 50ms.
    *   Display the current level.

8.  **User Interface (UI) Elements:**
    *   **Next Piece Preview:** Show a preview of the next falling tetromino.
    *   **Hold Piece Functionality:** Allow the player to store one tetromino for later use and swap it with the current falling piece. **Hold Sequence:** When "Hold" key is pressed:
        *   If hold slot is empty: Current piece immediately moves to hold slot, next piece becomes current piece, new next piece is generated
        *   If hold slot occupied: Current piece swaps with held piece, held piece immediately becomes active at spawn position
        *   The piece placed into hold slot has its orientation reset to initial upright position
        *   Current piece positioning is immediately updated to spawn location (centered horizontally, fully off-screen vertically)
        *   Hold restriction: Can only be used once per piece - availability resets when a piece locks
    *   **Ghost Piece:** Implement functionality to display a transparent outline showing where the current falling piece will land if immediately hard-dropped. The visibility of this ghost piece should be toggleable by the user.
    
    **Ghost Piece Position Calculation:**
    
    The ghost piece shows the landing position of the current piece if hard-dropped immediately:
    
    1. Start from current piece vertical position
    2. Keep moving down one cell at a time until a collision is detected
    3. The position just before collision is the ghost position
    4. Don't render if ghost piece feature is disabled by user
    5. Don't render if ghost would be at same position as current piece
    6. Render ghost piece blocks at calculated position using special styling (30% opacity, dashed white border, transparent background)
    7. Only render blocks within visible board boundaries
    
    **Ghost Piece Update Timing:**
    
    Recalculate and redraw ghost piece whenever:
    - Current piece moves horizontally (left/right)
    - Current piece rotates
    - Current piece moves down (automatic or manual)
    - Board state changes (lines cleared, piece locked)
    
    Implementation should redraw ghost as part of the main board rendering.
    
    **Key Points:**
    
    - Ghost position uses same collision detection as regular movement
    - Ghost rendering is skipped if disabled or if at same position as current piece
    - Ghost blocks must have transparent background - only the dashed white border is visible
    - Ghost calculation never modifies piece state, only calculates hypothetical landing position
    
    *   **Controls Display:** The keyboard controls for the game should be clearly listed on the left side of the game board (not below it), to ensure they are always visible and easily accessible within the viewport.
    *   **Autoplay Delay Configuration:** An on-screen dropdown element must allow users to configure the delay between automated moves in autoplay mode with these exact options: 20ms (Very Fast), 100ms (Fast), 200ms (Normal), 500ms (Slow), 1000ms (Very Slow).
    *   **Game Statistics:** Display real-time game statistics including:
        *   Total lines cleared during the current session (updated immediately when lines are cleared)
        *   Current drop speed in milliseconds (always shows level-based speed regardless of internal fast drop mechanics for first 3 drops)
        *   AI evaluation score when autoplay is active (updated when AI calculates best move with 1 decimal place precision, shows "N/A" during manual play)
        *   **Statistics Refresh:** Statistics update immediately on each triggering event with direct visual updates. No debouncing or batching is applied.
    *   **AI Tuning Panel:** Provide an advanced AI tuning interface with adjustable parameters for fine-tuning the autoplay algorithm:
        *   Lines Cleared weight (positive bonus for clearing lines)
        *   Holes penalty weight (negative penalty for creating holes)
        *   Bumpiness penalty weight (negative penalty for uneven surface)
        *   Height penalty weight (negative penalty for tall stacks)
        *   Valley Depth penalty weight (negative penalty for deep column gaps)
        *   Reset button to restore default AI weights
        *   All weights should be persistently saved using `localStorage`

9.  **Keyboard Controls:**
    *   **Move Left:** Left Arrow Key
    *   **Move Right:** Right Arrow Key
    *   **Rotate:** Up Arrow Key (or 'X' key - case insensitive)
    *   **Soft Drop:** Down Arrow Key
    *   **Hard Drop:** Spacebar
    *   **Hold:** 'C' key (case insensitive) or Shift key
    *   **Pause/Resume:** 'P' key (case insensitive) or Escape key. Displays pause screen with "Paused" heading and "Resume" button. **Pause State Preservation:** When paused, drop counter and timing states are preserved. Upon resume, the drop interval timer starts fresh - any elapsed time within the current drop tick before pausing is lost. This is intentional to simplify implementation. However, the accelerated initial descent phase continues from where it left off.
    *   **Restart Game:** 'R' key (case insensitive). If game over screen is displayed, restart directly without confirmation. Otherwise, show confirmation prompt: "Are you sure you want to restart the game? Your current score will be lost." All other game states require confirmation. Note: The restart confirmation works while paused - no need to unpause first. Restarting always goes straight to a new game (no title screen).
    *   **Toggle Ghost Piece:** 'G' key (case insensitive) to toggle the visibility of the ghost piece on and off.
    *   **Toggle Autoplay Mode:** 'A' key (case insensitive) to switch the autoplay mode on or off.
    *   **Key Handling:** All keyboard input is case-insensitive

## Advanced Feature: Autoplay Mode:

*   **Autoplay Functionality:** The game must include an autoplay mode, toggled on/off with the 'A' key at any time - including before the first game starts (title screen), during gameplay, when paused, or on the game over screen. If autoplay is enabled before starting, the game will auto-play immediately upon pressing Start. When active, the system should automatically play the game without user intervention. **Autoplay Visual Indicator:** A status badge is right-aligned within the AI Tuning panel heading showing "AI ON" (in green, #00FF00) when autoplay is active or "AI OFF" (in red, #FF4444) when inactive. **Autoplay + Gravity Interaction:** The normal drop interval continues running during autoplay move execution. This means gravity can cause the piece to fall while the AI is still performing rotations and horizontal movements. At slower autoplay delays, the piece may descend several rows before the AI completes its positioning sequence. This is intended behavior - the AI works within the same gravity constraints as a human player.

*   **Autoplay + Manual Input Interaction:**
    
    When autoplay is active, user input is handled according to these rules:
    
    **Input Blocking for Game Controls:**
    All manual game controls are blocked at the start of their handler functions:
    - Arrow keys (Left, Right, Down, Up)
    - Spacebar (hard drop)
    - X key (rotation)
    - C/Shift (hold piece)
    
    **Allowed Keys (always functional):**
    Meta-controls remain active:
    - 'A' key: Toggle autoplay on/off
    - 'P'/Escape: Pause/resume game
    - 'R' key: Restart game (with confirmation if not on game over screen)
    - 'G' key: Toggle ghost piece visibility
    
    **Toggling Autoplay Mid-Execution:**
    
    If the user presses 'A' to disable autoplay while the AI is executing moves:
    
    1. Set autoplay flag to false immediately
    2. Clear all pending scheduled AI action timers
    3. Current piece remains at its current position with its current shape
    4. All manual controls become immediately responsive
    5. Remaining AI moves are cancelled - no completion of AI sequence
    
    **Key Implementation Points:**
    
    - Manual input blocking uses early return check at start of all movement handlers
    - Toggling autoplay off clears all pending AI action callbacks
    - Current piece state is preserved when switching from autoplay to manual
    - No "finishing the current move" logic - disabling autoplay cancels all pending AI actions immediately
    - Re-enabling autoplay during active gameplay triggers AI calculation for the current piece
    
*   **Best Move Calculation:** The autoplay mechanism must intelligently calculate the "best move" for each new tetromino. This calculation should be based on a comprehensive scoring algorithm with five weighted parameters:
    *   **Lines Cleared (Weight: +2.0):** Positive bonus for clearing lines
    *   **Holes Penalty (Weight: -2.0):** Negative penalty for creating "holes" (empty cells with filled blocks above them) which hinder future line clearing opportunities.
    *   **Bumpiness Penalty (Weight: -1.0):** Negative penalty for creating uneven surface topology, measured as the sum of absolute height differences between adjacent columns.
    *   **Aggregate Height Penalty (Weight: -0.01):** Small negative penalty for overall board height to encourage keeping the board low.
    *   **Valley Depth Penalty (Weight: -1.0):** This penalty measures valley/well depth - penalizes columns that are lower than their adjacent neighbors by any amount, creating wells or valleys that are hard to fill. Calculated as sum of height differences where adjacent columns are taller.
    *   **Survival Priority:** Apply a penalty of -100,000 points to any move that would result in game over by placing blocks at the top row of the visible game board.
    *   **Advanced Evaluation:** The algorithm should systematically evaluate all possible rotations and horizontal positions for the current falling piece, and consider using the held piece if it yields a better outcome.
*   **Autoplay Delay:** Users configure the delay via dropdown control with options: 20ms (Very Fast), 100ms (Fast), 200ms (Normal, default), 500ms (Slow), 1000ms (Very Slow). **The best move evaluation itself happens instantly with no delay.** Once the best move is calculated, the configured delay applies to each individual step the AI takes to execute that move. The first action executes after waiting the configured milliseconds, then each subsequent action also waits before executing, including the final hard drop. This makes the AI's decision-making process visible to the user.

## Implementation Reference

This section provides a quick reference for commonly mentioned concepts throughout this document.

### **Core Game Concepts:**

- **Soft Drop Function**: Moves the current piece down by one cell immediately, awards +1 point per cell moved, and returns true if successful or false if the piece locks upon collision.
- **Hard Drop Function**: Instantly moves the current piece to the lowest possible position, awards +2 points per cell dropped, then immediately locks the piece.
- **Lock Piece Function**: Finalizes the current piece's position on the board, checks for game over conditions (blocks above visible area), triggers line clearing, and spawns the next piece.
- **Spawn Piece Function**: Creates a new falling tetromino at the spawn position (horizontally centered, fully off-screen above the board).
- **Rotate Clockwise Function**: Rotates the current piece 90 degrees clockwise, implementing wall kick logic to handle edge cases and collisions.
- **Move Horizontal Functions**: Move the current piece left or right by one cell if no collision occurs.
- **Move Down Function**: Moves the current piece down by one cell if possible, or starts the lock delay timer if collision detected.
- **Check Collision Function**: Validates whether a piece at a given position would collide with board boundaries or existing blocks.
- **Clear Lines Function**: Detects complete horizontal lines, removes them, shifts blocks down, updates score and line count.
- **Game Over Function**: Ends the game, displays the game over screen with final statistics, and clears active timers.

### **Level & Speed Concepts:**

- **Get Lines Per Level Function**: Returns the number of lines required to advance from the given level (10-20 lines depending on level range).
- **Get Drop Speed Function**: Calculates the current automatic drop interval in milliseconds based on the current level: max(50, 1000 - (level-1) × 25).
- **Set Drop Interval Function**: Establishes or resets the automatic piece descent timer, choosing between fast drop speed (100ms for first 3 drops) or normal level-based speed.

### **UI Update Concepts:**

- **Update Score Display Function**: Updates the score display and checks if a new high score has been achieved (saves to localStorage if so).
- **Update Statistics Function**: Refreshes the statistics display (lines cleared, speed, AI score) with current values.
- **Refresh Speed Display Function**: Updates the displayed drop speed value to show the current level-based speed.
- **Update Preview Functions**: Render the preview grids showing the next piece and held piece.

### **Autoplay Concepts:**

- **Schedule Autoplay Move Function**: Initiates the autoplay sequence by calling the best move calculation for the current piece.
- **Calculate Best Move Function**: Evaluates all possible placements (rotations, positions, hold options) and returns the highest-scoring move based on AI weights.
- **Execute Autoplay Move Function**: Performs the individual steps of the AI's chosen move (rotations, movements, hold, hard drop) with configured delays between each action.

### **Data Persistence Concepts:**

- **Load/Save High Score Functions**: Load/save the high score data (score, lines, level) from/to browser localStorage.
- **Load/Save AI Weights Functions**: Load/save the AI tuning parameters from/to browser localStorage.

### **Key Variables:**

- **Board Dimensions**: 10 blocks wide, 20 blocks high
- **Tetromino Shapes**: Seven standard pieces (I, O, T, S, Z, J, L) defined as 2D arrays
- **Tetromino Colors**: Array of color names (cyan, yellow, purple, green, red, blue, orange)
- **Current Drop Count**: Counter tracking drops for fast drop phase (first 3 drops at 100ms)
- **Level Line Count**: Counter tracking lines cleared since last level advancement
- **Current Position**: Variables storing the current piece's board coordinates
- **Can Hold Flag**: Boolean controlling hold function availability
- **AI Weights**: Object containing five evaluation weights
- **Default AI Weights**: Default values for reset functionality
- **Autoplay Delay**: Configured delay in milliseconds between AI actions
- **Autoplay Enabled Flag**: Boolean indicating if autoplay mode is active
- **Initial Fast Drop Count**: How many fast drops occur when piece spawns (3)

---

## Definition of Done - Implementation Checklist

A successful implementation MUST satisfy ALL of these criteria:

**✓ Core Functionality:**
- [ ] Page loads without JavaScript errors in console
- [ ] All 7 tetromino shapes (I, O, T, S, Z, J, L) spawn correctly
- [ ] Pieces fall automatically at level-based speed
- [ ] Left/Right arrow keys move pieces horizontally
- [ ] Up arrow or X key rotates pieces clockwise
- [ ] Rotation works near edges (wall kicks function)
- [ ] Pieces lock when they can no longer move down
- [ ] Completed horizontal lines disappear
- [ ] Blocks above cleared lines fall down
- [ ] Score updates correctly (100/300/500/800 × level)
- [ ] Level increases after threshold lines cleared
- [ ] Game ends when piece locks with blocks above board

**✓ Enhanced Features:**
- [ ] Down arrow performs soft drop (+1 point per cell)
- [ ] Spacebar performs hard drop (+2 points per cell)
- [ ] Next piece preview shows upcoming tetromino
- [ ] 'C' key or Shift swaps current piece with hold slot
- [ ] Hold can only be used once per piece
- [ ] Ghost piece shows landing position (if enabled)
- [ ] 'G' key toggles ghost piece visibility

**✓ UI and Persistence:**
- [ ] Title screen displays on page load
- [ ] "Start Game" button begins gameplay
- [ ] 'P' or Escape pauses/resumes game
- [ ] 'R' key restarts game (with confirmation)
- [ ] High score persists across page reloads
- [ ] Current score, level, lines, speed all display correctly
- [ ] Layout is responsive (works on mobile and desktop)

**✓ AI Autoplay:**
- [ ] 'A' key toggles autoplay mode
- [ ] AI plays game automatically when enabled
- [ ] AI tuning sliders adjust gameplay strategy
- [ ] AI weights persist in localStorage

## Technical Requirements:

*   **Single HTML File:** All HTML, CSS, and JavaScript code must be self-contained within one `index.html` file.
*   **Pure HTML, JavaScript, and CSS (with CDN allowed for libraries):** Use vanilla JavaScript. External JavaScript libraries are permitted only if they can be loaded via a CDN link within the `index.html` file. Do not use any build tools.
*   **Visual Design:** The game should have a clean, classic Tetris aesthetic with:
    *   Font specification: Use 'Press Start 2P' from Google Fonts as primary font, with fallback to cursive. **Font Loading Fallback:** If Google Fonts CDN is unavailable, the browser automatically falls back to system cursive fonts. Include the Google Fonts link: `<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">`
    *   Color scheme: Dark background (#222222), cyan accents (#00FFFF), white text (#FFFFFF), green highlights (#00FF00)
    *   Exact layout specification: Three-column flexbox layout with:
        *   Left column: Controls list and game statistics (min-width: 180px, flexible)
        *   Center column: Game board container with 20px internal gaps (fixed 250px game board + 120px sidebar + padding)
        *   Right column: AI tuning panel (min-width: 180px, flexible)
        *   **Column spacing**: 40px gaps between main sections, 20px internal gaps in game container
        *   **Responsive behavior**: At viewport widths ≥800px, three columns display side-by-side with 40px gaps. Below 800px, columns stack vertically centered with 20px gaps, reordered so game board appears first, followed by controls/stats, then AI panel. No min-width on body.
        *   **Alignment**: All columns align to flex-start with center justification for the overall layout
    *   Cyan-tinted glow effects using CSS box-shadow: `0 0 20px rgba(0, 255, 255, 0.5)`
*   **Responsiveness:** The game layout is fully responsive. At viewport widths ≥800px, all three columns display side-by-side. Below 800px, columns stack vertically with the game board first, controls/stats second, and AI panel third. The game board itself (250×500px) has a fixed size, but the surrounding layout adapts to available space.
*   **Data Persistence:** Use `localStorage` to persist:
    *   High score across game sessions
    *   AI tuning weights and user preferences
    *   Ensure data survives browser restarts and page reloads
    *   **Error Handling:** If localStorage is unavailable or corrupted:
        *   High score: Default to 0, continue operation normally
        *   AI weights: Reset to default values (Lines: +2.0, Holes: -2.0, Bumpiness: -1.0, Height: -0.01, Valley Depth: -1.0)
        *   **Error Display Method**: Log errors to browser console for debugging. No user-visible alerts. Implementation silently falls back to default values.

## Important Notes:

*   **Speed Control:** Properly implement the accelerated initial descent feature so pieces fall quickly when spawning off-screen (100ms intervals for first 3 drops) before transitioning to normal level-based speed (1000ms - (level-1)*25ms, minimum 50ms).
*   **Statistics Accuracy:** Ensure all game statistics (lines cleared, current speed, AI scores) update correctly and display accurate real-time values.
*   **AI Integration:** Seamlessly integrate manual and automated gameplay modes, with proper state management when switching between modes.
*   **UI Layout:** Organize the interface into logical sections: game board (center), controls and statistics (left), AI tuning panel (right) for optimal user experience and visibility.
*   **Game Over Detection with Off-Screen Spawning:** Because pieces spawn fully off-screen at negative Y coordinates, the standard collision detection alone is insufficient for game over detection. The collision check only validates cells within visible board boundaries, meaning pieces at fully negative Y positions will never register a collision. To properly detect game over, check during piece locking whether any block has a negative Y position. If any block is locked above the visible area, trigger game over immediately.

## Detailed Technical Specifications:

### **Exact UI Component Specifications:**

*   **Game Board:** 10×20 grid of 25px blocks with a 3px solid cyan (#00FFFF) border and 4px internal padding. The padding creates a visible gap between the border and the grid cells. Overlay screens (title, pause, game over) use `width: 100%; height: 100%` to match the board wrapper dimensions automatically.
*   **Preview Grids:** 80×80 pixels (4×4 grid, 20px per block) for Next and Hold pieces with styling:
    *   Preview cells: `border: 0.5px solid rgba(255, 255, 255, 0.05)` (subtle grid)
    *   Preview blocks: `border: 1px solid rgba(255, 255, 255, 0.3)` and `box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.5)` (matches main game blocks)
    *   **Piece Centering:** Pieces are positioned using their natural shape coordinates (starting at 0,0 in the 4×4 grid). No special centering algorithm needed.
*   **Control Panel Width:** 220 pixels maximum (min-width: 180px, max-width: 220px)
*   **AI Tuning Panel Width:** 240 pixels maximum (min-width: 180px, max-width: 240px)
*   **Responsive Breakpoint:** 800px - above this width, columns display side-by-side with 40px gaps; below, they stack vertically with 20px gaps

### **AI Algorithm Implementation Details:**

*   **Evaluation Function Formula:**
    ```
    score = (linesCleared × lineWeight × 100) +
            (holes × holeWeight × 5) +
            (bumpiness × bumpinessWeight) +
            (aggregateHeight × heightWeight) +
            (valleyDepth × valleyDepthWeight) +
            (gameOverPenalty)
    ```

*   **Default AI Weights and Slider Configurations:**
    *   Lines Cleared: +2.0 (range: 0 to 2, step 0.1)
    *   Holes Penalty: -2.0 (range: -2 to 0, step 0.1)
    *   Bumpiness Penalty: -1.0 (range: -1 to 0, step 0.1)
    *   Aggregate Height: -0.01 (range: -0.1 to 0, step 0.01)
    *   Valley Depth: -1.0 (range: -1 to 0, step 0.1)
    *   **Slider Range Design Intent:** Ranges are intentionally restricted to "sensible" values - Lines Cleared is non-negative (rewarding line clears), while all penalties are non-positive (penalizing bad board states). The Reset button restores defaults if experimentation leads to poor play.

### **Performance and Timing Specifications:**

*   **Maximum Autoplay Delay Range:** 20ms to 1000ms (as a dropdown)
*   **Line Clear Animation Duration:** No animation - immediate line removal and visual update
*   **Ghost Piece Styling:** 30% opacity, dashed white border (`border: 1px dashed white`), transparent background, no box shadow. Ghost piece displays as a transparent outline with only the dashed white border visible.
*   **Piece Lock Delay:** Implement lock delay mechanic: When a piece collides and cannot move down, a lock delay timer starts equal to the current drop speed. During this delay, the player can continue to move or rotate the piece, but these actions do **not** reset the timer. Once the timer expires, the piece locks. **Exceptions:** Hard drops and soft drops bypass the lock delay entirely and lock the piece immediately.
    
    **Lock Delay Timer Management:**
    
    **Global Timer Variables:**
    Two separate timers are maintained:
    - Main gravity timer (repeating interval)
    - Lock delay timer (single timeout)
    
    **Drop Interval Timer (Main Gravity):**
    - Runs continuously during active gameplay
    - Calls automatic downward movement at regular intervals
    - NOT paused when lock delay activates
    - Cleared and restarted when level changes, game pauses, game over, or transitioning from fast to normal speed
    
    **Lock Delay Timer:**
    - Created when automatic downward movement detects collision
    - Duration equals current drop speed
    - Only ONE lock delay timer should exist at a time
    - **Not** cleared or restarted when player moves or rotates piece; runs to completion
    - Never created for hard drops or soft drops (they lock immediately)
    
    **Timer Interaction Scenarios:**
    
    1. **Normal Drop with Lock Delay:**
       - Automatic downward movement detects collision
       - Lock delay timer starts
       - Main gravity timer continues running
       - If next gravity tick occurs while lock delay active and piece still can't move, no new timer created

    2. **Player Movement During Lock Delay:**
       - Player moves or rotates piece successfully
       - Lock delay timer continues running unchanged (it is **not** cleared or restarted)
       - Piece will still lock when the original timer expires
    
    3. **Hard Drop (No Lock Delay):**
       - Move piece to lowest possible position instantly
       - Award drop points
       - Clear any existing lock delay timer
       - Immediately lock piece (bypass lock delay)
    
    4. **Soft Drop with Collision (No Lock Delay):**
       - Attempt to move down one cell
       - If can move: move piece, award +1 point, clear lock delay if active
       - If collision: award +1 point, clear lock delay if active, immediately lock piece
    
    **Key Implementation Rules:**
    - Only one lock delay timer at a time - always clear before creating new one
    - Main gravity timer never stops during lock delay
    - Successful movement/rotation does **not** clear or restart the lock delay timer
    - Hard/soft drop bypass lock delay entirely (clear timer and lock immediately)
    - Clear lock delay timer when piece locks to prevent double-locking

### **Data Structure Specifications:**

*   **Board Array:** 2D array where 0 = empty, 1-7 = tetromino ID + 1
*   **Tetromino Shapes:** Seven standard pieces defined as 2D arrays:
    - I-piece: 4×4 array with horizontal line
    - O-piece: 2×2 array (square)
    - T-piece: 3×3 array (T-shape)
    - S-piece: 3×3 array (S-shape)
    - Z-piece: 3×3 array (Z-shape)
    - J-piece: 3×3 array (J-shape)
    - L-piece: 3×3 array (L-shape)
*   **Spawn Position:** Horizontally centered using formula: floor(board_width / 2) - ceiling(piece_width / 2), Vertically: negative Y equal to piece height (fully off-screen)
    
    **Spawn Position Calculation:**
    
    For standard 7 tetrominoes on 10-wide board:
    - I-piece (width 4): X = 3
    - O-piece (width 2): X = 4
    - T, S, Z, J, L (width 3): X = 3
    
    All standard pieces produce valid positions. This formula is designed specifically for standard Tetris implementation with 10-wide board and standard pieces (max width 4).
    
*   **localStorage Keys:**
    *   `"tetrisHighScore"`: JSON object with `score` (integer), `lines` (integer), and `level` (integer) properties. Missing properties default to: level=1, lines=0, score=0.
    *   `"tetrisAIWeights"`: JSON object with 5 weight parameters

### **Error Handling Protocols:**

*   **localStorage Unavailable:**
    *   Silently handle failures via try/catch blocks
    *   Continue with default values (high score = 0, default AI weights)
    *   No user notifications

*   **Corrupted Save Data:**
    *   Catch JSON parse errors in load functions
    *   Merge any valid properties from corrupted data with defaults using spread operator
    *   Reset invalid properties to defaults silently

*   **Invalid AI Weight Values:**
    *   Clamp values to specified ranges
    *   Reset to defaults if NaN or undefined
    *   Update UI sliders to reflect corrected values

### **Accessibility and Usability:**

*   **Keyboard Focus Management:** All interactive elements must be keyboard accessible
*   **Visual Feedback:** Buttons must have hover and active states:
    *   Button hover: Background color transition from #0f0 to #0c0 with 200ms transition
    *   Reset button hover: Background color transition from #666 to #888 with 200ms transition
*   **Text Contrast:** Minimum 4.5:1 contrast ratio for all text elements
*   **Clear Labels:** All sliders and controls must have descriptive labels
*   **Status Messages:** Game state changes are communicated via visual text overlay screens (title, pause, game over)

### **Browser Compatibility Requirements:**

*   **Minimum Browser Support:**
    *   Chrome 70+, Firefox 65+, Safari 12+, Edge 79+
    *   Requires ES6+ support (const, let, arrow functions, template literals)
    *   localStorage API required
    *   CSS Grid and Flexbox support required

### **Testing and Validation Criteria:**

*   **Functional Tests:**
    *   All 7 tetromino shapes spawn and rotate correctly
    *   Line clearing awards correct points (100, 300, 500, 800 × level)
    *   Level progression occurs based on lines cleared per level
    *   AI evaluation produces scores within expected ranges
    *   localStorage persists data across page reloads

*   **Performance Benchmarks:**
    *   Page load time under 2 seconds on standard connection
    *   AI move calculation completes within 100ms
    *   Memory usage remains stable during extended play sessions

*   **User Experience Validation:**
    *   Controls respond within 50ms of key press
    *   Visual feedback appears for all user interactions
    *   Game remains playable at minimum supported resolution (1024×768)
    *   All text remains readable at different zoom levels (100%-150%)

### **Detailed Input Handling Specifications:**

*   **Key Repeat Behavior:** The game relies on the browser's native key repeat behavior. When a key is held down, the browser fires repeated keydown events which the game processes normally.
*   **Simultaneous Key Handling:** Later keydown events override earlier ones - no special handling for simultaneous presses
*   **Input During States:**
    *   Game Over: Only 'R' (restart), 'A' (autoplay toggle), 'P'/'Escape' (pause toggle) work
    *   Paused: Only 'P'/'Escape' (resume), 'R' (restart), 'A' (autoplay toggle), 'G' (ghost toggle) work
    *   Autoplay Active: Only 'A' (autoplay toggle), 'R' (restart), 'P'/'Escape' (pause), 'G' (ghost toggle) work
    *   Normal Play: All movement keys work plus system keys
*   **Input Blocking:** Manual input is completely blocked during autoplay mode via early return check

### **AI Algorithm Clarification:**

*   **Valley Depth Algorithm:** For each column, finds maximum height of left and right adjacent columns. If current column is shorter than this maximum, adds the height difference to penalty total. No minimum threshold - any height difference contributes to penalty.
*   **Move Evaluation Sequence:** For each possible piece placement (4 rotations × board width × hold options), simulate placement, clear lines, evaluate resulting board state, keep track of best scoring moves

### **Error Recovery Protocols:**

*   **Mid-Game localStorage Failure:** Game continues with current values, new changes not persisted, console warning logged
*   **Partial Data Corruption:** When corrupted data detected, merge any valid properties with defaults using spread operator
*   **Game State Inconsistency:** If collision detected at spawn, immediately trigger game over - no recovery attempted

### **Precise Timing Specifications:**

*   **Fast Drop Duration:** Exactly first 3 drops at 100ms intervals
*   **Level Speed Transition:** Occurs immediately on 3rd drop completion
*   **Autoplay Action Delays:**
    *   Best move evaluation: Instant (no delay)
    *   Between each rotation step: configured delay
    *   Between each horizontal movement step: configured delay
    *   Before hard drop: configured delay
    *   After hard drop: No additional delay - next piece spawn triggers immediate evaluation
*   **Statistics Update Timing:**
    *   Lines cleared: Updated immediately after line clears
    *   Speed display: Updated when drop interval changes
    *   AI score: Updated when best move calculated, "N/A" during manual play

# Development Roadmap and Dependencies

This section provides clear guidance on implementation order, showing which features must be built first and what depends on them. The roadmap follows the 4-phase structure outlined in the Implementation Priority Guide.

## Phase 1: CORE - Foundation and Basic Gameplay

These components form the core engine and must be implemented before anything else:

1. **HTML Structure**
   - Three-column responsive layout
   - Game board container
   - Preview grids (Next/Hold)
   - Control panels and statistics areas
   - **Dependencies**: None - start here

2. **CSS Styling**
   - Retro Tetris aesthetic (colors, fonts, glow effects)
   - Responsive layout (800px breakpoint)
   - Grid system for game board
   - **Dependencies**: HTML structure
   - **Note**: Can be refined throughout development

3. **Board Matrix Data Structure**
   - 10×20 2D array initialization
   - Empty cell representation (0 values)
   - **Dependencies**: None - parallel with HTML
   - **Critical**: Required before any piece operations

4. **Tetromino Definitions**
   - Seven piece shapes as 2D arrays
   - Color mappings for each piece type
   - **Dependencies**: None - parallel with board
   - **Critical**: Required before piece spawning

5. **Collision Detection System**
   - Boundary checking (board edges)
   - Block overlap detection
   - **Dependencies**: Board matrix, tetromino definitions
   - **Critical**: Required for all movement operations
   - **Note**: Validates cells only within visible board (Y >= 0)

6. **Board Rendering System**
   - Clear and redraw entire board state
   - Render locked blocks from board array
   - Render current falling piece
   - **Dependencies**: Board matrix, DOM elements
   - **Critical**: Required to see anything on screen

7. **Piece Spawning**
   - Random piece generation
   - Next piece queue management
   - Spawn position calculation (centered, off-screen)
   - **Dependencies**: Board matrix, tetromino definitions, collision detection
   - **Note**: Spawn collision check exists but rarely triggers

8. **Automatic Descent (Gravity)**
   - Drop interval timer (setInterval)
   - Automatic downward movement
   - Speed calculation based on level
   - **Dependencies**: Piece spawning, collision detection, board rendering
   - **Critical**: This is the main game loop driver

9. **Manual Movement (Left/Right)**
   - Horizontal movement on arrow key press
   - Boundary and collision validation
   - **Dependencies**: Collision detection, board rendering
   - **Can implement**: Immediately after automatic descent

10. **Basic Rotation (Clockwise)**
    - 90-degree matrix transformation
    - Simple collision check at current position
    - **Dependencies**: Collision detection, board rendering
    - **Note**: Can implement without wall kicks initially

11. **Piece Locking**
    - Transfer piece blocks to board array
    - Game over detection (blocks above board)
    - Trigger next piece spawn
    - **Dependencies**: Board matrix, piece spawning
    - **Critical**: Completes the piece lifecycle

12. **Line Clearing**
    - Detect completed rows
    - Remove lines and shift blocks down
    - **Dependencies**: Board matrix, board rendering
    - **Critical**: Required for game to be winnable

13. **Basic Scoring**
    - Points for line clears (100/300/500/800 × level)
    - Score display updates
    - **Dependencies**: Line clearing, DOM elements
    - **Can implement**: With line clearing

**Milestone**: At this point, you have a basic playable Tetris game!

## Phase 2: ENHANCED - Quality of Life Improvements

These features improve gameplay beyond the basic function:

14. **Wall Kick Rotation**
    - Test multiple horizontal offsets [0, -1, +1, -2, +2]
    - Bounds validation before collision check
    - **Dependencies**: Basic rotation already working
    - **Enhancement**: Makes rotation feel better

15. **Lock Delay Timer**
    - Delay timer when piece lands
    - Timer is **not** reset by movement/rotation; runs to completion
    - **Dependencies**: Piece locking, automatic descent
    - **Enhancement**: Gives players adjustment time

16. **Soft Drop**
    - Manual downward movement on Down arrow
    - +1 point per cell dropped
    - Immediate lock on collision
    - **Dependencies**: Basic movement, scoring
    - **Enhancement**: Player control improvement

17. **Hard Drop**
    - Instant drop to lowest position
    - +2 points per cell dropped
    - Immediate lock (bypass lock delay)
    - **Dependencies**: Collision detection, piece locking, scoring
    - **Enhancement**: Player control improvement

18. **Accelerated Initial Descent**
    - First 3 drops at 100ms intervals
    - Drop counter tracking
    - Transition to normal speed
    - **Dependencies**: Automatic descent timer
    - **Enhancement**: Reduces perceived spawn delay

## Phase 3: ADVANCED - UI Features and Persistence

These enhance the interface and user experience:

19. **Next Piece Preview**
    - Display next piece in preview grid
    - Update on each spawn
    - **Dependencies**: Piece spawning, DOM elements
    - **Enhancement**: Strategy planning

20. **Ghost Piece**
    - Calculate landing position
    - Render transparent outline
    - Toggle visibility with 'G' key
    - **Dependencies**: Collision detection, board rendering
    - **Enhancement**: Visual aid, can be built anytime
    - **Independent**: Does not affect game logic

21. **Hold Piece Functionality**
    - Swap current piece with held piece
    - One-time use per piece lock
    - Reset piece orientation
    - **Dependencies**: Piece spawning, board rendering
    - **Enhancement**: Strategic gameplay option

22. **Level Progression**
    - Track lines cleared since level up
    - Variable threshold based on current level (10-20 lines)
    - Increase drop speed on level up
    - **Dependencies**: Line clearing, automatic descent timer
    - **Enhancement**: Difficulty progression

23. **Title Screen**
    - Overlay with game title and start button
    - Hide on game start
    - **Dependencies**: DOM elements, game initialization
    - **Enhancement**: Professional presentation

24. **Pause/Resume**
    - Pause all timers on 'P'/Escape
    - Display pause overlay
    - Resume gameplay
    - **Dependencies**: All timer systems
    - **Enhancement**: User control

25. **Game Over Screen**
    - Display final statistics
    - Restart button
    - **Dependencies**: Game over detection, statistics
    - **Enhancement**: User feedback

26. **Statistics Display**
    - Real-time lines cleared counter
    - Current drop speed display
    - Level display
    - **Dependencies**: Line clearing, level progression, DOM elements
    - **Enhancement**: Player feedback

27. **High Score System**
    - Track highest score across sessions
    - Save score, lines, level together
    - Update check on every score change
    - **Dependencies**: Scoring system, localStorage
    - **Enhancement**: Long-term engagement

28. **localStorage Persistence**
    - Save/load high score data
    - Error handling for unavailable/corrupted data
    - **Dependencies**: High score system
    - **Enhancement**: Cross-session persistence

29. **Input Handling Refinement**
    - Case-insensitive key detection
    - State-based input blocking
    - Restart confirmation dialog
    - **Dependencies**: All game states
    - **Polish**: User experience improvements

## Phase 4: AI AUTOPLAY

These are complex features that should be implemented after the earlier phases:

30. **AI Autoplay Foundation**
    - Toggle autoplay mode
    - Block manual input during autoplay
    - Visual indicator (AI ON/OFF badge)
    - **Dependencies**: Complete core game
    - **Complex**: Can be built incrementally

31. **AI Board Evaluation**
    - Calculate holes, bumpiness, height, valley depth
    - Weighted scoring formula
    - **Dependencies**: Board state analysis
    - **Complex**: Pure logic, no dependencies on game loop

32. **AI Move Calculation**
    - Evaluate all rotations and positions
    - Consider hold piece option
    - Return best scoring move
    - **Dependencies**: AI board evaluation, collision detection
    - **Complex**: Computationally intensive

33. **AI Move Execution**
    - Step-by-step move execution with delays
    - Rotation, movement, and drop actions
    - Configurable delay timing
    - **Dependencies**: AI move calculation, game controls
    - **Complex**: Timer coordination

34. **AI Tuning Panel**
    - Sliders for weight adjustment
    - Persistence via localStorage
    - Reset to defaults button
    - **Dependencies**: AI evaluation system, DOM elements
    - **Enhancement**: AI customization
