[English](README_EN.md) | [中文](README.md)

# Svelte Snake Game

This project is a classic Snake game example built with Svelte + TypeScript + Vite. It supports keyboard controls, score tracking, and increasing speed.

## How to Run

```bash
pnpm install
pnpm dev
```

## Main Features

- Classic Snake gameplay, supports arrow keys and WASD controls
- Eating red food makes the snake longer and increases the score, speed gradually increases
- Game over when hitting the wall or itself, can restart
- Real-time score display
- Responsive Canvas rendering, beautiful interface
- Pause/Resume supported (spacebar or button)

## Directory Structure

```
├── public/           # Static assets
├── src/
│   ├── app.css       # Global styles
│   ├── App.svelte    # App entry, mounts Snake component
│   ├── main.ts       # Startup entry
│   └── lib/
│       ├── Snake.svelte   # Main logic and UI for Snake game
│       └── Counter.svelte # Counter example (not used in main UI)
├── index.html        # HTML template
├── package.json      # Project dependencies and scripts
└── ...
```

## Game Controls

- Arrow keys or WASD to move the snake
- Spacebar to start/pause the game
- Eat red blocks to score and grow longer
- Game over if you hit the wall or yourself

## Other Notes

- `Counter.svelte` is a Svelte component counter example, not integrated into the main page, for learning reference only.
- To extend features (such as leaderboard, skins, etc.), continue development based on `Snake.svelte`.

---

This project is suitable for Svelte beginners and mini-game development enthusiasts to learn and reference.
