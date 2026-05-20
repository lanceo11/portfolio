---
layout: post
title: GameLevelBasketball CS 111 Breakdown
description: In-depth, non-table breakdown of how GameLevelBasketball demonstrates CS 111 and CSSE concepts with a runnable GameRunner.
permalink: /basketball-cs111-breakdown
hide: true
toc: true
toc_history: true
codemirror: true
---

**Designated GameRunner**

{% capture basketball_runner_challenge %}
Run the Basketball level here while reading the breakdown below. Move with WASD, press E to shoot, and use this one runner as your live example for the full lesson.
{% endcapture %}

{% capture basketball_runner_code %}
import GameControl from '{{site.baseurl}}/assets/js/GameEnginev1.1/essentials/GameControl.js';
import GameLevelBasketball from '{{site.baseurl}}/assets/js/projects/kirby-minigames/levels/GameLevelBasketball.js';

export const gameLevelClasses = [GameLevelBasketball];
export { GameControl };
{% endcapture %}

{% include runners/game.html
   runner_id="basketball-main-runner"
   challenge=basketball_runner_challenge
   code=basketball_runner_code
   height="430px"
   editor_height="260px"
%}

## Overview

`GameLevelBasketball.js` is a strong CS 111 capstone example because it is not just a background with a moving player. It combines object-oriented design, state management, collision logic, keyboard input, canvas rendering, local storage, leaderboard API usage, and debugging-friendly structure in one playable level.

The core gameplay idea is simple: Astro survives on the court while Kirby chases him, coins spawn around the arena, and the player can shoot a basketball projectile to stun the chaser. That simple loop lets one file demonstrate a lot of computer science concepts in a very concrete way.

## Software Engineering Practices

This file shows planning and organization by splitting the level into focused methods instead of making one giant function. The constructor stores game state and configuration. `initialize()` sets up the round. `update()` runs frame-by-frame logic. `resetRound()` restores the level. `destroy()` cleans up listeners and DOM elements.

```js
constructor(gameEnv) {
  this.gameEnv = gameEnv;
  this.playerStart = { x: Math.round(width * 0.12), y: Math.round(height * 0.68) };
  this.chaserStart = { x: Math.round(width * 0.72), y: Math.round(height * 0.55) };

  this.caught = false;
  this.projectiles = [];
  this.shootCooldownMs = 5000;
  this.targetSurvivalSeconds = 20;
}
```

That is good software engineering because the constructor only defines state and setup data. It does not try to run the whole game loop itself.

The level also follows single responsibility pretty well through helper methods:

```js
updateProjectiles(now, lebron) { ... }
createHud() { ... }
submitRoundScore() { ... }
showIntroDialogue() { ... }
applyCoinSpawnRules() { ... }
clearProjectiles() { ... }
```

Each method has one clear job, which makes the level easier to debug, explain, and revise.

## Object-Oriented Programming and Classes

The level itself is a custom class:

```js
class GameLevelBasketball {
  constructor(gameEnv) {
    this.gameEnv = gameEnv;
    ...
  }
}
```

That class acts like the controller for the whole minigame. It owns the state, creates the objects, and defines the level rules.

Inside the level, object instantiation is data-driven through `this.classes`. Instead of manually drawing everything, the level tells the engine what classes to create and what data to pass into them.

```js
this.classes = [
  { class: GameEnvBackground, data: image_data_court },
  { class: Player, data: sprite_data_player },
  { class: Npc, data: sprite_data_chaser },
  { class: Coin, data: coin_1 },
  { class: Coin, data: coin_2 },
  { class: Coin, data: coin_3 },
  { class: Barrier, data: barrier_bench_top }
];
```

This is a great example of object instantiation and object literals working together. The level does not hardcode every object as a separate drawing routine. It builds them through configuration objects.

For inheritance, the strongest evidence comes from the engine classes used by `GameLevelBasketball`. The basketball level imports classes that already extend deeper engine classes:

```js
import Player from '@assets/js/GameEnginev1.1/essentials/Player.js';
import Npc from '@assets/js/GameEnginev1.1/essentials/Npc.js';
import Coin from '@assets/js/GameEnginev1.1/Coin.js';
import Barrier from '@assets/js/GameEnginev1.1/essentials/Barrier.js';
```

The engine shows the inheritance chain:

```js
class Player extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);
  }
}
```

```js
class Npc extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);
  }
}
```

```js
class Coin extends Npc {
  constructor(data = null, gameEnv = null) {
    super(coinData, gameEnv);
  }
}
```

That gives you a clear hierarchy:

`GameObject -> Character -> Player`

`GameObject -> Character -> Npc -> Coin`

So even though `GameLevelBasketball` is not itself an `extends` class, it demonstrates OOP by composing and configuring objects from an inheritance-based engine.

## Data Types and Object Literals

This level uses all the core data types required by the rubric.

Numbers are everywhere: positions, cooldowns, movement speed, projectile size, score math, and time tracking.

```js
this.projectileSpeed = 9;
this.projectileRadius = 10;
this.projectileLifeMs = 2200;
this.shootCooldownMs = 5000;
this.targetSurvivalSeconds = 20;
```

Strings are used for IDs, UI messages, directions, and storage keys.

```js
id: 'BasketballPlayer',
greeting: 'Ball handler ready.',
id: 'LeBron',
dialogues: ['LeBron is in the gym.']
```

Booleans control game state:

```js
this.caught = false;
this.preGameLocked = true;
this.levelCompleted = false;
this.completionTriggered = false;
```

Arrays store collections of objects and systems:

```js
this.projectiles = [];
this.classes = [ ... ];
dialogues: ['LeBron is in the gym.']
```

JSON-style object literals are one of the most important patterns in the file. The player, NPC, coins, barriers, and background are all defined with structured objects.

```js
const coin_1 = {
  id: 'coin_1',
  INIT_POSITION: { x: Math.round(width * 0.25), y: Math.round(height * 0.35) },
  SCALE_FACTOR: 18,
  hitbox: coinHitbox,
  value: 1
};
```

This is also evidence of data-driven design. Instead of making separate custom classes for every coin and barrier, the level feeds object data into reusable engine classes.

## Operators and Mathematical Reasoning

The chase system is one of the best places to show operators. The code uses subtraction to measure distance, `Math.hypot()` to calculate vector length, division to normalize movement, multiplication to scale speed, and `Math.min()` and `Math.max()` to keep values fair and inside the court.

```js
const dx = player.position.x - lebron.position.x;
const dy = player.position.y - lebron.position.y;
const dist = Math.hypot(dx, dy);
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);

lebron.position.x += (dx / dist) * speed;
lebron.position.y += (dy / dist) * speed;
```

This is a real example of mathematical operators solving a gameplay problem: how do you make the enemy chase the player smoothly instead of teleporting or moving in only one direction at a time?

String operations appear in HUD text and style updates:

```js
this.timeHud.textContent =
  `Time: ${this.currentTime.toFixed(1)}s/${this.targetSurvivalSeconds}s | Best: ${this.bestTime.toFixed(1)}s | ` +
  `Coins: ${this.getCoinsCollected()} | Best Coins: ${this.bestCoins}`;
```

Boolean expressions help control logic:

```js
if (event.key.toLowerCase() !== 'e' || event.repeat) return;
if (this.preGameLocked || this.caught) return;
if (now - this.lastShotAt < this.shootCooldownMs) return;
```

This is not just syntax practice. These expressions directly control fairness, pacing, and gameplay rules.

## Control Structures and State Management

The level uses conditionals constantly to control what should happen next. The `update()` method is a strong example because it checks several game states in order:

```js
update() {
  const player = this.findById('BasketballPlayer');
  const lebron = this.findById('LeBron');
  if (!player || !lebron) return;

  const now = performance.now();
  this.updateProjectiles(now, lebron);
  if (this.preGameLocked) return;

  if (!this.caught) {
    this.currentTime = (now - this.startTime) / 1000;
    this.updateHud();

    if (this.currentTime >= this.targetSurvivalSeconds) {
      this.completeLevel();
      return;
    }
  }
}
```

This method shows nested conditions clearly. The level first verifies objects exist. Then it updates projectile logic. Then it pauses most gameplay until the intro dialogue is dismissed. Then it updates the timer only if the player has not already been caught. Then it checks the win condition.

The file also uses iteration in a practical way. The projectile system loops backward through the projectile array so it can safely remove items while iterating:

```js
for (let i = this.projectiles.length - 1; i >= 0; i -= 1) {
  const projectile = this.projectiles[i];
  projectile.x += projectile.vx;
  projectile.y += projectile.vy;

  if (this.isProjectileOutOfBounds(projectile) || now - projectile.bornAt > this.projectileLifeMs) {
    this.removeProjectileAt(i);
    continue;
  }
}
```

That is a meaningful control-structure example because it solves a real bug risk. If the loop went forward while removing elements, indexes could shift and skip items.

State management is one of the strongest parts of the level. These flags make the game loop understandable:

```js
this.caught = false;
this.preGameLocked = true;
this.scoreSubmittedThisRound = false;
this.levelCompleted = false;
this.completionTriggered = false;
```

The state variables are then used to control round flow, pause conditions, completion behavior, and duplicate API prevention.

## Input, Output, and Rendering

Keyboard input is handled through event listeners:

```js
document.addEventListener('keydown', this.handleRestartKey);
document.addEventListener('keydown', this.handleShootKey);
```

The shoot handler is a clean example of input validation:

```js
handleShootKey(event) {
  if (event.key.toLowerCase() !== 'e' || event.repeat) return;
  if (this.preGameLocked || this.caught) return;
  const now = performance.now();
  if (now - this.lastShotAt < this.shootCooldownMs) return;
  ...
}
```

Canvas rendering appears in the projectile drawing code. Instead of relying on a sprite sheet, this level creates a basketball projectile directly with canvas drawing commands:

```js
drawProjectileSprite(ctx, width, height) {
  const cx = width / 2;
  const cy = height / 2;
  const r = Math.min(width, height) * 0.42;

  ctx.beginPath();
  ctx.arc(cx, cy, r, 0, Math.PI * 2);
  ctx.fillStyle = '#f68b1f';
  ctx.fill();
}
```

That is strong evidence of output and rendering because the program generates a visible game object from code.

The level also uses `gameEnv` for game configuration:

```js
const width = gameEnv.innerWidth;
const height = gameEnv.innerHeight;
this.gameEnv.stats.coinsCollected = 0;
const container = this.gameEnv.container || this.gameEnv.gameContainer;
```

So the file is not working in isolation. It reads environment size, stores live stats, and attaches new DOM/canvas elements into the game container.

## Collision Detection, Hitboxes, and Gameplay Logic

The collision system is one of the most teachable parts of this file. Instead of relying only on full sprite boundaries, the level calculates custom hitbox rectangles:

```js
getHitboxRect(obj) {
  const width  = obj.width  || 0;
  const height = obj.height || 0;
  const pos = obj.position || { x: 0, y: 0 };
  const widthReduction  = width * 0.2;
  const heightReduction = height * 0.2;

  return {
    left: pos.x + widthReduction,
    right: pos.x + width - widthReduction,
    top: pos.y + heightReduction,
    bottom: pos.y + height
  };
}
```

Then it compares two objects with rectangle overlap logic:

```js
isHitboxCollision(a, b) {
  const ar = this.getHitboxRect(a);
  const br = this.getHitboxRect(b);
  return (
    ar.left   < br.right  &&
    ar.right  > br.left   &&
    ar.top    < br.bottom &&
    ar.bottom > br.top
  );
}
```

That shows boolean expressions, geometry, and defensive programming working together.

Projectile collision uses a different pattern: circle-to-rectangle collision.

```js
isCircleHittingObject(projectile, obj) {
  const rect = this.getHitboxRect(obj);
  const nearestX = Math.max(rect.left, Math.min(projectile.x, rect.right));
  const nearestY = Math.max(rect.top,  Math.min(projectile.y, rect.bottom));
  const dx = projectile.x - nearestX;
  const dy = projectile.y - nearestY;
  return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius);
}
```

This is a very good snippet for a mini-lesson because it proves the game is using more than one collision strategy depending on the shape of the object.

## API Integration, Asynchronous I/O, and Persistence

This level also goes beyond local gameplay by saving data and submitting scores.

Local persistence is handled with `localStorage`:

```js
loadBestTime() {
  try {
    return Number(localStorage.getItem('basketball_best_time') || 0);
  } catch (_) {
    return 0;
  }
}
```

```js
saveBestCoins() {
  try {
    localStorage.setItem('basketball_best_coins', String(this.bestCoins));
  } catch (_) {}
}
```

That is good application debugging evidence too, because those values can be inspected in the browser Application tab.

For API integration, the level uses the leaderboard system:

```js
initLeaderboard() {
  if (this.leaderboard) return;
  this.leaderboard = new Leaderboard(this.gameEnv.gameControl, {
    gameName: 'Basketball',
    initiallyHidden: false
  });
}
```

Score submission is asynchronous:

```js
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));
```

This demonstrates promise-based async I/O and API error handling. It also prevents duplicate submission with a state flag:

```js
if (!this.leaderboard || this.scoreSubmittedThisRound) return;
```

So this part of the file shows both asynchronous programming and defensive design.

## Documentation, Debugging, and Testing Evidence

This file includes comments that explain why certain logic exists, especially around chase fairness, court boundaries, rendering, and collision alignment.

```js
// Speed curve -> LeBron gets slightly faster over time but has a cap to keep the game fair
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);
```

```js
// Keep the DOM hitbox aligned with the new spawn immediately.
if (typeof coin.setupCanvas === 'function') {
  coin.setupCanvas();
}
```

There is also debugging evidence in the warning messages:

```js
console.warn('Failed to emit basketball concept focus event:', err);
console.warn('Leaderboard score submit failed:', err);
```

For source-level debugging, this file is easy to step through because the logic is separated into named methods instead of buried in anonymous callbacks.

For element inspection, the HUD and projectile canvases are created dynamically:

```js
this.timeHud = document.createElement('div');
this.messageHud = document.createElement('div');
canvas: document.createElement('canvas')
```

For gameplay testing and verification, the level includes:

- A start dialogue to gate the round correctly
- A visible timer HUD
- Coin collection and respawn behavior
- A stun attack with cooldown
- Reset logic after getting caught
- Level completion after surviving long enough

All of that makes the game easy to demo live while also giving strong evidence for code review.

## Final CS 111 Alignment

`GameLevelBasketball` demonstrates the major CS 111 and CSSE objectives in a very direct way:

- It uses classes, objects, inheritance-based engine components, and constructor chaining.
- It uses numbers, strings, booleans, arrays, and object literals throughout the level.
- It uses conditionals, nested logic, and loops for real gameplay systems.
- It uses keyboard input, canvas rendering, DOM output, and `GameEnv` configuration.
- It uses API integration, asynchronous score submission, and local storage persistence.
- It shows state management, collision systems, debugging evidence, and testable game behavior.

What makes this file especially strong is that the concepts are not isolated practice exercises. They all support one playable level, so every programming concept is tied to an actual feature the player can see and test.
