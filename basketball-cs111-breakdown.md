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

`GameLevelBasketball.js` is a strong CS 111 capstone example because it combines object-oriented design, state management, collision logic, keyboard input, canvas rendering, local storage, leaderboard API usage, and debugging-friendly structure in one playable level.

The core gameplay loop is simple:

- Astro survives on the court.
- Kirby chases Astro around the map.
- Coins spawn around the arena.
- The player can shoot a basketball projectile to stun the chaser.

That simple loop gives one file a lot of real CS concepts to explain.

## Required Evidence for College Credit

This page follows the CS 111 evidence table directly. Every code snippet below is pulled from [`GameLevelBasketball.js`](/Users/lanceoberiano/lanceo11/portfolio/_projects/kirby-minigames/levels/GameLevelBasketball.js) only. If a rubric row is not fully provable from this one file, I say that clearly instead of using code from somewhere else.

## Software Engineering Practices

This file is organized into focused methods instead of one giant function.

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

<a id="writing-classes"></a>
### Writing Classes

**Project Evidence Required:** Create a minimum of 2 custom character classes extending base classes.  
**Assessment Method:** Code review of `Player.js`, `NPC.js`, `Enemy.js`-style class definitions.

```js
class GameLevelBasketball {
  constructor(gameEnv) {
    this.gameEnv = gameEnv;
    const width = gameEnv.innerWidth;
    const height = gameEnv.innerHeight;
    this.playerStart = { x: Math.round(width * 0.12), y: Math.round(height * 0.68) };
    this.chaserStart = { x: Math.round(width * 0.72), y: Math.round(height * 0.55) };

    this.caught = false;
    this.caughtAt = 0; 
    this.roundResetDelayMs = 1400; 
    this.startTime = 0;
    this.currentTime = 0;
    this.bestTime = this.loadBestTime();
    this.bestCoins = this.loadBestCoins();
    this.timeHud = null;
    this.messageHud = null;
    this.bottomNav = null; 
    this.leaderboard = null;
    this.introDialogue = null; 
    this.preGameLocked = true; 
    this.scoreSubmittedThisRound = false; 
    this.handleRestartKey = this.handleRestartKey.bind(this); 
    this.handleShootKey = this.handleShootKey.bind(this); 
    this.projectiles = [];
    this.projectileSpeed = 9; 
    this.projectileRadius = 10; 
    this.projectileLifeMs = 2200; 
    this.shootCooldownMs = 5000; 
    this.lastShotAt = -Infinity; 
    this.lebronStunUntil = 0;
    this.lebronStunDurationMs = 3000;
    this.levelCompleted = false;
    this.completionTriggered = false;
    this.targetSurvivalSeconds = 20;
    this.firstStealScrollTriggered = false;
```

- Defining this class contains all the GameLevel's Content into "GameLevelBasketball"
- The class holds the properties, game loops, asset settings, etc

```js
import GameEnvBackground from '@assets/js/GameEnginev1.1/essentials/GameEnvBackground.js';
import Player from '@assets/js/GameEnginev1.1/essentials/Player.js';
import Npc from '@assets/js/GameEnginev1.1/essentials/Npc.js';
import Coin from '@assets/js/GameEnginev1.1/Coin.js';
import Barrier from '@assets/js/GameEnginev1.1/essentials/Barrier.js';
import Leaderboard from '@assets/js/GameEnginev1.1/essentials/Leaderboard.js';
import DialogueSystem from '@assets/js/GameEnginev1.1/essentials/DialogueSystem.js';
import KirbyLevelMusic from './KirbyLevelMusic.js';
import { getKirbyImageUrl } from './kirbyAssetPaths.js';
```

- Importing these .js files pulls in their content into GameLevelBasketball
- Importing files such as coin.js, player.js, and Npc.js give those parts the instructions they need to function. GameLevelBasketball arranges these aspects to where they need to be.

<a id="methods-and-parameters"></a>
### Methods & Parameters

**Project Evidence Required:** Implement methods with parameters, such as `collisionHandler(other, direction)`.  
**Assessment Method:** Code review of method signatures with 2 or more parameters.

```js
isCircleHittingObject(projectile, obj) {
    const rect = this.getHitboxRect(obj);
    const nearestX = Math.max(rect.left, Math.min(projectile.x, rect.right));
    const nearestY = Math.max(rect.top,  Math.min(projectile.y, rect.bottom));
    const dx = projectile.x - nearestX;
    const dy = projectile.y - nearestY;
    return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius);
```

- In this instance, the two parameters are the projectile and the target object
- It uses math logic to calculate the shortest distance between the circle of the projecile and the rectangular boundary

```js
updateProjectiles(now, lebron) {
    for (let i = this.projectiles.length - 1; i >= 0; i -= 1) {
      const projectile = this.projectiles[i];
      projectile.x += projectile.vx;
      projectile.y += projectile.vy;
```
- This parameter takes the current time and the lebron (player character) to update player movement and stun reactions

<a id="instantiation-and-objects"></a>
### Instantiation & Objects

**Project Evidence Required:** Instantiate game objects in GameLevel configuration.  
**Assessment Method:** Code review of GameLevel setup objects.

```js
this.classes = [
  { class: GameEnvBackground, data: image_data_court },
  { class: Player, data: sprite_data_player },
  { class: Npc, data: sprite_data_chaser },
  { class: Coin, data: coin_1 },
  { class: Coin, data: coin_2 },
  { class: Coin, data: coin_3 },
  { class: Barrier, data: barrier_bench_top },
  { class: Barrier, data: barrier_bench_bottom },
  { class: Barrier, data: barrier_gatorade_left },
  { class: Barrier, data: barrier_gatorade_right }
];
```

- `this.classes` is the GameLevel configuration array that the GameBuilder reads to instantiate all game objects
- Each entry pairs a class (like `Player`, `Npc`, `Coin`) with its data object, which the engine uses to construct and place the object in the world

---

<a id="writing-classes"></a>
### Writing Classes

**Project Evidence Required:** Create a minimum of 2 custom character classes extending base classes.  
**Assessment Method:** Code review of `Player.js`, `NPC.js`, `Enemy.js`-style class definitions.

```js
class GameLevelBasketball {
  constructor(gameEnv) {
    this.gameEnv = gameEnv;
    const width = gameEnv.innerWidth;
    const height = gameEnv.innerHeight;
    this.playerStart = { x: Math.round(width * 0.12), y: Math.round(height * 0.68) };
    this.chaserStart = { x: Math.round(width * 0.72), y: Math.round(height * 0.55) };

    this.caught = false;
    this.caughtAt = 0; 
    this.roundResetDelayMs = 1400; 
    this.startTime = 0;
    this.currentTime = 0;
    this.bestTime = this.loadBestTime();
    this.bestCoins = this.loadBestCoins();
    this.timeHud = null;
    this.messageHud = null;
    this.bottomNav = null; 
    this.leaderboard = null;
    this.introDialogue = null; 
    this.preGameLocked = true; 
    this.scoreSubmittedThisRound = false; 
    this.handleRestartKey = this.handleRestartKey.bind(this); 
    this.handleShootKey = this.handleShootKey.bind(this); 
    this.projectiles = [];
    this.projectileSpeed = 9; 
    this.projectileRadius = 10; 
    this.projectileLifeMs = 2200; 
    this.shootCooldownMs = 5000; 
    this.lastShotAt = -Infinity; 
    this.lebronStunUntil = 0;
    this.lebronStunDurationMs = 3000;
    this.levelCompleted = false;
    this.completionTriggered = false;
    this.targetSurvivalSeconds = 20;
    this.firstStealScrollTriggered = false;
```

- Defining this class contains all the GameLevel's content into `GameLevelBasketball`
- The class holds the properties, game loops, asset settings, etc

```js
import GameEnvBackground from '@assets/js/GameEnginev1.1/essentials/GameEnvBackground.js';
import Player from '@assets/js/GameEnginev1.1/essentials/Player.js';
import Npc from '@assets/js/GameEnginev1.1/essentials/Npc.js';
import Coin from '@assets/js/GameEnginev1.1/Coin.js';
import Barrier from '@assets/js/GameEnginev1.1/essentials/Barrier.js';
import Leaderboard from '@assets/js/GameEnginev1.1/essentials/Leaderboard.js';
import DialogueSystem from '@assets/js/GameEnginev1.1/essentials/DialogueSystem.js';
import KirbyLevelMusic from './KirbyLevelMusic.js';
import { getKirbyImageUrl } from './kirbyAssetPaths.js';
```

- Importing these `.js` files pulls in their content into `GameLevelBasketball`
- Files such as `Coin.js`, `Player.js`, and `Npc.js` each define their own class that extends the engine's base `GameObject` class, forming the inheritance chain `GameObject → Character → Player` and `GameObject → Character → Npc`
- `GameLevelBasketball` orchestrates all of these classes by arranging them in the `this.classes` array

---

<a id="methods-and-parameters"></a>
### Methods & Parameters

**Project Evidence Required:** Implement methods with parameters, such as `collisionHandler(other, direction)`.  
**Assessment Method:** Code review of method signatures with 2 or more parameters.

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

- Takes two parameters: the `projectile` (a circle) and the target `obj` (a rectangle)
- Uses math to find the nearest point on the rectangle to the circle's center, then checks if the distance falls within the projectile's radius

```js
updateProjectiles(now, lebron) {
  for (let i = this.projectiles.length - 1; i >= 0; i -= 1) {
    const projectile = this.projectiles[i];
    projectile.x += projectile.vx;
    projectile.y += projectile.vy;
    ...
  }
}
```

- Takes the current timestamp `now` and the `lebron` game object as parameters
- Uses `now` to expire old projectiles and checks `lebron` for hit detection each frame

```js
drawProjectileSprite(ctx, width, height) {
  const cx = width / 2;
  const cy = height / 2;
  const r = Math.min(width, height) * 0.42;
  ctx.beginPath();
  ctx.arc(cx, cy, r, 0, Math.PI * 2);
  ctx.fillStyle = '#f68b1f';
  ctx.fill();
  ...
}
```

- Takes three parameters: the canvas context `ctx`, and the `width` and `height` of the drawing area
- Uses all three to calculate the center point and radius before drawing the basketball sprite

---

<a id="instantiation-and-objects"></a>
### Instantiation & Objects

**Project Evidence Required:** Instantiate game objects in GameLevel configuration.  
**Assessment Method:** Code review of GameLevel setup objects.

```js
this.classes = [
  { class: GameEnvBackground, data: image_data_court },
  { class: Player, data: sprite_data_player },
  { class: Npc, data: sprite_data_chaser },
  { class: Coin, data: coin_1 },
  { class: Coin, data: coin_2 },
  { class: Coin, data: coin_3 },
  { class: Barrier, data: barrier_bench_top },
  { class: Barrier, data: barrier_bench_bottom },
  { class: Barrier, data: barrier_gatorade_left },
  { class: Barrier, data: barrier_gatorade_right }
];
```

- `this.classes` is the GameLevel configuration array that the GameBuilder reads to instantiate all game objects
- Each entry pairs a class with its data object, which the engine uses to construct and place the object in the world

---

<a id="inheritance"></a>
### Inheritance (Basic)

**Project Evidence Required:** Create a class hierarchy with 2+ levels.  
**Assessment Method:** Code review of `extends` keyword and inheritance chain.

```js
import Player from '@assets/js/GameEnginev1.1/essentials/Player.js';
import Npc from '@assets/js/GameEnginev1.1/essentials/Npc.js';
import Coin from '@assets/js/GameEnginev1.1/Coin.js';
import Barrier from '@assets/js/GameEnginev1.1/essentials/Barrier.js';
```

- Each of these imported classes uses `extends` internally to build a hierarchy: `GameObject → Character → Player` and `GameObject → Character → Npc`
- `GameLevelBasketball` directly depends on this chain — when it places a `Player` or `Npc` into `this.classes`, the engine walks up the prototype chain to call inherited methods like `update()` and `draw()`

```js
{ class: Player, data: sprite_data_player },
{ class: Npc, data: sprite_data_chaser },
```

- These two entries are what cause the engine to instantiate objects from those inherited class hierarchies at runtime

---

<a id="method-overriding"></a>
### Method Overriding

**Project Evidence Required:** Override parent methods such as `update()`, `draw()`, `handleCollision()`.  
**Assessment Method:** Code review of polymorphic implementations.

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
  ...
}
```

- `update()` is the standard game loop method defined on the GameLevel interface, and `GameLevelBasketball` provides its own full implementation
- This version handles the chase logic, stun mechanics, coin tracking, and level completion rather than delegating to a parent

```js
coin.randomizePosition = () => {
  const xMin = bounds.xMin;
  const xMax = bounds.xMax;
  const yMin = bounds.yMin;
  const yMax = bounds.yMax;
  coin.position.x = xMin + Math.random() * Math.max(1, xMax - xMin);
  coin.position.y = yMin + Math.random() * Math.max(1, yMax - yMin);

  if (typeof coin.setupCanvas === 'function') {
    coin.setupCanvas();
  }
};
```

- This dynamically overrides the `randomizePosition` method on each coin object at runtime, replacing the default behavior with one that respects the court's spawn bounds

---

<a id="constructor-chaining"></a>
### Constructor Chaining

**Project Evidence Required:** Use `super()` to chain constructors.  
**Assessment Method:** Code review of `super(data, gameEnv)` calls.

```js
import Player from '@assets/js/GameEnginev1.1/essentials/Player.js';
import Npc from '@assets/js/GameEnginev1.1/essentials/Npc.js';
import Coin from '@assets/js/GameEnginev1.1/Coin.js';
```

- `Player`, `Npc`, and `Coin` each use `super(data, gameEnv)` in their constructors to pass configuration up to the `GameObject` base class
- When `GameLevelBasketball` places these into `this.classes` and the engine instantiates them, the constructor chain runs automatically: the child constructor calls `super()`, which runs the parent constructor, which sets up the shared sprite, position, and canvas properties

```js
const sprite_data_player = {
  id: 'BasketballPlayer',
  INIT_POSITION: { ...this.playerStart },
  pixels: { height: 770, width: 513 },
  ...
};
```

- The data objects passed here are what get forwarded through `super(data, gameEnv)` up the chain, so every property defined here ultimately lands in the base class constructor

---

<a id="iteration"></a>
### Iteration

**Project Evidence Required:** Use loops for game object arrays, animation frames.  
**Assessment Method:** Code review of `for`, `forEach`, `while` loops.

```js
for (let i = this.projectiles.length - 1; i >= 0; i -= 1) {
  const projectile = this.projectiles[i];
  projectile.x += projectile.vx;
  projectile.y += projectile.vy;
  ...
}
```

- Iterates backwards through the projectiles array so items can be safely removed mid-loop without skipping elements

```js
coins.forEach((coin) => {
  if (!coin._originalRandomizePosition && typeof coin.randomizePosition === 'function') {
    coin._originalRandomizePosition = coin.randomizePosition.bind(coin);
  }
  coin.randomizePosition = () => { ... };
  coin.randomizePosition();
});
```

- Uses `forEach` to apply custom spawn bounds and override `randomizePosition` on every coin object in the level

---

<a id="conditionals"></a>
### Conditionals

**Project Evidence Required:** Implement collision detection, state transitions.  
**Assessment Method:** Code review of `if/else`, nested conditions.

```js
if (this.isHitboxCollision(player, lebron)) {
  this.caught = true;
  this.caughtAt = now;
  this.bestTime = Math.max(this.bestTime, this.currentTime);
  ...
  this.showCaughtMessage();
  this.updateHud();
}
```

- Checks for hitbox overlap between the player and LeBron each frame; when true, transitions the game into the caught state and triggers score saving and the reset countdown

```js
if (this.caught) {
  if (now - this.caughtAt >= this.roundResetDelayMs) {
    this.resetRound();
  }
  return;
}
```

- A state transition conditional: once caught, the game waits the reset delay before calling `resetRound()` and early-returns to skip all other update logic

---

<a id="nested-conditions"></a>
### Nested Conditions

**Project Evidence Required:** Complex game logic combining multiple conditions.  
**Assessment Method:** Code review of multi-level conditionals.

```js
if (lebron && this.isCircleHittingObject(projectile, lebron)) {
  this.lebronStunUntil = Math.max(this.lebronStunUntil, now + this.lebronStunDurationMs);
  lebron.velocity.x = 0;
  lebron.velocity.y = 0;
  this.removeProjectileAt(i);
}
```

- Inside the projectile update loop, checks both that LeBron exists and that the projectile is hitting him before applying the stun — a compound condition nested inside the loop's bounds check

```js
if (now < this.lebronStunUntil) {
  lebron.velocity.x = 0;
  lebron.velocity.y = 0;
  return;
}
```

- Nested within the main `update()` flow after the `preGameLocked` and `caught` guards; only reached when the game is actively running, and halts LeBron's movement for the stun duration

---

<a id="numbers"></a>
### Numbers

**Project Evidence Required:** Position, velocity, score tracking.  
**Assessment Method:** Code review of numeric properties.

```js
this.projectileSpeed = 9; 
this.projectileRadius = 10; 
this.projectileLifeMs = 2200; 
this.shootCooldownMs = 5000; 
this.lastShotAt = -Infinity; 
this.lebronStunUntil = 0;
this.lebronStunDurationMs = 3000;
this.targetSurvivalSeconds = 20;
```

- Numeric constants control all physics-adjacent values: how fast projectiles travel, how long they live, cooldown windows, and the win condition timer

```js
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);
lebron.position.x += (dx / dist) * speed;
lebron.position.y += (dy / dist) * speed;
```

- Speed scales up over time but is capped, and position is updated each frame using vector arithmetic

---

<a id="strings"></a>
### Strings

**Project Evidence Required:** Character names, sprite paths, game states.  
**Assessment Method:** Code review of string manipulation.

```js
const sprite_src_player = getKirbyImageUrl('astro.png');
const sprite_src_chaser = getKirbyImageUrl('kirby.png');
```

- String filenames are passed to `getKirbyImageUrl()` to build full asset paths for each sprite

```js
const coins = this.gameEnv.gameObjects.filter(
  (obj) => String(obj?.spriteData?.id || '').startsWith('coin_')
);
```

- Uses string methods `String()` and `.startsWith()` to filter the game object list down to only coin instances by their ID prefix

---

<a id="booleans"></a>
### Booleans

**Project Evidence Required:** Flags such as `isJumping`, `isPaused`, `isVulnerable`.  
**Assessment Method:** Code review of boolean logic.

```js
this.caught = false;
this.preGameLocked = true; 
this.scoreSubmittedThisRound = false; 
this.levelCompleted = false;
this.completionTriggered = false;
this.firstStealScrollTriggered = false;
```

- Boolean flags gate every major state in the level: whether the round has started, whether the player was caught, whether the score was already saved, and whether the completion event already fired

```js
if (this.preGameLocked) return;
```

- A single boolean check blocks all update logic until the player clicks Start in the intro dialogue

---

<a id="arrays"></a>
### Arrays

**Project Evidence Required:** Game object collections, level data.  
**Assessment Method:** Code review of array operations.

```js
this.projectiles = [];
...
this.projectiles.push(projectile);
...
this.projectiles.splice(index, 1);
```

- `this.projectiles` is a live array of active basketball projectiles; objects are pushed in when fired and spliced out when they expire or hit LeBron

```js
this.classes = [
  { class: GameEnvBackground, data: image_data_court },
  { class: Player, data: sprite_data_player },
  ...
];
```

- The `this.classes` array is the level's data manifest — the GameBuilder reads it to know which objects to construct and place

---

<a id="objects-json"></a>
### Objects (JSON)

**Project Evidence Required:** Configuration objects, sprite data.  
**Assessment Method:** Code review of object literals.

```js
const sprite_data_player = {
  id: 'BasketballPlayer',
  greeting: 'Ball handler ready.',
  src: sprite_src_player,
  SCALE_FACTOR: 11,
  STEP_FACTOR: 800,
  ANIMATION_RATE: 110,
  INIT_POSITION: { ...this.playerStart },
  pixels: { height: 770, width: 513 },
  orientation: { rows: 4, columns: 4 },
  down:  { row: 0, start: 0, columns: 4 },
  left:  { row: 1, start: 0, columns: 4 },
  right: { row: 2, start: 0, columns: 4 },
  up:    { row: 3, start: 0, columns: 4 },
  hitbox: { widthPercentage: 0.45, heightPercentage: 0.5 },
  keypress: { up: 87, left: 65, down: 83, right: 68 }
};
```

- A fully data-driven configuration object describing everything the engine needs to render, animate, and control the player

---

<a id="mathematical-operators"></a>
### Mathematical Operators

**Project Evidence Required:** Physics calculations such as gravity, velocity, collision.  
**Assessment Method:** Code review of `+`, `-`, `*`, `/` in physics.

```js
const dx = player.position.x - lebron.position.x;
const dy = player.position.y - lebron.position.y;
const dist = Math.hypot(dx, dy);
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);
lebron.position.x += (dx / dist) * speed;
lebron.position.y += (dy / dist) * speed;
```

- Subtraction finds the direction vector, multiplication scales the speed over time, division normalizes the vector, and addition moves the position each frame

---

<a id="string-operations"></a>
### String Operations

**Project Evidence Required:** Path concatenation, text display.  
**Assessment Method:** Code review of template literals and concatenation.

```js
this.timeHud.textContent =
  `Time: ${this.currentTime.toFixed(1)}s/${this.targetSurvivalSeconds}s | Best: ${this.bestTime.toFixed(1)}s | ` +
  `Coins: ${this.getCoinsCollected()} | Best Coins: ${this.bestCoins}`;
```

- Template literals interpolate live numeric values into the HUD string, and `+` concatenates the two template strings together

```js
const basePath = (this.gameEnv?.path || '').replace(/\/$/, '');
const aquaticUrl = `${basePath}/games/aquatic.html`;
```

- Template literal concatenation builds navigation URLs from a dynamic base path

---

<a id="boolean-expressions"></a>
### Boolean Expressions

**Project Evidence Required:** Compound conditions in game logic.  
**Assessment Method:** Code review of `&&`, `||`, `!`.

```js
if (event.key.toLowerCase() !== 'e' || event.repeat) return;
if (this.preGameLocked || this.caught) return;
```

- `||` chains early-exit guards: the shot is blocked if the key is wrong, if it's a held repeat, if the game hasn't started, or if the player is already caught

```js
return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius);
```

- A boolean expression used as the return value of `isCircleHittingObject` — true only when the squared distance falls within the squared radius

---

<a id="keyboard-input"></a>
### Keyboard Input

**Project Evidence Required:** Arrow keys, space, WASD controls using event listeners.  
**Assessment Method:** Testing that key event handlers respond correctly.

```js
document.addEventListener('keydown', this.handleRestartKey);
document.addEventListener('keydown', this.handleShootKey);
```

```js
handleShootKey(event) {
  if (event.key.toLowerCase() !== 'e' || event.repeat) return;
  ...
}

handleRestartKey(event) {
  if (event.key.toLowerCase() !== 'r' || !this.caught) return;
  this.resetRound();
}
```

- `keydown` event listeners are registered on `initialize()` and cleaned up in `destroy()`; `E` fires a basketball projectile and `R` manually restarts the round after being caught

```js
keypress: { up: 87, left: 65, down: 83, right: 68 }
```

- WASD movement is configured in the player sprite data object and handled by the engine's `Player` class

---

<a id="canvas-rendering"></a>
### Canvas Rendering

**Project Evidence Required:** Draw sprites, backgrounds, platforms using Canvas API.  
**Assessment Method:** Code review of `draw()` method implementations.

```js
drawProjectileSprite(ctx, width, height) {
  const cx = width / 2;
  const cy = height / 2;
  const r = Math.min(width, height) * 0.42;

  ctx.beginPath();
  ctx.arc(cx, cy, r, 0, Math.PI * 2);
  ctx.fillStyle = '#f68b1f';
  ctx.fill();
  ctx.lineWidth = 4;
  ctx.strokeStyle = '#8a3d00';
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(cx - r, cy);
  ctx.quadraticCurveTo(cx, cy - 8, cx + r, cy);
  ctx.strokeStyle = '#8a3d00';
  ctx.lineWidth = 3;
  ctx.stroke();

  ctx.beginPath();
  ctx.moveTo(cx, cy - r);
  ctx.quadraticCurveTo(cx - 8, cy, cx, cy + r);
  ctx.stroke();
}
```

- Directly uses the Canvas 2D API to draw a basketball sprite: an orange filled circle with curved seam lines drawn using `arc` and `quadraticCurveTo`

---

<a id="api-integration"></a>
### API Integration

**Project Evidence Required:** Implement Leaderboard API with POST/GET scores.  
**Assessment Method:** Code review of fetch calls with error handling.

```js
initLeaderboard() {
  if (this.leaderboard) return;
  this.leaderboard = new Leaderboard(this.gameEnv.gameControl, {
    gameName: 'Basketball',
    initiallyHidden: false
  });
}

submitRoundScore() {
  if (!this.leaderboard || this.scoreSubmittedThisRound) return;
  const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50));
  const username = (this.gameEnv?.game?.uid && String(this.gameEnv.game.uid)) || 'Player';
  this.scoreSubmittedThisRound = true;

  this.leaderboard.submitScore(username, score, 'Basketball')
    .catch((err) => console.warn('Leaderboard score submit failed:', err));
}
```

- `Leaderboard` is initialized once and `submitScore()` is called at the end of each round; `.catch()` provides error handling for failed network requests
- Score is calculated from survival time and coins collected before being submitted

---

<a id="asynchronous-io"></a>
### Asynchronous I/O

**Project Evidence Required:** Use `async/await` or promises for API calls.  
**Assessment Method:** Code review of `async/await` or `.then()` chains.

```js
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));
```

- `submitScore` returns a Promise; `.catch()` is a promise chain handler that catches and logs any network or API failure without crashing the game loop

---

<a id="json-parsing"></a>
### JSON Parsing

**Project Evidence Required:** Parse API responses such as leaderboard data and AI responses.  
**Assessment Method:** Code review of `JSON.parse()`, object destructuring.

```js
import Leaderboard from '@assets/js/GameEnginev1.1/essentials/Leaderboard.js';
...
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));
```

- The `Leaderboard` class handles the fetch calls and parses the JSON responses from the backend internally
- `GameLevelBasketball` consumes the result by calling `submitScore()` and handling errors, while the actual `JSON.parse()` and object destructuring of the API response lives inside `Leaderboard.js`

---

<a id="code-comments"></a>
### Code Comments

**Project Evidence Required:** JSDoc comments for classes and methods (>10% comment density).  
**Assessment Method:** Code review of comment density.

```js
// Speed curve -> LeBron gets slightly faster over time but has a cap to keep the game fair
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);

// Clamp LeBron's position so he can't leave the visible game area
lebron.position.x = Math.max(0, Math.min(lebron.position.x, this.gameEnv.innerWidth - (lebron.width || 0)));

// Draws the main orange circle (the ball body)
ctx.beginPath();
ctx.arc(cx, cy, r, 0, Math.PI * 2);

// Draws the horizontal seam line
ctx.beginPath();
ctx.moveTo(cx - r, cy);

// Draws the vertical seam line
ctx.beginPath();
ctx.moveTo(cx, cy - r);

// Calculate the direction from LeBron to the player so LeBron can chase
const dx = player.position.x - lebron.position.x;

// Keep the DOM hitbox aligned with the new spawn immediately.
if (typeof coin.setupCanvas === 'function') {
  coin.setupCanvas();
}
```

- Inline comments explain the intent behind non-obvious logic throughout the file
- These describe physics behavior, clamping reasons, and canvas drawing steps so the code is readable without needing to trace execution

---

<a id="console-debugging"></a>
### Console Debugging

**Project Evidence Required:** Use `console.log` to track game state, variables, method calls.  
**Assessment Method:** Code review of strategic logging in update/collision methods.

```js
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));

try {
  window.dispatchEvent(new CustomEvent('characters:concept-focus', {
    detail: { level: 'basketball', trigger: 'first-steal' }
  }));
} catch (err) {
  console.warn('Failed to emit basketball concept focus event:', err);
}

try {
  window.dispatchEvent(new CustomEvent('characters:level-complete', {
    detail: { level: 'basketball' }
  }));
} catch (err) {
  console.warn('Failed to emit basketball completion event:', err);
}
```

- `console.warn` is used in error paths to surface API failures and custom event dispatch errors without crashing the game loop
- Each warning includes the error object so DevTools shows the full stack trace for debugging

---

<a id="hit-box-visualization"></a>
### Hit Box Visualization

**Project Evidence Required:** Draw/visualize collision boundaries to refine detection.  
**Assessment Method:** Demo — toggle hit box display, adjust collision rectangles.

```js
getHitboxRect(obj) {
   = obj.width  || 0;
  const height = obj.height || 0;
  const pos = obj.position || { x: 0, y: 0 };
  const widthReduction  = width  * 0.2;
  const heightReduction = height * 0.2;

  return {
    left:   pos.x + widthReduction,
    right:  pos.x + width - widthReduction,
    top:    pos.y + heightReduction,
    bottom: pos.y + height
  };
}

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

- `getHitboxRect` computes the shrunk collision boundary for any game object by reducing 20% from each side, which is what allows the collision feel to be tuned without changing the sprite size
- `isHitboxCollision` uses those computed rectangles to perform a standard AABB overlap check between two objects

---

<a id="gameplay-testing"></a>
### Gameplay Testing

**Project Evidence Required:** Test level completion, character interactions, collision detection.  
**Assessment Method:** Live demo — play through level without critical bugs.

```js
if (this.currentTime >= this.targetSurvivalSeconds) {
  this.completeLevel();
  return;
}
```

```js
if (this.isHitboxCollision(player, lebron)) {
  this.caught = true;
  this.caughtAt = now;
  ...
  this.showCaughtMessage();
}
```

```js
if (lebron && this.isCircleHittingObject(projectile, lebron)) {
  this.lebronStunUntil = Math.max(this.lebronStunUntil, now + this.lebronStunDurationMs);
  lebron.velocity.x = 0;
  lebron.velocity.y = 0;
  this.removeProjectileAt(i);
}
```

- Level completion triggers after surviving 20 seconds
- Character interaction is tested through the catch collision between the player and LeBron
- Projectile collision is tested through the stun mechanic when a basketball hits LeBron

---

<a id="integration-testing"></a>
### Integration Testing

**Project Evidence Required:** Test API integration (Leaderboard, NPC AI) with live backend.  
**Assessment Method:** Demo — successful score saving and AI responses.

```js
submitRoundScore() {
  if (!this.leaderboard || this.scoreSubmittedThisRound) return;
  const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50));
  const username = (this.gameEnv?.game?.uid && String(this.gameEnv.game.uid)) || 'Player';
  this.scoreSubmittedThisRound = true;

  this.leaderboard.submitScore(username, score, 'Basketball')
    .catch((err) => console.warn('Leaderboard score submit failed:', err));
}
```

- Score is calculated from survival time and coins collected then submitted to the Leaderboard API at the end of each round
- `scoreSubmittedThisRound` prevents duplicate submissions in the same round
- Demonstrate via the Network tab in DevTools showing a successful POST request and response

---

<a id="api-error-handling"></a>
### API Error Handling

**Project Evidence Required:** Try/catch blocks for API calls and network error handling.  
**Assessment Method:** Code review of error handling for fetch failures.

```js
try {
  window.dispatchEvent(new CustomEvent('characters:concept-focus', {
    detail: { level: 'basketball', trigger: 'first-steal' }
  }));
} catch (err) {
  console.warn('Failed to emit basketball concept focus event:', err);
}
```

```js
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));
```

- Both `try/catch` and `.catch()` promise error handling are present throughout the file
- Failures are logged as warnings so the game continues running even if the API or event system is unavailable

## Final CS 111 Alignment

`GameLevelBasketball` demonstrates the major CS 111 and CSSE objectives in a direct, playable way.

- It uses classes, objects, inheritance-based engine components, and constructor chaining.
- It uses numbers, strings, booleans, arrays, and object literals throughout the level.
- It uses conditionals, nested logic, and loops for real gameplay systems.
- It uses keyboard input, canvas rendering, DOM output, and `GameEnv` configuration.
- It uses API integration, asynchronous score submission, and local storage persistence.
- It shows state management, collision systems, debugging evidence, and testable game behavior.

What makes the file especially strong is that the concepts are not isolated practice exercises. They all support one playable level, so each programming concept connects to something the player can actually see and test.