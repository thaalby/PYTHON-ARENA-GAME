# 🐍 Πthon Arena — Snake Battle

A real-time, two-player online snake battle game built with **Python** and **Pygame**, using a custom TCP client-server architecture. Players connect to a central server, challenge each other to matches, and compete to outlast their opponent by collecting pies and avoiding obstacles — all through a fully graphical interface with music, sound effects, and a match replay system.

---


## ✨ Features

### Core Gameplay
- Real-time snake movement controlled via **Arrow Keys** or **WASD**
- Server-side game logic — clients only send inputs, the server is the single source of truth
- **40×30 board** running at a **150 ms tick rate** (~6.7 updates/second)
- **120-second match timer** — highest HP wins on timeout
- Players start with **100 HP**; the first to hit 0 loses

### Items & Obstacles
| Item | Effect |
|---|---|
| 🟡 Gold Pie | +HP & snake growth |
| ⚪ Silver Pie | +HP & snake growth |
| 🟣 Poison Pie | −HP |
| 🔴 Spike | −20 HP on collision |
| ⬛ Wall | −30 HP on collision |

### Progressive Difficulty
The pie pool shifts over time — early game is gold-heavy (easy), late game is poison-heavy (hard). Collision damage stays constant throughout.

### Customization
- **8 snake colors** to choose from
- **6 hat accessories**: None, Crown, Top Hat, Halo, Party Hat, Cowboy Hat
- **10 tile-based map themes**: Stone, Brick, Blue Tiles, Cobblestone, Ice, Sand, Wood, Slate, Marble, Jungle

### Multiplayer & Social
- Username verification (unique usernames enforced server-side)
- Real-time **lobby** showing all online players and active matches
- **Challenge system** with accept/decline flow
- **Spectator mode** — watch any ongoing match live from the lobby
- **In-game text chat** (press `T` during a match)

### Audio
- Looping **background music** during gameplay
- **Win** and **game over** sound effects on match end

### Match Replay *(Creative Feature)*
Every game tick is recorded on the client. After a match, the full replay is saved to a `replays/` folder as a timestamped JSON file. From the end screen, click **Watch Replay** to re-watch the entire match with:
- ⏸ **Space** — pause / resume
- ⬅➡ **Arrow keys** — scrub ±5 seconds
- **ESC** — exit replay

---

## 🏗 Architecture

Πthon Arena uses a **centralized client-server architecture over TCP/IP**.

```
┌─────────────┐        TCP / JSON        ┌──────────────────────┐
│   Client A  │ ◄──────────────────────► │                      │
│  (Pygame)   │                          │   server.py          │
└─────────────┘                          │                      │
                                         │  • Game logic        │
┌─────────────┐        TCP / JSON        │  • State broadcast   │
│   Client B  │ ◄──────────────────────► │  • Lobby management  │
│  (Pygame)   │                          │  • Thread per client │
└─────────────┘                          └──────────────────────┘
```

All messages are **newline-delimited JSON** over a single persistent TCP connection. The `"type"` field in every message identifies its purpose.

### Key Protocol Messages

| Message | Direction | Description |
|---|---|---|
| `join` | Client → Server | Request to join with a username |
| `join_ok` | Server → Client | Username accepted |
| `lobby` | Server → All | Updated player list and active games |
| `challenge` | Client → Server | Challenge another player |
| `challenge_request` | Server → Client | Notify target of incoming challenge |
| `challenge_accepted` | Server → Client | Challenge accepted, proceed to customization |
| `player_ready` | Client → Server | Done customizing, ready to start |
| `game_start` | Server → Players+Spectators | Match starting with board info |
| `move` | Client → Server | Directional input (UP/DOWN/LEFT/RIGHT) |
| `state` | Server → Players+Spectators | Full game state every tick |
| `game_over` | Server → Players+Spectators | Match ended with results |
| `watch` | Client → Server | Join an ongoing game as a spectator |
| `chat` | Client ↔ Server ↔ Client | Peer-to-peer message relayed by server |

---

## 🗂 Project Structure

```
πthon-arena/
│
├── server.py                  # TCP server — all game logic lives here
├── client.py                  # Pygame client — rendering and input only
│
├── resources/
│   ├── tilesets/              # PNG textures for the 10 map themes
│   │   ├── stone_gray.png
│   │   ├── brick_red.png
│   │   └── ...
│   └── sounds/
│       ├── bg_music.mp3       # Background music (loops during gameplay)
│       ├── game_over.wav      # Played on defeat
│       └── you_win.mp3        # Played on victory
│
└── replays/                   # Auto-created; stores match replay JSON files
```

---

## ⚙️ Requirements

- Python **3.8+**
- [Pygame](https://www.pygame.org/) — `pip install pygame`

All other dependencies (`socket`, `threading`, `json`, `uuid`, `random`, `time`) are part of the Python standard library.

---

## 🚀 Running the Game

### 1. Start the server

```bash
python3 server.py <port>
```

Example:
```bash
python3 server.py 5000
```

The server will print `[SERVER] Listening on port 5000 ...` when ready.

### 2. Launch a client

```bash
python3 client.py
```

Each player runs their own instance of the client. On the login screen, enter the **server IP**, **port**, and a **unique username**, then hit Connect.

> To play locally, use `127.0.0.1` as the server IP.

---

## 🎮 Controls

| Key | Action |
|---|---|
| `↑` / `W` | Move Up |
| `↓` / `S` | Move Down |
| `←` / `A` | Move Left |
| `→` / `D` | Move Right |
| `T` | Open chat input |
| `M` | Cycle map theme mid-game |
| `ESC` | Cancel / go back |

**Replay controls:**

| Key | Action |
|---|---|
| `Space` | Pause / Resume |
| `←` / `→` | Seek ±5 seconds |
| `ESC` | Exit replay |

---

## 🔌 Game Flow

```
Splash Screen
    └─► Login (IP · Port · Username)
            └─► Lobby (see online players & active games)
                    ├─► Challenge a player
                    │       └─► Wait for accept/decline
                    │               └─► Customize snake (color + hat)
                    │                       └─► Pick map theme
                    │                               └─► Match starts
                    │                                       └─► End Screen
                    │                                               └─► Watch Replay / Back to Lobby
                    └─► Spectate an active game
```

---

## 📡 Server Details

- One **thread per client** connection (`handle_client`)
- One **thread per active game** (`game_loop`)
- A global `threading.Lock()` protects all shared state (`clients`, `games`, `pending`, `ready`, `customs`)
- Snakes respawn at a random free cell after any collision (wall, obstacle, self, or opponent)
- Disconnecting mid-game forfeits the match to the opponent

---

## 📄 License

This project is open source. Feel free to fork, modify, and build on it.
