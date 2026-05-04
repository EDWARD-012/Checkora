# ♟️ Checkora C++ Chess Engine Architecture

This document explains how Checkora integrates its Django backend with a high-performance C++ chess engine.
It focuses on actual implementation — how commands flow, how communication works, and how AI decisions are computed.

---

## 🧠 1. System Overview

Checkora uses a hybrid architecture:

* **Django (Python)** → Handles API, game state, and frontend communication
* **C++ Engine** → Handles computation (move generation, validation, AI)

The two systems communicate using a **subprocess-based model**.

---

## 🔄 High-Level Flow

1. User performs an action on the frontend
2. Django receives and processes the request
3. Django sends a command to the C++ engine
4. C++ engine processes the request
5. Result is returned to Django
6. Django sends response back to frontend

---

## 🔗 2. Django ↔ C++ Communication

Django communicates with the C++ engine using Python’s subprocess module.

### ⚙️ Core Implementation

```python
proc = subprocess.Popen(
    self._build_engine_command(engine_path),
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
    text=True,
)

stdout, _ = proc.communicate(input=command, timeout=5)
return stdout.strip()
```

### 🧠 Explanation

* Django runs the C++ engine as a separate process
* Commands are sent via stdin
* Output is received via stdout

👉 This forms a **text-based communication protocol**

---

## 📡 3. Command Protocol

### 🔹 MOVES Command

**Purpose:** Get valid moves for a selected piece

**Input:**

```
MOVES <board> <castling_rights> <turn> <row> <col>
```

**Output:**

```
MOVES r c is_capture is_promotion ...
```

**Used in:** `game/engine.py` → `_get_engine_moves()`

---

### 🔹 BESTMOVE Command

**Purpose:** Calculate best move using AI

**Input:**

```
BESTMOVE <board> <castling_rights> <turn> <depth>
```

**Output:**

```
BESTMOVE <from_row> <from_col> <to_row> <to_col>
```

**Used in:** `get_ai_move()`

---

### 🔹 STATUS Command

**Purpose:** Check game state

**Output:**

```
STATUS checkmate | stalemate | check | ok
```

---

### 🔹 PROMOTE Command

**Purpose:** Handle pawn promotion

---

## ⚙️ 4. C++ Engine Handling

C++ engine works as a command processor:

1. Reads input using `cin`
2. Identifies command
3. Executes logic
4. Returns result using `cout`

👉 This follows a **command dispatcher pattern**

---

## 🤖 5. AI Logic — Minimax + Alpha-Beta Pruning

### Minimax

* Explores possible moves
* Evaluates game states
* Selects best move

### Alpha-Beta Pruning

* Skips unnecessary branches
* Improves performance

### Depth Control

```python
depth = self._get_ai_search_depth()
```

---

## 🔄 6. End-to-End Flow

1. User makes a move
2. Django processes request
3. Command is generated
4. C++ engine is invoked
5. Engine computes result
6. Output returned
7. Django parses result
8. Frontend updates UI

---

## 💥 Final Insight

Checkora combines:

* Python → control and flexibility
* C++ → speed and performance
* Communication → lightweight protocol

This results in a fast and scalable system.
