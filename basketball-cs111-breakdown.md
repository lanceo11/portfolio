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

This page follows the CS 111 evidence table directly. For each concept below, I name the required project evidence, the assessment method, and then explain how the basketball project demonstrates it. Most proof comes from [`GameLevelBasketball.js`](/Users/lanceoberiano/lanceo11/portfolio/_projects/kirby-minigames/levels/GameLevelBasketball.js), and when the rubric explicitly asks for class-writing evidence, I also reference the supporting classes it instantiates: [`Player.js`](/Users/lanceoberiano/lanceo11/portfolio/assets/js/GameEnginev1.1/essentials/Player.js), [`Npc.js`](/Users/lanceoberiano/lanceo11/portfolio/assets/js/GameEnginev1.1/essentials/Npc.js), and [`Coin.js`](/Users/lanceoberiano/lanceo11/portfolio/assets/js/GameEnginev1.1/Coin.js).

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

```js
this.classes = [
  { class: Player, data: sprite_data_player },
  { class: Npc, data: sprite_data_chaser },
  { class: Coin, data: coin_1 }
];
```

- `GameLevelBasketball.js` proves these classes are actually used in a live level, not just defined somewhere else.
- `Player` and `Npc` both extend `Character`, and `Coin` extends `Npc`, so the project exceeds the minimum two-class requirement.
- The level file instantiates those classes through `this.classes`, which ties the class-writing evidence directly to gameplay.

<a id="methods-and-parameters"></a>
### Methods & Parameters

**Project Evidence Required:** Implement methods with parameters, such as `collisionHandler(other, direction)`.  
**Assessment Method:** Code review of method signatures with 2 or more parameters.

```js
updateProjectiles(now, lebron) { ... }
spawnProjectileFromPlayer(player, now) { ... }
isHitboxCollision(a, b) { ... }
drawProjectileSprite(ctx, width, height) { ... }
```

- Methods are functions that belong to the class.
- Parameters are the values passed into those methods.
- `now` gives the current time so cooldowns and lifetimes can be checked.
- `player`, `lebron`, `a`, and `b` let the methods work with different objects.
- This makes the code reusable instead of hardcoding everything for one exact case.
- This satisfies the rubric clearly because several methods take two or three parameters and use them for different gameplay situations.

<a id="single-responsibility"></a>
### Single Responsibility and Organization

```js
initialize() { ... }
update() { ... }
resetRound() { ... }
destroy() { ... }
createHud() { ... }
clearProjectiles() { ... }
```

- `initialize()` prepares the round.
- `update()` handles frame-by-frame gameplay.
- `resetRound()` restores the level after the player gets caught.
- `destroy()` cleans up DOM elements and listeners.
- Each method has one main job, which makes the file easier to debug and explain.

## Object-Oriented Programming and Classes

<a id="object-oriented-programming"></a>
### Object-Oriented Programming (OOP)

```js
class GameLevelBasketball {
  constructor(gameEnv) {
    this.gameEnv = gameEnv;
    ...
  }
}
```

- The class acts like the controller for the whole minigame.
- It owns data such as timers, flags, and projectile arrays.
- It also owns behavior such as updating, resetting, and rendering helpers.
- That is OOP because state and behavior live together inside one object.

<a id="instantiation-and-objects"></a>
### Instantiation & Objects

**Project Evidence Required:** Instantiate game objects in `GameLevel` configuration.  
**Assessment Method:** Code review of `GameLevel` setup objects.

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

- `this.classes` is a configuration array for the engine.
- Each item is an object literal that says what class to create and what data to use.
- The engine reads that configuration and instantiates real objects.
- This is a clean example of objects being created from structured data.

<a id="inheritance-basic"></a>
### Inheritance (Basic)

**Project Evidence Required:** Create a class hierarchy with 2 or more levels, such as `GameObject -> Character -> Player`.  
**Assessment Method:** Code review of the `extends` keyword and the inheritance chain.

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

- `Player`, `Npc`, and `Coin` are part of an inheritance chain.
- Child classes reuse behavior from parent classes.
- That keeps shared movement and object logic from being rewritten over and over.
- `GameLevelBasketball.js` demonstrates this requirement by importing and instantiating those inherited classes inside one level.

<a id="method-overriding"></a>
### Method Overriding

**Project Evidence Required:** Override parent methods such as `update()`, `draw()`, or `handleCollision()`.  
**Assessment Method:** Code review of polymorphic implementations.

```js
update() {
  super.update();
  ...
}
```

```js
draw() {
  if (!this.ctx) return;
  ...
  this.setupCanvas();
}
```

```js
handleCollisionReaction(other) {
  ...
  super.handleCollisionReaction(other);
}
```

- `Player.js` overrides `update()` and `handleCollisionReaction(other)`.
- `Npc.js` overrides `update()` to add patrol and interaction behavior.
- `Coin.js` overrides `update()` and `draw()` to add collection and custom rendering.
- `GameLevelBasketball.js` depends on those overridden methods when it creates the player, chaser, and coins.

<a id="constructor-chaining"></a>
### Constructor Chaining

**Project Evidence Required:** Use `super()` to chain constructors.  
**Assessment Method:** Code review of `super(data, gameEnv)` calls.

```js
constructor(data = null, gameEnv = null) {
  super(data, gameEnv);
}
```

```js
constructor(data = null, gameEnv = null) {
  super(coinData, gameEnv);
}
```

- Constructor chaining happens when a child constructor calls `super(...)`.
- `Player`, `Npc`, and `Coin` all do this so the parent classes initialize shared game-object behavior first.
- The basketball level relies on that constructor chain every time it instantiates one of those classes from `this.classes`.

## Data Types and Object Literals

<a id="numbers"></a>
### Numbers

**Project Evidence Required:** Use numeric data for position, velocity, timing, or score tracking.  
**Assessment Method:** Code review of numeric properties and calculations.

```js
this.projectileSpeed = 9;
this.projectileRadius = 10;
this.projectileLifeMs = 2200;
this.shootCooldownMs = 5000;
this.targetSurvivalSeconds = 20;
```

- Numbers store measurable values in the game.
- `9` controls how fast the projectile moves.
- `2200` controls how long a projectile lives in milliseconds.
- `5000` creates a 5-second cooldown between shots.
- `20` means the player wins after surviving 20 seconds.

<a id="strings"></a>
### Strings

**Project Evidence Required:** Use strings for character names, sprite paths, or game text.  
**Assessment Method:** Code review of string values and text output.

```js
id: 'BasketballPlayer',
greeting: 'Ball handler ready.',
id: 'LeBron',
dialogues: ['LeBron is in the gym.']
```

- Strings store text data.
- IDs help the program find specific objects later.
- Greetings and dialogue strings are shown to the player.
- Strings also appear in HUD text, API names, and browser storage keys.

<a id="booleans"></a>
### Booleans

**Project Evidence Required:** Use flags such as `isJumping`, `isPaused`, or `isVulnerable`.  
**Assessment Method:** Code review of boolean state and logic.

```js
this.caught = false;
this.preGameLocked = true;
this.levelCompleted = false;
this.completionTriggered = false;
```

- Booleans only hold `true` or `false`.
- They act like switches for game state.
- `caught` tracks whether the player has been tagged.
- `preGameLocked` blocks gameplay before the intro is finished.
- These flags make the game logic easier to read than using random numbers or text labels.

<a id="arrays"></a>
### Arrays

**Project Evidence Required:** Use arrays for game object collections or level data.  
**Assessment Method:** Code review of array operations.

```js
this.projectiles = [];
this.classes = [ ... ];
dialogues: ['LeBron is in the gym.']
```

- Arrays store ordered groups of values.
- `this.projectiles` holds every active basketball shot.
- `this.classes` holds every game object definition for the level.
- Arrays are useful because the game needs to loop through multiple similar items.

<a id="objects-json"></a>
### Objects (JSON)

**Project Evidence Required:** Use configuration objects and structured data.  
**Assessment Method:** Code review of object literals and nested properties.

```js
const coin_1 = {
  id: 'coin_1',
  INIT_POSITION: { x: Math.round(width * 0.25), y: Math.round(height * 0.35) },
  SCALE_FACTOR: 18,
  hitbox: coinHitbox,
  value: 1
};
```

- This is a JSON-style object literal.
- One object groups several related properties together.
- `INIT_POSITION` is itself a nested object with `x` and `y`.
- `value: 1` stores how many points the coin is worth.
- This is data-driven design because reusable engine classes read object data to decide behavior.

## Operators and Mathematical Reasoning

<a id="mathematical-operators"></a>
### Mathematical

**Project Evidence Required:** Use math for movement, collisions, or timing.  
**Assessment Method:** Code review of `+`, `-`, `*`, `/`, and math helpers in gameplay logic.

```js
const dx = player.position.x - lebron.position.x;
const dy = player.position.y - lebron.position.y;
const dist = Math.hypot(dx, dy);
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);

lebron.position.x += (dx / dist) * speed;
lebron.position.y += (dy / dist) * speed;
```

- `-` measures the horizontal and vertical gap between two characters.
- `Math.hypot(dx, dy)` finds the total distance.
- `/` normalizes the direction so movement stays smooth.
- `*` scales movement by speed.
- `+=` updates the enemy position every frame.

<a id="string-operations"></a>
### String Operations

**Project Evidence Required:** Use template literals or concatenation for text output.  
**Assessment Method:** Code review of string-building logic.

```js
this.timeHud.textContent =
  `Time: ${this.currentTime.toFixed(1)}s/${this.targetSurvivalSeconds}s | Best: ${this.bestTime.toFixed(1)}s | ` +
  `Coins: ${this.getCoinsCollected()} | Best Coins: ${this.bestCoins}`;
```

- Template literals combine plain text with live values.
- `${...}` injects JavaScript values into a string.
- `toFixed(1)` turns a number into cleaner HUD text.
- The player sees an updated message without manually rebuilding every part.

<a id="boolean-expressions"></a>
### Boolean Expressions

**Project Evidence Required:** Use compound conditions in game logic.  
**Assessment Method:** Code review of `&&`, `||`, and `!`.

```js
if (event.key.toLowerCase() !== 'e' || event.repeat) return;
if (this.preGameLocked || this.caught) return;
if (now - this.lastShotAt < this.shootCooldownMs) return;
```

- A boolean expression evaluates to `true` or `false`.
- `!==` checks whether the pressed key is not `e`.
- `||` means "if either condition is true."
- These checks block invalid input, repeated input, and cooldown abuse.
- This directly controls fairness and pacing in the game.

## Control Structures and State Management

<a id="conditionals"></a>
### Conditionals

**Project Evidence Required:** Implement collision detection and state transitions with `if/else`.  
**Assessment Method:** Code review of conditionals and nested decisions.

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

- `if` statements decide whether code should run.
- The method first checks whether the player and enemy exist.
- It pauses most gameplay if the intro sequence is still active.
- It only updates the timer if the player has not been caught.
- It ends the level when the survival goal is reached.

<a id="nested-conditions"></a>
### Nested Conditions

**Project Evidence Required:** Implement complex game logic with multi-level conditionals.  
**Assessment Method:** Code review of multi-step conditional reasoning.

- Nested conditions happen when one decision is inside another.
- In `update()`, the win check only happens inside the `if (!this.caught)` block.
- That means the level does not try to win after the player has already lost the round.
- This helps the logic happen in a safe and sensible order.

<a id="iteration"></a>
### Iteration

**Project Evidence Required:** Use loops for game object arrays or animation/game-state updates.  
**Assessment Method:** Code review of `for`, `forEach`, or `while` loops.

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

- The `for` loop repeats logic for every projectile.
- It starts at the end of the array and moves backward.
- That matters because the loop may remove projectiles while iterating.
- Going backward avoids index-shift bugs that can skip items.
- This is iteration solving a real programming problem.

<a id="state-management"></a>
### State Management

```js
this.caught = false;
this.preGameLocked = true;
this.scoreSubmittedThisRound = false;
this.levelCompleted = false;
this.completionTriggered = false;
```

- State is the saved information that tells the game what is happening right now.
- These flags control whether the round is active, locked, completed, or already submitted.
- The game loop reads those flags to decide what to do next.
- Clear state variables make the flow much easier to reason about.

## Input, Output, and Rendering

<a id="keyboard-input"></a>
### Keyboard Input

**Project Evidence Required:** Support keyboard controls with event listeners.  
**Assessment Method:** Testing that key handlers respond correctly.

```js
document.addEventListener('keydown', this.handleRestartKey);
document.addEventListener('keydown', this.handleShootKey);
```

```js
handleShootKey(event) {
  if (event.key.toLowerCase() !== 'e' || event.repeat) return;
  if (this.preGameLocked || this.caught) return;
  const now = performance.now();
  if (now - this.lastShotAt < this.shootCooldownMs) return;
  ...
}
```

- The browser listens for `keydown` events.
- The handler checks what key the player pressed.
- It validates the input before allowing a shot.
- It also checks state flags and cooldown timing.
- This is user input because the player's keyboard actions directly affect gameplay.

<a id="canvas-rendering"></a>
### Canvas Rendering

**Project Evidence Required:** Draw sprites, backgrounds, or projectiles with the Canvas API.  
**Assessment Method:** Code review of `draw()` logic and canvas method usage.

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

- `ctx` is the canvas drawing context.
- `ctx.arc(...)` creates the circular basketball shape.
- `ctx.fillStyle` chooses the color.
- `ctx.fill()` paints the shape on the screen.
- This is direct rendering because code creates a visible object without a sprite image.

<a id="gameenv-configuration"></a>
### GameEnv Configuration

**Project Evidence Required:** Use shared game settings such as canvas size, container, or game state.  
**Assessment Method:** Code review of `GameEnv`-based configuration and setup.

```js
const width = gameEnv.innerWidth;
const height = gameEnv.innerHeight;
this.gameEnv.stats.coinsCollected = 0;
const container = this.gameEnv.container || this.gameEnv.gameContainer;
```

- `gameEnv` provides shared environment information from the engine.
- `innerWidth` and `innerHeight` help place objects based on screen size.
- `stats` stores live values such as collected coins.
- `container` tells the level where HUD and projectile elements should be attached.
- This shows the level working inside the larger engine system.

## Collision Detection, Hitboxes, and Gameplay Logic

<a id="hit-box-visualization"></a>
### Hit Box Visualization

**Project Evidence Required:** Show or refine collision boundaries for better gameplay.  
**Assessment Method:** Demo or code review of hitbox calculations and adjustments.

```js
getHitboxRect(obj) {
  const width = obj.width || 0;
  const height = obj.height || 0;
  const pos = obj.position || { x: 0, y: 0 };
  const widthReduction = width * 0.2;
  const heightReduction = height * 0.2;

  return {
    left: pos.x + widthReduction,
    right: pos.x + width - widthReduction,
    top: pos.y + heightReduction,
    bottom: pos.y + height
  };
}
```

- A hitbox is the area that counts for collisions.
- The code reduces the hitbox so collisions feel fairer than using the full sprite size.
- `left`, `right`, `top`, and `bottom` define the rectangle boundaries.
- This makes collision areas easier to reason about during testing and debugging.

<a id="collision-detection"></a>
### Collision Detection

```js
isHitboxCollision(a, b) {
  const ar = this.getHitboxRect(a);
  const br = this.getHitboxRect(b);
  return (
    ar.left < br.right &&
    ar.right > br.left &&
    ar.top < br.bottom &&
    ar.bottom > br.top
  );
}
```

- The method computes one rectangle for each object.
- It checks whether those rectangles overlap horizontally and vertically.
- If both overlaps are true, the objects are colliding.
- This combines geometry with boolean logic in a very direct way.

<a id="physics-and-movement"></a>
### Physics and Movement

- The level updates positions over time instead of teleporting objects.
- Enemy movement uses vector math to move smoothly toward the player.
- Projectile movement uses velocity values like `vx` and `vy`.
- Repeated frame updates create the feeling of motion.

<a id="enemy-ai-chase-logic"></a>
### Enemy AI and Chase Logic

- Kirby reacts to the player's current position.
- The game calculates a direction vector from Kirby to Astro.
- That vector is scaled into movement every frame.
- This is simple AI because the enemy behavior responds to the player instead of moving randomly.

<a id="dynamic-difficulty-scaling"></a>
### Dynamic Difficulty Scaling

```js
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);
```

- The enemy starts at a base speed.
- The speed increases as the survival timer goes up.
- `Math.min(...)` sets a cap so the game stays fair.
- This makes the level harder over time without making it impossible.

<a id="projectile-system"></a>
### Projectile System

```js
isCircleHittingObject(projectile, obj) {
  const rect = this.getHitboxRect(obj);
  const nearestX = Math.max(rect.left, Math.min(projectile.x, rect.right));
  const nearestY = Math.max(rect.top, Math.min(projectile.y, rect.bottom));
  const dx = projectile.x - nearestX;
  const dy = projectile.y - nearestY;
  return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius);
}
```

- The basketball is treated like a circle.
- The enemy hitbox is treated like a rectangle.
- The code finds the nearest point on the rectangle to the circle center.
- It then checks whether that point falls inside the projectile radius.
- This is a second collision strategy that fits a different shape.

<a id="cooldown-system"></a>
### Cooldown System

- Cooldowns stop the player from spamming actions.
- The level stores the time of the last shot in `lastShotAt`.
- It compares the current time against that stored value.
- If not enough time has passed, the shot is blocked.
- This creates better balance and pacing.

## API Integration, Asynchronous I/O, and Persistence

<a id="application-debugging"></a>
### Application Debugging

**Project Evidence Required:** Inspect cookies, `localStorage`, or session state used by the game.  
**Assessment Method:** Demo in the browser Application tab.

```js
loadBestTime() {
  try {
    return Number(localStorage.getItem('basketball_best_time') || 0);
  } catch (_) {
    return 0;
  }
}
```

- `localStorage` lets the browser keep values after the page is refreshed.
- The game uses it for best-time and best-coin records.
- Those values can be checked in the browser Application tab.
- That makes this a good example of application-level debugging.

<a id="api-integration"></a>
### API Integration

**Project Evidence Required:** Implement leaderboard score saving through an API-backed system.  
**Assessment Method:** Code review of leaderboard submission flow and error handling.

```js
initLeaderboard() {
  if (this.leaderboard) return;
  this.leaderboard = new Leaderboard(this.gameEnv.gameControl, {
    gameName: 'Basketball',
    initiallyHidden: false
  });
}
```

- The leaderboard object connects the game to backend score handling.
- `gameName: 'Basketball'` identifies which game the submitted score belongs to.
- `GameLevelBasketball.js` does not call `fetch(...)` directly, but it does trigger the leaderboard's API workflow through `submitScore(...)`.
- This still counts as API integration because the level hands score data to a networking component.

<a id="asynchronous-io"></a>
### Asynchronous I/O

**Project Evidence Required:** Use promises, `async/await`, or equivalent async behavior for API calls.  
**Assessment Method:** Code review of promise chains or `async` code.

```js
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));
```

- The score request does not finish instantly.
- It returns a promise because it depends on outside work.
- `.catch(...)` handles failure without crashing the game.
- This is asynchronous I/O because the program is communicating with something external.

<a id="api-error-handling"></a>
### API Error Handling

**Project Evidence Required:** Protect API calls with guards and failure handling.  
**Assessment Method:** Code review of submission checks and error capture.

```js
if (!this.leaderboard || this.scoreSubmittedThisRound) return;
```

- The code checks whether the leaderboard exists before submitting.
- It also prevents duplicate submissions in the same round.
- The warning in `.catch(...)` gives useful context if the request fails.
- This is defensive programming around network behavior.

<a id="json-parsing"></a>
### JSON Parsing

**Project Evidence Required:** Parse or work with JSON-style API or configuration data.  
**Assessment Method:** Code review of nested property access, structured objects, or JSON handling.

```js
this.gameEnv.stats.coinsCollected = 0;
```

- The file reads and updates structured object data constantly.
- That includes nested properties like positions, hitboxes, stats, and config fields.
- Even without a dramatic `JSON.parse(...)` call, this still demonstrates working with JSON-style structured data.

<a id="local-storage-persistence"></a>
### Local Storage Persistence

```js
saveBestCoins() {
  try {
    localStorage.setItem('basketball_best_coins', String(this.bestCoins));
  } catch (_) {}
}
```

- Persistence means the data remains after the page reloads.
- `setItem(...)` stores the value in the browser.
- The game can load that value again later.
- This turns temporary results into saved progress/history.

## Documentation, Debugging, and Testing Evidence

<a id="code-comments"></a>
### Code Comments

**Project Evidence Required:** Include comments that explain non-trivial logic.  
**Assessment Method:** Code review of meaningful documentation in code.

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

- The comments explain why the code exists, not just what it does.
- The speed comment explains the fairness reason for the cap.
- The DOM comment explains why the hitbox setup happens immediately.
- That makes the comments useful for studying and debugging.

<a id="console-debugging"></a>
### Console Debugging

**Project Evidence Required:** Use console output to inspect state or failures.  
**Assessment Method:** Code review of strategic logging in gameplay or API code.

```js
console.warn('Failed to emit basketball concept focus event:', err);
console.warn('Leaderboard score submit failed:', err);
```

- `console.warn(...)` helps developers notice problems while the game still runs.
- The warning text gives context about what failed.
- This is useful during gameplay testing and API debugging.

<a id="source-level-debugging"></a>
### Source-Level Debugging

**Project Evidence Required:** Organize code so breakpoints and step-through debugging are practical.  
**Assessment Method:** Demo in DevTools Sources tab.

- The file is split into named methods such as `update()`, `updateProjectiles()`, and `submitRoundScore()`.
- That makes it easier to set breakpoints and inspect variables in DevTools.
- Smaller methods make cause-and-effect much easier to trace.

<a id="element-inspection"></a>
### Element Inspection

**Project Evidence Required:** Inspect DOM and canvas elements while the game runs.  
**Assessment Method:** Demo in the browser Elements tab.

```js
this.timeHud = document.createElement('div');
this.messageHud = document.createElement('div');
canvas: document.createElement('canvas')
```

- These HUD and canvas elements are added to the DOM while the game is running.
- That means they can be inspected in the browser Elements panel.
- You can verify text, styles, and placement visually.

<a id="gameplay-testing"></a>
### Gameplay Testing

**Project Evidence Required:** Test level completion, collision logic, and interactions in a live run.  
**Assessment Method:** Playthrough demo without critical bugs.

- The intro dialogue can be tested to make sure the level starts at the right time.
- The timer HUD can be tested by surviving and watching it update.
- Coin collection and respawn can be tested by moving around the arena.
- The stun projectile and cooldown can be tested by pressing `E`.
- Reset and win conditions can be tested through repeated playthroughs.

<a id="integration-testing"></a>
### Integration Testing

**Project Evidence Required:** Test whether gameplay systems, HUD, persistence, and leaderboard work together.  
**Assessment Method:** End-to-end demo of combined systems.

- Integration testing checks whether multiple systems work together.
- In this level, gameplay, HUD updates, local storage, and leaderboard submission all connect.
- A full round is a good test of that combined flow.

<a id="mini-lesson-documentation"></a>
### Mini-Lesson Documentation

**Project Evidence Required:** Create a comic or visual post with an embedded runtime game demo.  
**Assessment Method:** Portfolio review of the mini-lesson in the personal portfolio.

- This page itself works as mini-lesson documentation.
- It explains course concepts in plain language next to real code.
- That makes it useful for learning, not just for grading evidence.

<a id="code-highlights"></a>
### Code Highlights

**Project Evidence Required:** Annotate key code snippets for OOP, APIs, collision, and gameplay logic.  
**Assessment Method:** Portfolio review of highlighted code examples with explanations.

- The page uses short code snippets instead of dumping the entire file.
- Each snippet highlights one idea such as arrays, collisions, or async behavior.
- That makes the project easier to study section by section.

<a id="network-debugging"></a>
### Network Debugging

**Project Evidence Required:** Inspect API requests, response data, and failures.  
**Assessment Method:** Demo in the browser Network tab.

- The leaderboard request can be inspected in the browser Network tab.
- You can verify whether the score request was sent and whether it succeeded.
- That helps debug API communication problems.

## Final CS 111 Alignment

`GameLevelBasketball` demonstrates the major CS 111 and CSSE objectives in a direct, playable way.

- It uses classes, objects, inheritance-based engine components, and constructor chaining.
- It uses numbers, strings, booleans, arrays, and object literals throughout the level.
- It uses conditionals, nested logic, and loops for real gameplay systems.
- It uses keyboard input, canvas rendering, DOM output, and `GameEnv` configuration.
- It uses API integration, asynchronous score submission, and local storage persistence.
- It shows state management, collision systems, debugging evidence, and testable game behavior.

What makes the file especially strong is that the concepts are not isolated practice exercises. They all support one playable level, so each programming concept connects to something the player can actually see and test.
