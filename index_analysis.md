# Detailed Code Analysis: index.html

This document provides a comprehensive line-by-line analysis of the Lines 98 game implementation in index.html.

---

## Document Information

- **File Analyzed**: index.html
- **Total Lines**: 407
- **Game Type**: Lines 98 Puzzle Game (Mobile-Optimized)
- **Framework**: Phaser 3.60.0

---

## Section 1: HTML Structure (Lines 1-22)

### Line 1: `<!DOCTYPE html>`
Declares the document type as HTML5. This ensures the browser uses standards mode for rendering.

### Line 2: `<html lang="en">`
Opens the HTML document with English language specification, important for accessibility tools and search engines.

### Lines 3-5: Meta Tags
```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```
- `charset="UTF-8"` ensures proper character encoding
- `viewport` tag configures responsive display:
  - `width=device-width`: Matches device screen width
  - `initial-scale=1.0`: No initial zoom
  - `maximum-scale=1.0`: Prevents user zoom (prevents pinch-zoom)
  - `user-scalable=no`: Disables user scaling
- **Issue**: Missing `color-scheme` meta tag for better mobile dark mode integration

### Line 6: Title
```html
<title>Lines 98 - Mobile</title>
```
Sets the browser tab title. Descriptive but could include version info.

### Line 7: Phaser Library
```html
<script src="https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser.min.js"></script>
```
Loads Phaser 3.60.0 from CDN. Using a specific version ensures consistent behavior. Consider pinning to exact version or using local copy for offline play.

### Lines 8-21: CSS Styles
```css
body {
    margin: 0;
    padding: 0;
    background-color: #1a1a1a;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}
canvas {
    box-shadow: 0 0 20px rgba(0,0,0,0.5);
}
```
- Removes default body margins/padding
- Uses dark background (`#1a1a1a`)
- Centers game canvas using flexbox
- `height: 100vh` ensures full viewport height
- Adds subtle shadow to canvas for visual depth
- **Good**: Clean, minimal styling focused on centering the game

---

## Section 2: Game Configuration (Lines 26-38)

### Lines 26-38: CONFIG Object
```javascript
const CONFIG = {
    GRID_SIZE: 9,
    TILE_SIZE: 76,
    COLORS: [
        0xff0000, // Red
        0x00ff00, // Green
        0x0088ff, // Blue
        0xffff00, // Yellow
        0x00ffff, // Cyan
        0xff00ff, // Magenta
        0xff8800  // Orange
    ]
};
```
- `GRID_SIZE: 9`: Standard 9x9 board size for Lines 98
- `TILE_SIZE: 76`: Each cell is 76 pixels (results in 684px board)
- `COLORS`: Array of 7 colors in hex format:
  - Red (0xff0000)
  - Green (0x00ff00)
  - Blue (0x0088ff)
  - Yellow (0xffff00)
  - Cyan (0x00ffff)
  - Magenta (0xff00ff)
  - Orange (0xff8800)
- **Observation**: Colors are vibrant and distinguishable. Using hex without alpha (fully opaque).

---

## Section 3: Lines98 Class - Constructor (Lines 40-43)

### Lines 40-43: Class Definition
```javascript
class Lines98 extends Phaser.Scene {
    constructor() {
        super('Lines98');
    }
```
- Extends `Phaser.Scene` to create a game scene
- Calls parent constructor with scene name 'Lines98'
- **Good**: Clean class-based architecture

---

## Section 4: create() Method (Lines 45-72)

### Lines 45-72: Main Initialization
```javascript
create() {
    // Game State
    this.board = Array(CONFIG.GRID_SIZE).fill(null).map(() => Array(CONFIG.GRID_SIZE).fill(null));
    this.score = 0;
    this.nextColors = [];
    this.selectedBall = null;
    this.isAnimating = false;
    
    // Layout calculations
    this.boardWidth = CONFIG.GRID_SIZE * CONFIG.TILE_SIZE;
    this.offsetX = (this.scale.width - this.boardWidth) / 2;
    this.offsetY = 300; // Leave space at the top for UI

    // Generate Textures programmatically
    this.generateTextures();

    // UI Setup
    this.createUI();
    this.drawGrid();

    // Input
    this.input.on('pointerdown', this.handleInput, this);

    // Start Game
    this.generateNextColors();
    this.spawnBalls(5); // Spawn initial 5 balls
    this.generateNextColors(); // Queue the next 3
}
```

### Line 47: Board Initialization
```javascript
this.board = Array(CONFIG.GRID_SIZE).fill(null).map(() => Array(CONFIG.GRID_SIZE).fill(null));
```
- Creates 2D array (9x9) representing the game board
- Each cell initialized to `null` (empty)
- **Good**: Proper 2D array creation with proper scoping

### Lines 48-51: Game State Variables
- `this.score`: Tracks player score (starts at 0)
- `this.nextColors`: Array to store upcoming ball colors for preview
- `this.selectedBall`: Tracks currently selected ball (null if none selected)
- `this.isAnimating`: Flag to prevent input during animations

### Lines 54-56: Layout Calculations
- `this.boardWidth`: Calculated as GRID_SIZE × TILE_SIZE (9 × 76 = 684px)
- `this.offsetX`: Centers board horizontally on screen
- `this.offsetY`: Fixed 300px offset from top for UI elements
- **Observation**: Hard-coded offsetY may cause issues on different screen sizes

### Line 59: Texture Generation
```javascript
this.generateTextures();
```
Calls method to programmatically create ball textures.

### Lines 62-63: UI and Grid Setup
```javascript
this.createUI();
this.drawGrid();
```
Creates game UI elements and draws the grid background.

### Line 66: Input Handler
```javascript
this.input.on('pointerdown', this.handleInput, this);
```
Registers pointer/touch input listener. Uses `bind` pattern via third parameter.

### Lines 69-71: Game Start Sequence
1. `generateNextColors()`: Generate initial color queue
2. `spawnBalls(5)`: Spawn initial 5 balls on board
3. `generateNextColors()`: Generate the next 3 colors for preview
- **Flow**: Initial spawn → queue next colors → ready for player input

---

## Section 5: generateTextures() Method (Lines 74-86)

### Lines 74-86: Dynamic Texture Generation
```javascript
generateTextures() {
    const g = this.add.graphics();
    
    // Ball Texture
    g.fillStyle(0xffffff, 1);
    g.fillCircle(30, 30, 28);
    // Add a little highlight for 3D effect
    g.fillStyle(0xffffff, 0.5);
    g.fillCircle(20, 20, 8);
    g.generateTexture('ball', 60, 60);
    
    g.clear();
}
```

### Line 75: Graphics Object
```javascript
const g = this.add.graphics();
```
Creates a new Graphics object for drawing.

### Lines 78-79: Main Ball Circle
- Fills white circle at position (30, 30) with radius 28
- Creates the main ball body

### Lines 81-82: Highlight Effect
- Adds smaller white circle (20, 20) with 0.5 alpha
- Creates pseudo-3D highlight effect
- **Observation**: Works visually but highlight is white, not colored

### Line 83: Texture Generation
```javascript
g.generateTexture('ball', 60, 60);
```
Generates a 60x60 pixel texture named 'ball' from the graphics.

### Line 85: Clear Graphics
```javascript
g.clear();
```
Clears the graphics object.
- **Issue**: Graphics object `g` is not destroyed, causing memory leak

---

## Section 6: createUI() Method (Lines 88-105)

### Lines 88-105: UI Elements Creation
```javascript
createUI() {
    this.add.text(this.scale.width / 2, 80, 'LINES 98', { fontSize: '48px', fontStyle: 'bold', color: '#ffffff' }).setOrigin(0.5);
    
    this.scoreText = this.add.text(this.scale.width / 2, 140, 'Score: 0', { fontSize: '32px', color: '#ffff00' }).setOrigin(0.5);
    
    this.add.text(this.scale.width / 2, 200, 'Next:', { fontSize: '24px', color: '#aaaaaa' }).setOrigin(0.5);
    
    this.nextPreviewSprites = [];
    for (let i = 0; i < 3; i++) {
        let sprite = this.add.sprite(this.scale.width / 2 - 60 + (i * 60), 250, 'ball').setScale(0.5);
        this.nextPreviewSprites.push(sprite);
    }

    let restartBtn = this.add.text(this.scale.width / 2, this.scale.height - 80, 'RESTART GAME', { 
        fontSize: '28px', backgroundColor: '#333333', padding: {x: 20, y: 10}, color: '#ffffff' 
    }).setOrigin(0.5).setInteractive();
    restartBtn.on('pointerdown', () => this.scene.restart());
}
```

### Line 89: Title Text
- Displays "LINES 98" at y=80
- White color, bold, 48px font
- Centered horizontally

### Line 91: Score Display
- Yellow text showing "Score: 0"
- 32px font size
- Stored in `this.scoreText` for updates

### Line 93: "Next:" Label
- Gray text label at y=200
- Indicates upcoming balls preview

### Lines 96-99: Next Balls Preview
- Creates 3 sprite objects at y=250
- Positioned with 60px spacing, centered
- Scaled to 50% (0.5) of original size
- Stored in array for color updates

### Lines 101-104: Restart Button
- Text button at bottom of screen
- Dark gray background with padding
- Interactive (clickable)
- Restarts scene on click
- **Good**: Clear restart mechanism

---

## Section 7: drawGrid() Method (Lines 107-124)

### Lines 107-124: Grid Drawing
```javascript
drawGrid() {
    const g = this.add.graphics();
    g.lineStyle(2, 0x444444);
    g.fillStyle(0x222222);

    g.fillRect(this.offsetX, this.offsetY, this.boardWidth, this.boardWidth);
    g.strokeRect(this.offsetX, this.offsetY, this.boardWidth, this.boardWidth);

    for (let i = 1; i < CONFIG.GRID_SIZE; i++) {
        // Vertical lines
        g.moveTo(this.offsetX + i * CONFIG.TILE_SIZE, this.offsetY);
        g.lineTo(this.offsetX + i * CONFIG.TILE_SIZE, this.offsetY + this.boardWidth);
        // Horizontal lines
        g.moveTo(this.offsetX, this.offsetY + i * CONFIG.TILE_SIZE);
        g.lineTo(this.offsetX + this.boardWidth, this.offsetY + i * CONFIG.TILE_SIZE);
    }
    g.strokePath();
}
```

### Line 108: New Graphics Object
Creates separate graphics object for grid (not reused).

### Lines 109-110: Style Definitions
- Line style: 2px gray (`0x444444`)
- Fill style: Dark gray (`0x222222`)

### Lines 112-113: Board Background
- Fills and strokes the main board rectangle
- Uses calculated offset and dimensions

### Lines 115-122: Grid Lines Loop
- Iterates from 1 to GRID_SIZE-1 (1 to 8)
- Draws vertical and horizontal grid lines
- **Good**: Clean loop for grid line generation

### Line 123: Stroke Path
Renders all queued line segments.

---

## Section 8: getEmptyCells() Method (Lines 126-134)

### Lines 126-134: Find Empty Cells
```javascript
getEmptyCells() {
    let empty = [];
    for (let r = 0; r < CONFIG.GRID_SIZE; r++) {
        for (let c = 0; c < CONFIG.GRID_SIZE; c++) {
            if (!this.board[r][c]) empty.push({ r, c });
        }
    }
    return empty;
}
```

### Lines 127-133: Empty Cell Detection
- Iterates through entire 9x9 board
- Checks if each cell is falsy (null/undefined)
- Collects coordinates of empty cells
- Returns array of {r, c} objects
- **Good**: Simple and efficient implementation

---

## Section 9: generateNextColors() Method (Lines 136-144)

### Lines 136-144: Color Queue Generation
```javascript
generateNextColors() {
    this.nextColors = [];
    for (let i = 0; i < 3; i++) {
        let color = Phaser.Utils.Array.GetRandom(CONFIG.COLORS);
        this.nextColors.push(color);
        this.nextPreviewSprites[i].setTint(color);
        this.nextPreviewSprites[i].setVisible(true);
    }
}
```

### Line 137: Reset Array
Clears the nextColors array.

### Lines 138-143: Generate 3 Random Colors
- Uses Phaser's random utility to pick from CONFIG.COLORS
- Updates preview sprites with appropriate tints
- Ensures sprites are visible
- **Observation**: Colors are truly random with no weighting

---

## Section 10: spawnBalls() Method (Lines 146-181)

### Lines 146-181: Ball Spawning Logic
```javascript
spawnBalls(count) {
    let emptyCells = this.getEmptyCells();
    if (emptyCells.length === 0) return this.gameOver();

    let spawnedCount = Math.min(count, emptyCells.length);
    
    for (let i = 0; i < spawnedCount; i++) {
        let cellIndex = Phaser.Math.Between(0, emptyCells.length - 1);
        let cell = emptyCells.splice(cellIndex, 1)[0];
        
        let color = count === 5 ? Phaser.Utils.Array.GetRandom(CONFIG.COLORS) : this.nextColors[i];
        
        let x = this.offsetX + cell.c * CONFIG.TILE_SIZE + CONFIG.TILE_SIZE / 2;
        let y = this.offsetY + cell.r * CONFIG.TILE_SIZE + CONFIG.TILE_SIZE / 2;
        
        let sprite = this.add.sprite(x, y, 'ball').setTint(color).setScale(0);
        
        this.board[cell.r][cell.c] = { color: color, sprite: sprite };
        
        this.tweens.add({
            targets: sprite,
            scale: 1,
            duration: 300,
            ease: 'Back.out'
        });
    }

    if (this.getEmptyCells().length === 0) {
        this.time.delayedCall(400, () => {
            // Allows surviving if the last spawn clears a line!
            if (this.getEmptyCells().length === 0) {
                this.gameOver();
            }
        }
    }
}
```

### Line 148: Check for Empty Cells
- If no empty cells, triggers game over
- **Good**: Proper game over trigger

### Line 150: Determine Spawn Count
```javascript
let spawnedCount = Math.min(count, emptyCells.length);
```
Prevents spawning more balls than available cells.

### Lines 152-154: Random Cell Selection
- Uses `Phaser.Math.Between` for random index
- Removes selected cell from array using splice (prevents duplicates)

### Lines 156: Color Assignment Logic
```javascript
let color = count === 5 ? Phaser.Utils.Array.GetRandom(CONFIG.COLORS) : this.nextColors[i];
```
- If spawning initial 5 balls: random color
- Otherwise: use pre-generated nextColors array

### Lines 158-159: Position Calculation
- Calculates x, y center of cell based on:
  - offsetX/offsetY + column/row × TILE_SIZE + TILE_SIZE/2 (center)
- **Good**: Proper center-point calculation

### Line 161: Sprite Creation
- Creates sprite at calculated position
- Applies color tint
- Starts at scale 0 (for animation)

### Line 163: Board Update
- Stores ball data in board array
- Contains color and sprite reference

### Lines 165-170: Spawn Animation
- Tweens scale from 0 to 1
- Uses 'Back.out' easing for "pop" effect
- 300ms duration
- **Good**: Satisfying spawn animation

### Lines 173-180: Game Over Check
- Checks if board is full after spawning
- Uses delayed call to allow line clears to complete
- Re-checks after delay to allow survival via line clears
- **Good**: Allows "second chance" after spawning

---

## Section 11: handleInput() Method (Lines 183-222)

### Lines 183-222: Input Handling
```javascript
handleInput(pointer) {
    if (this.isAnimating) return;

    let col = Math.floor((pointer.x - this.offsetX) / CONFIG.TILE_SIZE);
    let row = Math.floor((pointer.y - this.offsetY) / CONFIG.TILE_SIZE);

    // Check out of bounds
    if (row < 0 || row >= CONFIG.GRID_SIZE || col < 0 || col >= CONFIG.GRID_SIZE) return;

    let cell = this.board[row][col];

    if (cell) {
        // Select ball
        if (this.selectedBall) {
            this.selectedBall.sprite.clearTint();
            this.selectedBall.sprite.setTint(this.selectedBall.color); // reset color
            this.tweens.killTweensOf(this.selectedBall.sprite);
            this.selectedBall.sprite.setScale(1);
        }
        this.selectedBall = { row, col, sprite: cell.sprite, color: cell.color };
        
        // Selection Animation (Pulsing)
        this.tweens.add({
            targets: cell.sprite,
            scale: 1.15,
            duration: 300,
            yoyo: true,
            repeat: -1
        });
    } else if (this.selectedBall) {
        // Attempt to move
        let path = this.findPath(this.selectedBall.row, this.selectedBall.col, row, col);
        if (path) {
            this.moveBall(path, row, col);
        } else {
            // Shake ball if no path
            this.cameras.main.shake(200, 0.01);
        }
    }
}
```

### Line 184: Animation Guard
```javascript
if (this.isAnimating) return;
```
Prevents input during animations.

### Lines 186-187: Coordinate Calculation
- Converts screen coordinates to grid coordinates
- Uses floor division to find column/row

### Line 190: Bounds Checking
- Validates row and column are within grid
- Returns early if click is outside board

### Line 192: Get Cell Data
Retrieves ball data at clicked position (null if empty).

### Line 194: Ball Selection Branch
When clicking on a ball (non-null cell):

### Lines 196-201: Deselect Previous Ball
```javascript
if (this.selectedBall) {
    this.selectedBall.sprite.clearTint();
    this.selectedBall.sprite.setTint(this.selectedBall.color);
    this.tweens.killTweensOf(this.selectedBall.sprite);
    this.selectedBall.sprite.setScale(1);
}
```
- **Issue**: Redundant tint operations (clear then set same color)
- **Bug**: Uses `this.selectedBall.color` which is the NEW selection, not previous

### Line 202: Set New Selection
Stores selected ball data including row, col, sprite reference, and color.

### Lines 205-211: Selection Animation
- Scales ball to 1.15x (pulsing effect)
- 300ms duration
- Yoyos back and forth
- Repeats infinitely (-1)
- **Good**: Visual feedback for selection

### Lines 212-221: Movement Attempt
When clicking empty cell with a ball selected:

### Line 214: Path Finding
Calls findPath with start and target coordinates.

### Lines 216-220: Movement or Shake
- If path exists: move ball along path
- If no path: shake camera as feedback
- **Issue**: No feedback for valid moves that are blocked

---

## Section 12: findPath() Method (Lines 224-252)

### Lines 224-252: BFS Pathfinding
```javascript
// Breadth-First Search for pathfinding
findPath(startR, startC, targetR, targetC) {
    let queue = [{ r: startR, c: startC, path: [] }];
    let visited = Array(CONFIG.GRID_SIZE).fill(null).map(() => Array(CONFIG.GRID_SIZE).fill(false));
    visited[startR][startC] = true;

    let directions = [[-1,0], [1,0], [0,-1], [0,1]];

    while (queue.length > 0) {
        let current = queue.shift();

        if (current.r === targetR && current.c === targetC) {
            return current.path;
        }

        for (let dir of directions) {
            let r = current.r + dir[0];
            let c = current.c + dir[1];

            if (r >= 0 && r < CONFIG.GRID_SIZE && c >= 0 && c < CONFIG.GRID_SIZE) {
                if (!visited[r][c] && this.board[r][c] === null) {
                    visited[r][c] = true;
                    queue.push({ r, c, path: [...current.path, {r, c}] });
                }
            }
        }
    }
    return null; // No path found
}
```

### Lines 226-228: Initialization
- Queue starts with initial position and empty path
- 2D visited array tracks visited cells
- Marks start position as visited

### Line 230: Direction Vectors
- Up (-1, 0), Down (1, 0), Left (0, -1), Right (0, 1)
- **Observation**: No diagonal movement (correct for Lines 98)

### Lines 232-237: BFS Loop
- Processes queue in FIFO order (guarantees shortest path)
- Returns path when target is reached

### Lines 239-248: Neighbor Exploration
- Checks all 4 directions
- Validates bounds (0 to GRID_SIZE-1)
- Checks if cell is empty (not visited AND null)
- Adds valid neighbors to queue with updated path

### Line 251: No Path Found
Returns null if queue exhausted without finding target.

---

## Section 13: moveBall() Method (Lines 254-289)

### Lines 254-289: Ball Movement Animation
```javascript
moveBall(path, targetR, targetC) {
    this.isAnimating = true;
    let ballData = this.board[this.selectedBall.row][this.selectedBall.col];
    this.board[this.selectedBall.row][this.selectedBall.col] = null; // Free old spot
    
    this.tweens.killTweensOf(ballData.sprite);
    ballData.sprite.setScale(1);
    
    this.selectedBall = null;

    // Phaser 3.60 Sequence using tweens.chain()
    let tweensList = path.map(step => ({
        x: this.offsetX + step.c * CONFIG.TILE_SIZE + CONFIG.TILE_SIZE / 2,
        y: this.offsetY + step.r * CONFIG.TILE_SIZE + CONFIG.TILE_SIZE / 2,
        duration: 50 // fast movement per tile
    }));

    this.tweens.chain({
        targets: ballData.sprite,
        tweens: tweensList,
        onComplete: () => {
            this.board[targetR][targetC] = ballData;
            
            let linesCleared = this.checkLines();
            if (!linesCleared) {
                this.spawnBalls(3);
                let newlyCleared = this.checkLines();
                this.generateNextColors();
                
                if (!newlyCleared && this.getEmptyCells().length > 0) {
                    this.isAnimating = false;
                }
            }
        }
    });
}
```

### Line 255: Set Animation Flag
```javascript
this.isAnimating = true;
```
Prevents further input during movement.

### Line 256-257: Remove Ball from Board
- Gets ball data from old position
- Sets old position to null

### Lines 259-260: Stop Selection Animation
- Kills any running tweens on sprite
- Resets scale to 1

### Line 262: Clear Selection
Sets selectedBall to null.

### Lines 265-269: Build Path Tweens
- Maps path steps to tween configurations
- Each step: 50ms duration
- **Good**: Smooth sequential movement

### Lines 271-288: Chain Animation
```javascript
this.tweens.chain({
    targets: ballData.sprite,
    tweens: tweensList,
    onComplete: () => {
        this.board[targetR][targetC] = ballData;
        
        let linesCleared = this.checkLines();
        if (!linesCleared) {
            this.spawnBalls(3);
            let newlyCleared = this.checkLines();
            this.generateNextColors();
            
            if (!newlyCleared && this.getEmptyCells().length > 0) {
                this.isAnimating = false;
            }
        }
    }
});
```

### Line 275: Update Board
Places ball at new position after movement completes.

### Line 277: Check for Lines
Checks if move created any lines of 5+.

### Lines 278-286: Post-Move Logic
- If no lines cleared: spawn 3 new balls
- Check again for lines after spawn
- Generate next colors
- **Critical Bug**: When `linesCleared` is true, `isAnimating` is never set to `false`, causing game to freeze
- **Issue**: Game becomes unresponsive after clearing lines

---

## Section 14: checkLines() Method (Lines 291-334)

### Lines 291-334: Line Detection Algorithm
```javascript
checkLines() {
    let ballsToRemove = new Set();
    let dirs = [
        [{r:0, c:1}, {r:0, c:-1}], // Horizontal
        [{r:1, c:0}, {r:-1, c:0}], // Vertical
        [{r:1, c:1}, {r:-1, c:-1}],// Diagonal 1
        [{r:1, c:-1}, {r:-1, c:1}] // Diagonal 2
    ];

    for (let r = 0; r < CONFIG.GRID_SIZE; r++) {
        for (let c = 0; c < CONFIG.GRID_SIZE; c++) {
            if (!this.board[r][c]) continue;
            let color = this.board[r][c].color;

            for (let dirPair of dirs) {
                let line = [{r, c}];
                
                for (let dir of dirPair) {
                    let stepR = r + dir.r;
                    let stepC = c + dir.c;
                    while (stepR >= 0 && stepR < CONFIG.GRID_SIZE && stepC >= 0 && stepC < CONFIG.TILE_SIZE) {
                        if (this.board[stepR][stepC] && this.board[stepR][stepC].color === color) {
                            line.push({r: stepR, c: stepC});
                            stepR += dir.r;
                            stepC += dir.c;
                        } else {
                            break;
                        }
                    }
                }

                if (line.length >= 5) {
                    line.forEach(pos => ballsToRemove.add(`${pos.r},${pos.c}`));
                }
            }
        }
    }

    if (ballsToRemove.size > 0) {
        this.removeBalls(ballsToRemove);
        return true;
    }
    return false;
}
```

### Lines 293-298: Direction Pairs
Defines 4 direction pairs for line detection:
- Horizontal (left/right)
- Vertical (up/down)
- Diagonal 1 (top-left/bottom-right)
- Diagonal 2 (top-right/bottom-left)

### Lines 300-327: Board Iteration
- Iterates through every cell
- Skips empty cells
- For each ball, checks all 4 direction pairs

### Lines 306-320: Line Building
- Starts with current cell in line
- For each direction in pair, extends as far as possible
- Checks both positive and negative directions
- **Note**: Uses CONFIG.TILE_SIZE in bounds check (should be CONFIG.GRID_SIZE)
- **Bug**: Line 312 has typo - uses CONFIG.TILE_SIZE instead of CONFIG.GRID_SIZE on line 312

### Lines 322-324: Line Validation
- If line has 5+ balls: add to removal set
- Uses Set to handle overlapping lines (balls in multiple lines)

### Lines 329-333: Removal and Return
- If balls to remove: calls removeBalls() and returns true
- Otherwise returns false

---

## Section 15: removeBalls() Method (Lines 336-366)

### Lines 336-366: Ball Removal
```javascript
removeBalls(ballsSet) {
    this.isAnimating = true;
    let count = ballsSet.size;
    
    // Classic scoring: 10 pts for 5 balls. Every extra ball adds 5 more points.
    let addedScore = 10 + (count - 5) * 5; 
    this.score += addedScore;
    this.scoreText.setText(`Score: ${this.score}`);

    ballsSet.forEach(posString => {
        let [r, c] = posString.split(',').map(Number);
        let ball = this.board[r][c];
        this.board[r][c] = null; // clear board immediately

        this.tweens.add({
            targets: ball.sprite,
            scale: 0,
            alpha: 0,
            duration: 300,
            onComplete: () => {
                ball.sprite.destroy();
            }
        });
    });

    this.time.delayedCall(300, () => {
        if (this.getEmptyCells().length > 0) {
            this.isAnimating = false;
        }
    });
}
```

### Line 337: Set Animation Flag
```javascript
this.isAnimating = true;
```
Prevents input during removal animation.

### Lines 340-343: Score Calculation
```javascript
let addedScore = 10 + (count - 5) * 5; 
this.score += addedScore;
this.scoreText.setText(`Score: ${this.score}`);
```
- Base 10 points for 5 balls
- +5 points for each additional ball
- **Observation**: Classic scoring formula

### Lines 345-359: Remove Each Ball
- Iterates through Set of positions
- Splits position string to get row, col
- Clears board position
- Creates scale/alpha tween to 0
- Destroys sprite on animation complete

### Lines 361-365: Animation Completion
- Uses delayed call (300ms) to wait for animations
- Only clears animation flag if space available
- **Good**: Proper async handling

---

## Section 16: gameOver() Method (Lines 368-387)

### Lines 368-387: Game Over Screen
```javascript
gameOver() {
    this.isAnimating = true;
    let overlay = this.add.rectangle(0, 0, this.scale.width, this.scale.height, 0x000000, 0.8).setOrigin(0);
    this.add.text(this.scale.width / 2, this.scale.height / 2 - 50, 'GAME OVER', { fontSize: '64px', fontStyle: 'bold', color: '#ff0000' }).setOrigin(0.5);
    this.add.text(this.scale.width / 2, this.scale.height / 2 + 20, `Final Score: ${this.score}`, { fontSize: '40px', color: '#ffffff' }).setOrigin(0.5);
    
    let restartTxt = this.add.text(this.scale.width / 2, this.scale.height / 2 + 100, 'Tap to Restart', { fontSize: '32px', color: '#00ffff' }).setOrigin(0.5);
    
    this.tweens.add({
        targets: restartTxt,
        alpha: 0.2,
        yoyo: true,
        repeat: -1,
        duration: 500
    });

    this.input.once('pointerdown', () => {
        this.scene.restart();
    });
}
```

### Line 369: Set Animation Flag
```javascript
this.isAnimating = true;
```

### Line 370: Dark Overlay
- Full-screen semi-transparent black rectangle
- 80% opacity (0.8 alpha)
- **Issue**: Overlay not stored, can't be cleaned up on restart

### Line 371: Game Over Title
- Large red 64px bold text
- Centered above score

### Line 372: Final Score Display
- White 40px text
- Shows player's final score

### Line 374: Restart Prompt
- Cyan 32px text
- "Tap to Restart" message

### Lines 376-382: Pulsing Animation
- Animates restart text alpha
- Yoyos between full and 0.2 opacity
- Infinite repeat

### Lines 384-386: Restart Handler
- Uses `once` for single-use listener
- Restarts scene on tap
- **Issue**: Game over overlay elements not destroyed on restart

---

## Section 17: Phaser Configuration (Lines 390-403)

### Lines 390-403: Game Configuration
```javascript
// Phaser Configuration for Mobile Portrait
const config = {
    type: Phaser.AUTO,
    scale: {
        mode: Phaser.Scale.FIT, // Scales the game to fit the screen
        autoCenter: Phaser.Scale.CENTER_BOTH,
        width: 720,
        height: 1280
    },
    backgroundColor: '#1a1a1a',
    scene: Lines98
};

const game = new Phaser.Game(config);
```

### Line 392: Renderer Selection
```javascript
type: Phaser.AUTO,
```
Automatically chooses WebGL or Canvas.

### Lines 393-398: Scale Configuration
- `mode: Phaser.Scale.FIT`: Scales to fit screen
- `autoCenter: Phaser.Scale.CENTER_BOTH`: Centers both axes
- `width: 720`: Base width (portrait)
- `height: 1280`: Base height
- **Good**: Proper mobile portrait configuration

### Line 399: Background Color
Matches body background (`#1a1a1a`).

### Line 400: Scene Registration
Registers the Lines98 scene.

### Line 403: Game Initialization
```javascript
const game = new Phaser.Game(config);
```
Creates and starts the Phaser game instance.

---

## Summary of Issues

### Critical Bugs

1. **Game Freeze After Line Clear** (Lines 277-286)
   - When `linesCleared` is true, `isAnimating` never set to false
   - Game becomes unresponsive

2. **Bounds Check Typo** (Line 312)
   - Uses `CONFIG.TILE_SIZE` instead of `CONFIG.GRID_SIZE`
   - Could cause out-of-bounds errors

### Memory Issues

3. **Unused Graphics Object** (Line 75)
   - Graphics object not destroyed after texture generation

4. **Game Over Overlay Elements** (Lines 370-372)
   - Not stored or cleaned up on restart

### Logic Issues

5. **Selection Reset Bug** (Lines 196-201)
   - Uses new selection color instead of previous

6. **No Touch Feedback** (Lines 217-220)
   - Valid blocked paths have no visual feedback

### Minor Issues

7. Missing `color-scheme` meta tag
8. Hard-coded offsetY may cause layout issues
9. No "no moves available" detection

---

## Positive Aspects

- Clean BFS pathfinding implementation
- Proper use of Phaser tweens and scene management
- Good responsive scaling configuration
- Proper bounds checking on input
- Smooth animations with easing
- Classic scoring system
- Mobile-optimized viewport settings

---

*Analysis generated on 2026-04-06*
