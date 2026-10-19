# Online Chess: Requirements & Design

## 1. Overview
A web-based chess game where two players in different browsers can play each other
in real time, or play against a computer engine, backed by a C++ HTTP server that
enforces the rules.

## 2. Scope

### v0.1 (MVP): Multiplayer
- Create a game and receive a shareable game ID
- Second player joins using the game ID
- Server validates every move under standard chess rules
- Both players see the board update after each move
- Check, checkmate, stalemate, and automatic draws detected
- Resign and draw offers

### v0.2: Chess Engine
- Play against a server-side engine with multiple difficulty levels

### v0.3: Real-Time & Polish
- Replace HTTP polling with WebSockets for live game updates

### Future (out of scope for now)
Accounts, ratings, chess clocks, chat, spectators, game history database,
browser-side engine via WebAssembly.

## 3. Architecture

### Components
| Component | Language | Responsibility |
|---|---|---|
| Chess rules engine | C++ | Board representation, move generation, legality checks, game-over detection. No networking. |
| Game manager | C++ | Tracks active games, player colors, turns, draw offers. |
| HTTP API layer | C++ | Receives HTTP requests, calls the game manager, returns JSON. |
| Chess AI (v0.2) | C++ | Searches for and returns the best move for a position. Runs on a separate thread. |
| Web client | HTML/CSS/JavaScript | Draws the board, sends moves, polls the server for updates. |

### Tech Stack
- **Server:** C++17, CMake, cpp-httplib, nlohmann/json
- **Testing:** GoogleTest, perft tests, cutechess-cli (engine matches)
- **Client:** HTML, CSS, JavaScript (chessboard.js for board rendering)
- **CI:** GitHub Actions
- **Board state format:** FEN

### API (v0.1 draft)
| Method | Endpoint | Description |
|---|---|---|
| POST | /games | Create a game. Returns `gameId` and the creator's color. Body may include `{"opponent":"engine","level":3}` (v0.2). |
| POST | /games/{id}/join | Join an existing game as the other color. |
| GET | /games/{id} | Get board state (FEN), whose turn, game status, move list, pending draw offers. |
| POST | /games/{id}/moves | Make a move. Body: `{"from":"e2","to":"e4"}` (plus `"promotion":"q"` when needed). |
| POST | /games/{id}/resign | Resign the game. |
| POST | /games/{id}/draw | Offer, accept, or decline a draw. Body: `{"action":"offer"}`, `"accept"`, or `"decline"`. |

### Real-Time Updates
- v0.1: Clients poll `GET /games/{id}` about once per second.
- v0.3: Upgrade to WebSockets so the server can push moves instantly.

## 4. Requirements

### Game Setup
| ID | Requirement | Milestone | Verification |
|---|---|---|---|
| REQ-001 | The system shall allow a user to create a new game and receive a unique game ID. | v0.1 | Unit/API test |
| REQ-002 | The system shall allow a second user to join an existing game using its ID. | v0.1 | API test |

### Rules & Gameplay
| ID | Requirement | Milestone | Verification |
|---|---|---|---|
| REQ-003 | The server shall reject any move that is illegal under standard chess rules. | v0.1 | Unit tests, perft tests |
| REQ-004 | The server shall support castling, en passant, and pawn promotion. | v0.1 | Unit tests, perft tests |
| REQ-005 | The server shall reject moves made out of turn. | v0.1 | Unit/API test |
| REQ-006 | The system shall detect and report checkmate and stalemate. | v0.1 | Unit tests |
| REQ-007 | Each client shall display the opponent's move within 2 seconds. | v0.1 | Manual/integration test |
| REQ-008 | A player shall be able to resign, ending the game. | v0.1 | API test |

### Draws
| ID | Requirement | Milestone | Verification |
|---|---|---|---|
| REQ-009 | A player shall be able to offer a draw to their opponent. | v0.1 | API test |
| REQ-010 | The opponent shall be able to accept or decline a draw offer. | v0.1 | API test |
| REQ-011 | A pending draw offer shall be withdrawn automatically when the opponent makes a move. | v0.1 | Unit test |
| REQ-012 | The system shall declare a draw by threefold repetition, the fifty-move rule, and insufficient material. | v0.1 | Unit tests |

### Chess Engine
| ID | Requirement | Milestone | Verification |
|---|---|---|---|
| REQ-013 | The system shall allow a user to start a game against the engine as either color. | v0.2 | API test |
| REQ-014 | The engine shall only ever play legal moves. | v0.2 | Automated engine matches |
| REQ-015 | The engine shall return a move within 5 seconds of the player's move. | v0.2 | Timing test |
| REQ-016 | The engine shall offer at least three difficulty levels. | v0.2 | API test |
| REQ-017 | At its highest level, the engine shall score at least 50% against Stockfish limited to 1800 Elo over 100 games. | v0.2 | cutechess-cli match |
| REQ-018 | At its highest level, the engine shall solve at least 70% of the "Win At Chess" tactical test suite within the per-move time limit. | v0.2 | Test suite run |
| REQ-019 | Engine searches shall not block other games from being served. | v0.2 | Concurrency test |

## 5. Traceability
Each requirement will be linked to the GitHub Issue that implements it and the
automated test that verifies it. Tests are named after their requirement
(e.g. `TEST(Req003, RejectsIllegalMove)`).

| Requirement | GitHub Issue | Test |
|---|---|---|
| REQ-001 | TBD | TBD |
| ... | | |
