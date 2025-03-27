# Tic-Tac-Toe Game Logic Documentation

This document explains the logic behind a Vue.js-based Tic-Tac-Toe game implementation. The game features player-vs-player or player-vs. computer modes, multiple difficulty levels, and game state management.

## Core Components

### 1. State Management

```javascript
// Reactive state management using Vue 3 Composition API
const squares = ref([...]); // 3x3 game board
const utils = reactive({...}); // Game utilities and state
```

### 2. Game Board Initialization

The game board is represented as a 3x3 matrix:
```javascript
[
  ["", "", ""],
  ["", "", ""],
  ["", "", ""]
]
```
- Empty string `""` represents an unmarked square
- "X" or "O" represent marked squares

### 3. Game Utilities (utils)

Key reactive properties:
- `player`: Current player ("X" or "O")
- `gameEnd`: Computed boolean indicating if game has ended
- `level`: Computer difficulty ("noob", "easy", "medium", "hard")
- `isComputer`: Boolean for player vs computer mode
- `usedBoxes`: Computed count of marked squares
- `leftBoxes`: Computed count of empty squares
- `playerXPoints/playerOPoints`: Score tracking
- `round`: Current round number
- `winBoxes`: Matrix tracking winning squares
- `conditionThatMatched`: Stores the winning condition indices

### 4. Win Conditions

```javascript
const winConditions = [
  // Rows
  [0, 1, 2], [3, 4, 5], [6, 7, 8],
  // Columns
  [0, 3, 6], [1, 4, 7], [2, 5, 8],
  // Diagonals
  [0, 4, 8], [2, 4, 6]
];
```
These represent all possible winning combinations using flat array indices (0-8).
Here are visual representations of all 8 possible winning conditions in a 3×3 Tic-Tac-Toe game, formatted as ASCII diagrams with explanations:

### 1. Horizontal Wins

```
Top Row Win:
X | X | X
---------
  |   |  
---------
  |   |  

Middle Row Win:
  |   |  
---------
O | O | O
---------
  |   |  

Bottom Row Win:
  |   |  
---------
  |   |  
---------
X | X | X
```

### 2. Vertical Wins

```
Left Column Win:
X |   |  
---------
X |   |  
---------
X |   |  

Center Column Win:
  | O |  
---------
  | O |  
---------
  | O |  

Right Column Win:
  |   | X
---------
  |   | X
---------
  |   | X
```

### 3. Diagonal Wins

```
Main Diagonal (Top-Left to Bottom-Right):
X |   |  
---------
  | X |  
---------
  |   | X

Anti-Diagonal (Top-Right to Bottom-Left):
  |   | O
---------
  | O |  
---------
O |   |  
```

### All Conditions Summary Table

| Win Type       | Positions | Example |
|----------------|-----------|---------|
| **Top Row**    | 0,1,2     | `X X X` |
| **Middle Row** | 3,4,5     | `O O O` |
| **Bottom Row** | 6,7,8     | `X X X` |
| **Left Col**   | 0,3,6     | `X`<br>`X`<br>`X` |
| **Center Col** | 1,4,7     | `O`<br>`O`<br>`O` |
| **Right Col**  | 2,5,8     | `X`<br>`X`<br>`X` |
| **Main Diag**  | 0,4,8     | `X`<br>` X`<br>`  X` |
| **Anti-Diag**  | 2,4,6     | `  O`<br>` O`<br>`O` |

Would you like me to provide these as actual image files (PNG/SVG) instead of ASCII art? I can generate those programmatically if needed.

For your Vue.js game, these correspond to the win conditions array:
```javascript
const winConditions = [
  [0,1,2], [3,4,5], [6,7,8], // Rows
  [0,3,6], [1,4,7], [2,5,8], // Columns
  [0,4,8], [2,4,6]           // Diagonals
];
```

## Core Game Logic

### 1. Player Move Handling

```javascript
const move = (x, y) => {
  if (squares.value[x][y] === "" && !utils.gameEnd) {
    squares.value[x][y] = utils.player;
    playerSwap();
    if (utils.isComputer && !winner.value) executeComputer();
  }
};
```
- Validates the move is legal (empty square, game not ended)
- Updates the board
- Swaps players
- Triggers computer move if in computer mode

### 2. Computer Move Logic

```javascript
const executeComputer = () => {
  if (utils.player === "O" && utils.usedBoxes < 9) {
    const newSquares = computerChoice(squares.value.flat());
    squares.value = newSquares;
  }
};
```
- Only executes when computer's turn ("O")
- Uses `computerChoice()` to determine move based on difficulty level

### 3. Smart Choice Algorithm

```javascript
const smartChoice = (square, level) => {
  // Different strategies based on difficulty level
  if(level==="hard") {
    // 1. Try to win
    // 2. Block opponent
  } else if (level==="medium") {
    // Just block opponent
  } else if (level==="easy") {
    // Just try to win
  } else {
    // Random moves
  }
};
```

### 4. Win Detection

```javascript
const findWinner = (square) => {
  for (const conditions of winConditions) {
    const [a, b, c] = conditions;
    if (square[a] && square[a] === square[b] && square[a] === square[c]) {
      utils.conditionThatMatched = [a,b,c]; // Store winning indices
      return square[a]; // Return winning player
    }
  }
  return ""; // No winner
};
```

### 5. Game Flow Control

- `restart()`: Resets the entire game (scores, board, round)
- `next()`: Advances to next round while preserving scores

## Helper Functions

1. **Array Conversion**:
   - `array1dTo2d()`: Converts flat array back to 2D matrix

2. **Player Management**:
   - `playerSwap()`: Toggles between "X" and "O"

3. **Win Visualization**:
   - `updateWinBoxes()`: Highlights winning squares on the board

4. **Sound Effects**:
   - `playSounds()`: Plays sound effects for moves

## Key Design Patterns

1. **Reactive State Management**:
   - Uses Vue 3's Composition API (`reactive`, `ref`, `computed`)
   - Separates game board state (`squares`) from game utilities (`utils`)

2. **Computer AI**:
   - Implements different difficulty levels with strategic priorities
   - Uses the same win condition checking for both players

3. **Game Flow**:
   - Round-based system with score persistence
   - Clear separation between round advancement and full restart

## Usage Notes

1. The board is always represented as a 3x3 matrix but often flattened (0-8) for win checking
2. Computer moves are only triggered when `isComputer` is true and it's "O"'s turn
3. The `winBoxes` matrix tracks which squares are part of a winning condition for UI highlighting
4. All game state is reactive and will automatically update the UI when changed

This implementation provides a complete Tic-Tac-Toe game with configurable difficulty levels, proper win detection, and responsive UI updates through Vue's reactivity system.
