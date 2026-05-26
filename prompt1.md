# **Python Game Development Prompt**

---

## **Role and Context**

You are a Senior Python Game Developer and UI/UX Engineer with deep expertise in building production-quality desktop applications using Python's tkinter GUI framework. You specialize in crafting immersive, visually polished game experiences that feel modern and intentional  \- not like typical beginner projects. Your work bridges clean software architecture with delightful user experience design.

Your task is to design and implement a complete, single-file Python Tic Tac Toe game that meets production-level standards across code quality, visual design, user interaction, and intelligent gameplay. The game must feel like a premium desktop application. The final deliverable is one `.py` file, runnable directly with no external dependencies beyond Python's standard library  \- portable, self-contained, and immediately playable by any user on Python 3.8 or above.

---

## **Core Objectives**

The game must simultaneously achieve all of the following:

* A beautiful, modern dark-themed GUI built with tkinter and Canvas-based rendering, featuring custom colors, hover effects, and smooth visual feedback.  
* Two fully functional game modes: Player vs. AI and Player vs. Player (two humans on the same machine).  
* An unbeatable AI opponent using the Minimax algorithm with Alpha-Beta Pruning, with three selectable difficulty levels  \- Easy, Medium, and Hard.  
* Live score tracking persisted across rounds within the session, with immediate visual updates after each result.  
* Animated move placement, win-cell highlighting, dynamic turn indicators, and contextual status messages throughout gameplay.  
* Robust game state management with clean separation between UI rendering, game logic, and AI computation.  
* Graceful handling of all edge cases: clicking occupied cells, interacting during AI thinking time, restarting mid-game, and rapid button presses.

---

## **Visual Design and Color System**

The game must implement a professional dark color palette applied consistently across every UI component. No color should ever be hardcoded inline  \- all values must be defined once in a `self.colors` dictionary.

| Element | Color | Hex |
| ----- | ----- | ----- |
| Background | Deep navy | `#1a1a2e` |
| Panel / Surface | Lighter navy | `#16213e` |
| Accent / Interactive | Rich indigo | `#0f3460` |
| Player X | Coral red | `#e94560` |
| Player O / AI | Electric cyan | `#00b4d8` |
| Win highlight | Gold | `#ffd700` |
| Start button / Success | Teal green | `#4ecca3` |
| Primary text | White | `#ffffff` |
| Secondary text | Muted lavender | `#a8b2d8` |

The window must be fixed at **520 × 720 pixels** with `resizable(False, False)`. The layout is a single vertical stack containing, from top to bottom: title bar, mode selector panel, difficulty selector, scoreboard, turn indicator, game board canvas, status label, and control buttons.

---

## **Animation and Interaction Requirements**

Every interaction must feel intentional. The following behaviors are required:

**Marker pop-in animation.** When X or O is placed, animate the font size through a multi-step sequence  \- small → growing → slightly overshooting → settling at final size. This creates a natural bounce-in effect. Implement using `root.after()` scheduling with no external libraries.

**Cell hover effects.** When a user hovers over an empty, playable cell, its background transitions to the accent color. On mouse leave, it reverts to the surface color. Hover must not activate on occupied cells, during the AI's turn, or when no game is active.

**Win highlight.** The three winning cells immediately switch to a gold background with dark-colored text on game end. This must be instant  \- no animation delay.

**AI thinking delay.** A deliberate 400ms delay before the AI places its marker, during which the turn label displays  \-AI is thinking… \-. This creates a realistic pause that makes gameplay feel natural and considered.

**Score feedback.** Score labels update immediately after each round ends, giving players instant visual confirmation of the result.

---

## **Game Board and Cell Interaction**

The game board is rendered entirely on a `tk.Canvas` widget  \- not a `tk.Button` grid. This provides full control over hover states, animated text items, win overlay colors, and clean line-based grid separators. The canvas is **360 × 360 pixels** with a 3×3 grid drawn via line primitives.

Each of the nine cells consists of two canvas items: a rectangle item and a text item, both tagged with a consistent naming convention (`cell0`–`cell8`, `text0`–`text8`) and both bound to the same click handler using `canvas.tag_bind()` with lambda default argument capture.

**Click guard rules (enforced at the top of the move handler, silently):**

* Clicking an occupied cell: ignored.  
* Clicking during the AI's turn: ignored.  
* Clicking before the game has been started: ignored.

**Human move flow:** User clicks cell → guard check → `place_marker()` → `animate_marker()` → `check_winner()` → if no winner, switch player → `update_turn_label()` → if AI mode and AI's turn, schedule `ai_move()` via `root.after(400)`.

**AI move flow:** `ai_move()` fires after 400ms → selects move by difficulty → `place_marker()` → `animate_marker()` → `check_winner()` → if no winner, switch to Player X → `update_turn_label()`.

---

## **AI Engine**

The AI system implements three difficulty tiers using a shared codebase:

**Easy:** Selects a completely random empty cell. No strategy. Beatable by any player.

**Medium:** Probabilistic blend  \- 60% of the time plays the optimal Minimax move, 40% of the time plays randomly. Creates a challenging but imperfect opponent.

**Hard:** Full Minimax with Alpha-Beta Pruning. Mathematically unbeatable  \- the best a human can achieve is a draw with perfect play.

**Minimax specification:**

minimax(board, depth, is\_maximizing, alpha, beta):  
    check terminal state:  
        AI win  → return (10 \- depth)  
        Player win → return (depth \- 10\)  
        Draw → return 0  
    if maximizing (AI):  
        iterate empty cells, recurse as minimizing  
        track max score, apply alpha pruning  
    if minimizing (Player):  
        iterate empty cells, recurse as maximizing  
        track min score, apply beta pruning  
    return best score

The depth adjustment (`10 - depth` / `depth - 10`) makes the AI prefer faster wins and slower losses, producing natural-feeling gameplay. The AI always plays as `PLAYER_O`. The best-move selector iterates all nine cells, temporarily places the AI marker, runs minimax, undoes the move, and returns the index with the highest score.

**Critical requirement:** The minimax engine must call a pure function `check_winner_board(board)`  \- never the UI-coupled `check_winner()` method. These two functions must remain strictly separate.

---

## **Game State Management**

All state lives in `self`. No global variables. The following variables must be maintained:

| Variable | Type | Purpose |
| ----- | ----- | ----- |
| `board` | `list[str]` (len 9\) | Current board; each cell is  `- -`,  `-X -`, or  `-O -` |
| `current_player` | `str` |  `-X -` or  `-O -`  \- whose turn it is |
| `game_mode` | `tk.StringVar` |  `-ai -` or  `-human -` |
| `ai_difficulty` | `tk.StringVar` |  `-easy -`,  `-medium -`, or  `-hard -` |
| `game_active` | `bool` | Whether a round is currently in play |
| `scores` | `dict` | `{X: int, O: int, draws: int}` persisted across rounds |
| `cell_rects` | `list` | Canvas rectangle item IDs for all 9 cells |
| `cell_texts` | `list` | Canvas text item IDs for all 9 cells |

**Win detection** checks all 8 combinations after every move  \- 3 rows, 3 columns, 2 diagonals. On a win: call `highlight_win(combo)` then `game_over(winner)`. If no empty cells remain and no winner: `game_over(None)` for a draw.

**`start_game()` must:** reset board to 9 empty strings, set `current_player` to X, set `game_active = True`, clear all canvas text and cell background colors, redraw grid lines, update the turn label, and change the start button text to  \-PLAY AGAIN \-.

**`reset_scores()` must:** zero out the scores dict, update all three score labels, and auto-start a new game.

---

## **Code Architecture**

Implement the entire game as a single class `TicTacToe` with strict method-level responsibility separation:

TicTacToe  
├── \_\_init\_\_()               Initialize state, create root, build UI, enter mainloop  
├── \_build\_ui()              Construct all widgets and layout  
├── \_draw\_grid()             Canvas grid line rendering  
├── \_create\_cells()          Canvas cell items and event bindings  
├── set\_mode(mode)           Toggle game mode, show/hide difficulty row  
├── start\_game()             Reset and begin a new round  
├── make\_move(idx)           Handle human player clicks with guards  
├── \_place\_marker(idx, p)    Update board state and canvas text  
├── \_animate\_marker(idx)     Multi-step bounce-in font animation  
├── \_hover\_in(idx)           Cell hover enter effect  
├── \_hover\_out(idx)          Cell hover leave effect  
├── \_ai\_move()               Trigger AI decision by difficulty  
├── \_random\_move()           Easy AI: random empty cell  
├── \_best\_move()             Hard AI: calls minimax, returns best index  
├── \_minimax(...)            Alpha-beta pruning engine (pure logic)  
├── \_winning\_combo(board)    Pure function: returns winning combo or None  
├── \_highlight\_win(combo)    Gold cell highlight on win  
├── \_game\_over(winner)       End state: update scores and labels  
├── \_update\_turn\_label()     Dynamic turn/thinking/result indicator  
├── \_update\_scores()         Refresh all three score display labels  
├── reset\_scores()           Session score reset  
└── \_add\_hover(widget, ...)  Reusable button hover binding helper

**Code quality standards:**

* Single, clearly defined responsibility per method  \- no method does double duty.  
* All colors referenced from `self.colors`  \- never a hardcoded hex anywhere else in the code.  
* Canvas tag naming: `f -rect_{i} -` and `f -text_{i} -` for consistent binding.  
* Event bindings use `lambda e, idx=i:` default argument capture to avoid closure bugs.  
* No imports beyond `tkinter`, `math`, and `random`.

---

## **Deliverable Requirements**

The output must be a single `.py` file:

* **Zero external dependencies**  \- no pip installs, no third-party packages.  
* **Run command:** `python goldenresponse1.py`  
* **Python version:** 3.8 and above.  
* **Cross-platform:** works on Windows, macOS, and Linux without modification. tkinter ships with all standard Python distributions.  
* **Self-contained entry point:** the `if __name__ ==  -__main__ -:` block instantiates `TicTacToe()` directly  \- no wrapper functions needed.

All edge cases must be handled silently (no error dialogs, no crashes): clicking occupied cells, clicking during AI thinking, clicking before game start, changing mode or difficulty mid-game (applies on next start), and rapid repeated clicks on the start button.

