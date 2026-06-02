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

This page follows the CS 111 evidence table directly. Code snippets are pulled from `GameLevelBasketball.js` and the engine files it depends on — `Player.js`, `Npc.js`, `Character.js`, and `GameLevel.js`. Where evidence lives in an engine file, the connection back to the basketball level is shown explicitly.

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
---

<a id="writing-classes"></a>
### Writing Classes

**Project Evidence Required:** Create a minimum of 2 custom character classes extending base classes.  
**Assessment Method:** Code review of `Player.js`, `NPC.js`, `Enemy.js`-style class definitions.

A **class** is a blueprint for creating objects — it defines their properties and behavior. `extends` sets up an inheritance chain so one class can reuse another's code. `this.classes` tells the engine which objects to create at startup.

```js
// From Player.js — first custom character class
class Player extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv); // parent setup
    this.id = data?.id ? data.id.toLowerCase() : `player${Player.playerCount}`; // id tag
    this.keypress = data?.keypress || { up: 87, left: 65, down: 83, right: 68 }; // WASD
    this.pressedKeys = {}; // key state
    this._boundHandleKeyDown = this.handleKeyDown.bind(this); // keep this
    this._boundHandleKeyUp = this.handleKeyUp.bind(this); // keep this
    this.bindMovementKeyListners(); // listen keys
    this.gravity = data.GRAVITY || false; // gravity flag
    this.acceleration = 0.001; // fall speed
    this.time = 0; // timer
    this.moved = false; // moved flag
  }
}
```

- `extends Character` means `Player` automatically inherits `Character`'s canvas setup, velocity, sprite drawing, and collision detection without rewriting any of it
- `Player` adds keyboard input, gravity, and touch controls on top of what it inherits

```js
// From Npc.js — second custom character class
class Npc extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);
    this.interact = data?.interact;
    this.currentQuestionIndex = 0;
    this.isInteracting = false;
    this.walkingArea = data?.walkingArea || null;
    this.speed = data?.speed || 1;
    this.moveDirection = data?.moveDirection || { x: 1, y: 1 };
    this.dialogueSystem = new DialogueSystem({
      dialogues: data.dialogues,
      id: this.uniqueId
    });
  }
}
```

- `Npc extends Character` is the second custom class — it inherits the same sprite and canvas system as `Player` but adds patrol movement, dialogue, and interaction handling instead of keyboard input
- Both `Player` and `Npc` sit at the end of a three-level chain: `GameObject → Character → Player` and `GameObject → Character → Npc`

```js
// From GameLevelBasketball.js — both classes are used here
{ class: Player, data: sprite_data_player },
{ class: Npc, data: sprite_data_chaser },
```

- `GameLevelBasketball` orchestrates both classes by placing them in `this.classes` — the engine reads this array and calls `new Player(...)` and `new Npc(...)` to bring them into the level

---

<a id="methods-and-parameters"></a>
### Methods & Parameters

**Project Evidence Required:** Implement methods with parameters, such as `collisionHandler(other, direction)`.  
**Assessment Method:** Code review of method signatures with 2 or more parameters.

A **method** is a function that belongs to a class; it uses `this` to access the instance's own data. **Parameters** are the named inputs declared in parentheses — they let one method body handle many different callers without duplicating code.

```js
isCircleHittingObject(projectile, obj) {
  const rect = this.getHitboxRect(obj); // target bounds
  const nearestX = Math.max(rect.left, Math.min(projectile.x, rect.right)); // clamp x
  const nearestY = Math.max(rect.top,  Math.min(projectile.y, rect.bottom)); // clamp y
  const dx = projectile.x - nearestX; // x gap
  const dy = projectile.y - nearestY; // y gap
  return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius); // hit?
}
```

- Takes two parameters: the `projectile` (a circle) and the target `obj` (a rectangle)
- Finds the nearest point on the rectangle to the circle's center, then checks if the distance falls within the projectile's radius — if it does, that's a hit
- This is the exact check that decides whether a basketball actually hits LeBron's hitbox

```js
updateProjectiles(now, lebron) {
  for (let i = this.projectiles.length - 1; i >= 0; i -= 1) {
    const projectile = this.projectiles[i]; // current shot
    projectile.x += projectile.vx; // move x
    projectile.y += projectile.vy; // move y
    ...
  }
}
```

- Takes the current timestamp `now` and the `lebron` game object as parameters
- Uses `now` to expire old projectiles based on their age, and checks `lebron` for hit detection each frame
- This keeps every shot moving, removes old shots, and freezes LeBron when a shot connects

```js
drawProjectileSprite(ctx, width, height) {
  const cx = width / 2; // center x
  const cy = height / 2; // center y
  const r = Math.min(width, height) * 0.42; // ball size
  ctx.beginPath(); // start circle
  ctx.arc(cx, cy, r, 0, Math.PI * 2); // draw ball
  ctx.fillStyle = '#f68b1f';
  ctx.fill();
}
```

- Takes three parameters: the canvas context `ctx`, and the `width` and `height` of the drawing area
- Uses all three to calculate the center point and radius before drawing the basketball sprite onto its own canvas element
- This is what makes each projectile look like a basketball instead of a plain dot

**Text Runner: AABB Hit Box Check**

Run this small JavaScript snippet to verify the exact bounding-box overlap math without booting the full game canvas.

{% capture hitbox_runner_code %}
const getHitboxRect = (obj) => {
  const width = obj.width || 0;
  const height = obj.height || 0;
  const position = obj.position || { x: 0, y: 0 };
  const widthReduction = width * 0.2;
  const heightReduction = height * 0.2;

  return {
    left: position.x + widthReduction,
    right: position.x + width - widthReduction,
    top: position.y + heightReduction,
    bottom: position.y + height - heightReduction
  };
};

const isHitboxCollision = (a, b) => {
  const ar = getHitboxRect(a);
  const br = getHitboxRect(b);
  return (
    ar.left < br.right &&
    ar.right > br.left &&
    ar.top < br.bottom &&
    ar.bottom > br.top
  );
};

const player = { id: 'BasketballPlayer', position: { x: 120, y: 80 }, width: 100, height: 160 };
const lebron = { id: 'LeBron', position: { x: 175, y: 130 }, width: 90, height: 90 };
const miss = { id: 'Open Space', position: { x: 320, y: 240 }, width: 90, height: 90 };

console.log('Player hitbox:', JSON.stringify(getHitboxRect(player)));
console.log('LeBron hitbox:', JSON.stringify(getHitboxRect(lebron)));
console.log('Overlap check:', isHitboxCollision(player, lebron));
console.log('Far miss:', isHitboxCollision(player, miss));
{% endcapture %}
{% include runners/code.html runner_id="basketball-hitbox-check" language="javascript" code=hitbox_runner_code %}

---

<a id="instantiation-and-objects"></a>
### Instantiation & Objects

**Project Evidence Required:** Instantiate game objects in GameLevel configuration.  
**Assessment Method:** Code review of GameLevel setup objects.

**Instantiation** means calling `new ClassName()` to create an independent object from a class blueprint. Each object owns its own copy of the class's properties — changing one coin's position doesn't affect any other. The engine does this inside `GameLevel.js` by reading `this.classes` from the level and calling `new` on each entry.

```js
// From GameLevel.js — this is where the engine actually instantiates every game object
for (let descriptor of this.gameObjectClasses) {
  const expandedDescriptors = this.expandDescriptor(descriptor)
  for (let gameObjectClass of expandedDescriptors) {
    if (!gameObjectClass.data) gameObjectClass.data = {}
    let gameObject = new gameObjectClass.class(gameObjectClass.data, this.gameEnv)
    this.gameEnv.gameObjects.push(gameObject)
  }
}
```

- `new gameObjectClass.class(gameObjectClass.data, this.gameEnv)` is where instantiation happens — the engine reads each entry in `this.classes`, calls `new` with the data object and game environment, and pushes the result into `gameEnv.gameObjects`
- This is why `GameLevelBasketball` defines `this.classes` the way it does — every entry in that array becomes one `new` call here

```js
// From GameLevelBasketball.js — the level's class manifest that GameLevel.js reads
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

- The same `Coin` class is instantiated three separate times with different position data, producing three independent coin objects — `new` is what makes each one its own independent instance

---

<a id="inheritance"></a>
### Inheritance (Basic)

**Project Evidence Required:** Create a class hierarchy with 2+ levels.  
**Assessment Method:** Code review of `extends` keyword and inheritance chain.

**Inheritance** lets a child class automatically get all the properties and methods of its parent without rewriting them. The chain here is `GameObject → Character → Player` and `GameObject → Character → Npc`. Each level adds only what it specifically needs.

```js
// From Character.js — middle of the chain
// Character extends GameObject, adding sprite rendering and physics
class Character extends GameObject {
  constructor(data = null, gameEnv = null) {
    super(gameEnv); // calls GameObject's constructor
    this.canvas = document.createElement("canvas");
    this.canvas.id = data.id || "default";
    this.velocity = { x: 0, y: 0 };
    this.position = { ...data.INIT_POSITION };
    this.spriteData = data;
    this.spriteSheet = new Image();
    this.spriteSheet.src = data.src;
  }
}
```

- `Character extends GameObject` is the middle level of the chain — it adds the canvas element, velocity, position, and sprite sheet loading that every game character needs
- `GameObject` at the root handles engine registration; `Character` adds rendering; `Player` and `Npc` add behavior on top

```js
// From Player.js — leaf of the chain
// Player extends Character, adding keyboard input on top
class Player extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv); // calls Character's constructor
    this.keypress = data?.keypress || { up: 87, left: 65, down: 83, right: 68 };
    this.pressedKeys = {};
    this.bindMovementKeyListners();
  }
}

// From Npc.js — separate leaf of the chain
// Npc extends Character, adding patrol and dialogue on top
class Npc extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv); // calls Character's constructor
    this.walkingArea = data?.walkingArea || null;
    this.speed = data?.speed || 1;
    this.dialogueSystem = new DialogueSystem({ dialogues: data.dialogues });
  }
}
```

- Both `Player` and `Npc` use `extends Character` — the `extends` keyword is what causes them to automatically inherit `Character`'s `draw()`, `move()`, `resize()`, and `collisionChecks()` without rewriting them
- The full chain runs three levels deep: `GameObject` (engine registration) → `Character` (sprite and physics) → `Player` or `Npc` (behavior)

---

<a id="method-overriding"></a>
### Method Overriding

**Project Evidence Required:** Override parent methods such as `update()`, `draw()`, `handleCollision()`.  
**Assessment Method:** Code review of polymorphic implementations.

**Overriding** means a child class defines a method with the same name as a parent method, replacing its behavior. Calling `super.methodName()` runs the parent's version first, then the child adds its own logic on top — so you extend rather than replace.

```js
// From Player.js — Player overrides update() from Character
// super.update() runs Character's version first (animation stepping and draw)
// then Player adds its own gravity logic on top
update() {
  super.update(); // ← calls Character's update(): runs sprite animation and draw()
  // Player-specific logic added after:
  if (!this.moved) {
    if (this.gravity) {
      this.time += 1;
      this.velocity.y += 0.5 + this.acceleration * this.time;
    }
  } else {
    this.time = 0;
  }
}
```

- `super.update()` runs `Character`'s version first so the sprite animates and draws correctly, then `Player` adds gravity accumulation on top — the parent behavior is preserved and extended, not replaced

```js
// From Player.js — Player completely overrides handleCollisionReaction()
// no super call here — Player replaces the parent behavior entirely
handleCollisionReaction(other) {
  try {
    const touchPoints = this.collisionData?.touchPoints?.this;
    if (touchPoints) {
      if (touchPoints.left || touchPoints.right) {
        this.velocity.x = 0;
      }
      if (touchPoints.top || touchPoints.bottom) {
        this.velocity.y = 0;
      }
    }
  } catch (_) {}
  super.handleCollisionReaction(other);
}
```

- `handleCollisionReaction` zeroes velocity along whichever axis was touched before calling `super.handleCollisionReaction(other)` — the child handles the physics correction first, then defers the rest to the parent

```js
// From GameLevelBasketball.js — the level overrides randomizePosition on coins at runtime
coin.randomizePosition = () => {
  coin.position.x = xMin + Math.random() * Math.max(1, xMax - xMin);
  coin.position.y = yMin + Math.random() * Math.max(1, yMax - yMin);
  if (typeof coin.setupCanvas === 'function') {
    coin.setupCanvas();
  }
};
```

- This dynamically overrides `randomizePosition` on each coin object at runtime, replacing the default behavior with one constrained to the basketball court's play area boundaries

---

a id="constructor-chaining"></a>
### Constructor Chaining

**Project Evidence Required:** Use `super()` to chain constructors.  
**Assessment Method:** Code review of `super(data, gameEnv)` calls.

`super()` inside a constructor calls the parent class's constructor, passing along any arguments it needs. JavaScript requires `super()` before you can use `this` in a child constructor — without it the engine throws a `ReferenceError`. Every level of the hierarchy initializes itself in order before the child adds its own properties.

```js
// Chain visualized top to bottom: Player → Character → GameObject

// From Player.js — child constructor; must call super() before touching 'this'
class Player extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv); // ① calls Character's constructor with data and gameEnv
    // ② only runs after Character (and GameObject above it) fully complete:
    this.keypress = data?.keypress || { up: 87, left: 65, down: 83, right: 68 };
    this.pressedKeys = {};
    this.gravity = data.GRAVITY || false;
  }
}

// From Character.js — middle constructor; forwards gameEnv up to GameObject
class Character extends GameObject {
  constructor(data = null, gameEnv = null) {
    super(gameEnv); // ① calls GameObject's constructor — sets up engine registration
    // ② Character's own setup runs after GameObject completes:
    this.canvas = document.createElement("canvas");
    this.velocity = { x: 0, y: 0 };
    this.position = { ...data.INIT_POSITION };
  }
}
```

- The chain runs in order: `Player` calls `super(data, gameEnv)` → `Character` runs and calls `super(gameEnv)` → `GameObject` runs and registers the object with the engine
- No level can use `this` until its own `super()` completes — that is why `GameObject` finishes first, then `Character`, then `Player` adds its own properties last

```js
// From GameLevelBasketball.js — the data object that travels through the chain
const sprite_data_player = {
  id: 'BasketballPlayer',
  INIT_POSITION: { ...this.playerStart },
  pixels: { height: 770, width: 513 },
  orientation: { rows: 4, columns: 4 },
  hitbox: { widthPercentage: 0.45, heightPercentage: 0.5 },
  keypress: { up: 87, left: 65, down: 83, right: 68 }
};
```

- `sprite_data_player` is what gets passed as `data` all the way through the chain — `keypress` is used by `Player`, `pixels` and `INIT_POSITION` are used by `Character`, and `gameEnv` is used by `GameObject`, all from this one object literal defined in `GameLevelBasketball`

```js
// Npc uses the same chain pattern — Npc → Character → GameObject
class Npc extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv); // ① chains up through Character → GameObject
    // ② Npc-specific properties only after chain completes:
    this.walkingArea = data?.walkingArea || null;
    this.speed = data?.speed || 1;
    this.isInteracting = false;
  }
}
```

- Both `Player` and `Npc` follow the identical chaining pattern — `super(data, gameEnv)` is the first line in both constructors, triggering the full parent chain before either child sets a single property

---

<a id="conditionals"></a>
### Conditionals

**Project Evidence Required:** Implement collision detection, state transitions.  
**Assessment Method:** Code review of `if/else`, nested conditions.

An `if/else` **conditional** picks one of two execution paths based on a boolean test. In the basketball level, every major game state — running, caught, stunned, complete — is controlled by a chain of conditionals that check flags and timestamps each frame.

```js
if (this.isHitboxCollision(player, lebron)) {
  this.caught = true;
  this.caughtAt = now;
  this.bestTime = Math.max(this.bestTime, this.currentTime);
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

- Once caught, the game waits the reset delay before calling `resetRound()` and early-returns to skip all other update logic — one boolean flag gates an entire branch of the game loop

---

<a id="nested-conditions"></a>
### Nested Conditions

**Project Evidence Required:** Complex game logic combining multiple conditions.  
**Assessment Method:** Code review of multi-level conditionals.

**Nested conditions** layer multiple independent checks — the outer test must pass before the inner test even runs. Each level enforces a real game rule: is the game active, is the projectile in bounds, is it actually hitting LeBron. Keeping them nested rather than flat with `&&` makes each rule easy to read and modify independently.

```js
// Level 1: skip all projectile logic while the game is locked or the round is over
if (!this.preGameLocked && !this.caught) {
  for (let i = this.projectiles.length - 1; i >= 0; i -= 1) {
    const projectile = this.projectiles[i]; // current shot

    // Level 2: is the projectile still alive and in bounds?
    if (this.isProjectileOutOfBounds(projectile) || now - projectile.bornAt > this.projectileLifeMs) {
      this.removeProjectileAt(i);
      continue;
    }

    // Level 3: is it hitting LeBron specifically?
    if (lebron && this.isCircleHittingObject(projectile, lebron)) {
      this.lebronStunUntil = Math.max(this.lebronStunUntil, now + this.lebronStunDurationMs); // set stun
      lebron.velocity.x = 0; // stop x
      lebron.velocity.y = 0; // stop y
      this.removeProjectileAt(i); // clear shot
    }
  }
}
```

- Level 1 gates the entire projectile system behind the game state flags — no projectile logic runs until the intro is dismissed and the round is active
- Level 2 checks bounds and lifetime before doing anything else — expired or off-screen projectiles are removed immediately without checking collision
- Level 3 only runs when a live, in-bounds projectile is also overlapping LeBron's hitbox — all three conditions must pass in sequence
- This is the projectile-to-sprite hit check, so it answers the "does the ball actually touch LeBron?" question

```js
if (now < this.lebronStunUntil) {
  lebron.velocity.x = 0; // hold still
  lebron.velocity.y = 0; // hold still
  return; // skip chase
}
```

- A separate nested guard inside `update()` — only reached after `preGameLocked` and `caught` checks both pass, then halts LeBron's chase logic for the full stun duration
- This is what makes the basketball stun work by pausing LeBron for a few seconds

---

<a id="numbers"></a>
### Numbers

**Project Evidence Required:** Position, velocity, score tracking.  
**Assessment Method:** Code review of numeric properties.

JavaScript has one number type covering both integers and floats. **Integers** count discrete things like coins collected. **Floats** power physics: positions, velocities, and timestamps update in sub-pixel or sub-millisecond increments each frame for smooth motion.

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
- `lastShotAt = -Infinity` ensures the first shot is always allowed — any real timestamp subtracted from negative infinity will be greater than the cooldown
- These numbers tune the difficulty and pacing of the whole basketball round

```js
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);
lebron.position.x += (dx / dist) * speed;
lebron.position.y += (dy / dist) * speed;
```

- Speed scales up gradually over time but is capped at 2.8 so LeBron never becomes impossible to outrun — floats power both the scaling formula and the per-frame position update

---

<a id="strings"></a>
### Strings

**Project Evidence Required:** Character names, sprite paths, game states.  
**Assessment Method:** Code review of string manipulation.

A **string** is a sequence of characters in quotes. Strings are used here for asset IDs, file paths, and filtering game objects by name — the same dot notation accesses them whether they come from a local object or an API response.

```js
const sprite_src_player = getKirbyImageUrl('astro.png');
const sprite_src_chaser = getKirbyImageUrl('kirby.png');
```

- String filenames are passed to `getKirbyImageUrl()` to build full asset paths for each sprite — the function returns a complete URL string that the engine uses to load the image
- These strings point the game at the exact character art it needs to render

```js
const coins = this.gameEnv.gameObjects.filter(
  (obj) => String(obj?.spriteData?.id || '').startsWith('coin_')
);
```

- Uses string methods `String()` and `.startsWith()` to filter the entire game object list down to only coin instances by their ID prefix — no separate coin registry needed

---

<a id="booleans"></a>
### Booleans

**Project Evidence Required:** Flags such as `isJumping`, `isPaused`, `isVulnerable`.  
**Assessment Method:** Code review of boolean logic.

A **boolean** is either `true` or `false` — a single-bit decision flag. Booleans guard state transitions and prevent a single event (one catch, one score submit) from triggering twice in the same round.

```js
this.caught = false;
this.preGameLocked = true;
this.scoreSubmittedThisRound = false;
this.levelCompleted = false;
this.completionTriggered = false;
this.firstStealScrollTriggered = false;
```

- Six boolean flags gate every major state in the level: whether the intro has been dismissed, whether the player was caught, whether the score was already saved, and whether the completion event already fired
- These true/false switches stop the same event from firing twice and keep the round state clean

```js
if (this.preGameLocked) return;
```

- A single boolean check blocks all update logic until the player clicks Start — one flag freezes LeBron, the timer, and all collision detection simultaneously

---

<a id="arrays"></a>
### Arrays

**Project Evidence Required:** Game object collections, level data.  
**Assessment Method:** Code review of array operations.

An **array** is an ordered list that can hold any number of values. The game stores every live projectile in an array that grows via `push()` as shots are fired and shrinks via `splice()` as they expire — the loop never needs to know how many there are in advance.

```js
this.projectiles = [];
...
this.projectiles.push(projectile);
...
this.projectiles.splice(index, 1);
```

- `this.projectiles` is a live array of active basketball projectiles; objects are pushed in when fired and spliced out when they expire or hit LeBron
- The array lets the game track every shot without needing one variable per projectile

```js
this.classes = [
  { class: GameEnvBackground, data: image_data_court },
  { class: Player, data: sprite_data_player },
  { class: Npc, data: sprite_data_chaser },
  { class: Coin, data: coin_1 },
  { class: Coin, data: coin_2 },
  { class: Coin, data: coin_3 },
  ...
];
```

- The `this.classes` array is the level's full data manifest — the GameBuilder iterates it and calls `new class(data, gameEnv)` for each entry

---

<a id="objects-json"></a>
### Objects (JSON)

**Project Evidence Required:** Configuration objects, sprite data.  
**Assessment Method:** Code review of object literals.

An **object literal** `{ key: value }` groups related values under one name. JSON uses the same syntax — objects in code and API responses have identical structure and access patterns.

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

- A fully data-driven configuration object — nested objects inside it (`pixels`, `orientation`, `hitbox`, `keypress`) are accessed with the same dot notation whether they come from local config or a parsed API response

---

<a id="mathematical-operators"></a>
### Mathematical Operators

**Project Evidence Required:** Physics calculations such as gravity, velocity, collision.  
**Assessment Method:** Code review of `+`, `-`, `*`, `/` in physics.

Math operators power all physics. `-` finds direction vectors. `/` normalizes them. `*` scales speed. `+` updates position. `Math.hypot` computes straight-line distance without manually squaring and square-rooting.

```js
const dx = player.position.x - lebron.position.x;
const dy = player.position.y - lebron.position.y;
const dist = Math.hypot(dx, dy);
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);
lebron.position.x += (dx / dist) * speed;
lebron.position.y += (dy / dist) * speed;
```

- Subtraction finds the direction vector from LeBron to the player, `Math.hypot` computes the distance, division normalizes the vector to length 1, multiplication scales it by speed, and addition moves the position each frame
- This math is what makes LeBron chase the player smoothly instead of teleporting

**Text Runner: Vector Chase Math**

Run this JavaScript snippet to see the exact frame-by-frame vector adjustments used for LeBron's chase logic.

{% capture vector_runner_code %}
const frames = [
  { frame: 1, player: { x: 420, y: 240 }, currentTime: 0 },
  { frame: 2, player: { x: 428, y: 236 }, currentTime: 5 },
  { frame: 3, player: { x: 436, y: 232 }, currentTime: 10 }
];

let lebron = { x: 120, y: 100 };

for (const state of frames) {
  const dx = state.player.x - lebron.x;
  const dy = state.player.y - lebron.y;
  const dist = Math.hypot(dx, dy);
  const speed = Math.min(2.1 + state.currentTime * 0.03, 2.8);
  const vx = (dx / dist) * speed;
  const vy = (dy / dist) * speed;
  const nextX = lebron.x + vx;
  const nextY = lebron.y + vy;

  console.log(
    `Frame ${state.frame}: dx=${dx.toFixed(2)}, dy=${dy.toFixed(2)}, dist=${dist.toFixed(2)}, ` +
    `speed=${speed.toFixed(2)}, vx=${vx.toFixed(2)}, vy=${vy.toFixed(2)}, next=(${nextX.toFixed(2)}, ${nextY.toFixed(2)})`
  );

  lebron = { x: nextX, y: nextY };
}
{% endcapture %}
{% include runners/code.html runner_id="basketball-vector-check" language="javascript" code=vector_runner_code %}

```js
const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50));
```

- Score is computed from two weighted components: survival time worth 10 points per second, and coins worth 50 points each — `Math.round` converts the float result to a clean integer

---

<a id="string-operations"></a>
### String Operations

**Project Evidence Required:** Path concatenation, text display.  
**Assessment Method:** Code review of template literals and concatenation.

**Template literals** (backtick strings with `${}`) replace concatenation for building display strings. They embed any live JavaScript expression inline, so the HUD updates in one readable line instead of several joined strings.

```js
this.timeHud.textContent =
  `Time: ${this.currentTime.toFixed(1)}s/${this.targetSurvivalSeconds}s | Best: ${this.bestTime.toFixed(1)}s | ` +
  `Coins: ${this.getCoinsCollected()} | Best Coins: ${this.bestCoins}`;
```

- Template literals interpolate live numeric values into the HUD string each frame, and `+` concatenates the two template strings — `.toFixed(1)` formats floats to one decimal place for clean display

```js
const basePath = (this.gameEnv?.path || '').replace(/\/$/, '');
const aquaticUrl = `${basePath}/games/aquatic.html`;
```

- Template literal concatenation builds navigation URLs from a dynamic base path — `replace(/\/$/, '')` strips a trailing slash before appending the route

---

<a id="boolean-expressions"></a>
### Boolean Expressions

**Project Evidence Required:** Compound conditions in game logic.  
**Assessment Method:** Code review of `&&`, `||`, `!`.

**Boolean operators**: `||` (OR) is true if either side is true; `&&` (AND) requires both sides. Short-circuit evaluation means `||` stops as soon as it finds a truthy value — used here as a safe default to avoid null errors.

```js
if (event.key.toLowerCase() !== 'e' || event.repeat) return;
if (this.preGameLocked || this.caught) return;
```

- `||` chains early-exit guards: the shot is blocked if the key is wrong, if it's a held repeat keydown, if the game hasn't started, or if the player is already caught — any one of these alone is enough to block the shot
- These checks protect the input so one key press only creates one clean shot

```js
return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius);
```

- A boolean expression used directly as the return value of `isCircleHittingObject` — true only when the squared distance between circle center and nearest rectangle point falls within the squared radius, avoiding a slow `Math.sqrt` call
- This returns a simple yes/no answer for whether the basketball hit the target

---

<a id="keyboard-input"></a>
### Keyboard Input

**Project Evidence Required:** Arrow keys, space, WASD controls using event listeners.  
**Assessment Method:** Testing that key event handlers respond correctly.

An **event listener** registers a callback function that runs whenever a named browser event fires. Because the game loop can't poll the keyboard directly, listeners write input state at the moment keys are pressed, and the loop reads that state every frame.

```js
document.addEventListener('keydown', this.handleRestartKey);
document.addEventListener('keydown', this.handleShootKey);
```

- Both listeners are registered in `initialize()` and removed in `destroy()` — binding them to `this` in the constructor ensures they reference the correct instance when called

```js
handleShootKey(event) {
  if (event.key.toLowerCase() !== 'e' || event.repeat) return;
  if (this.preGameLocked || this.caught) return;
  const now = performance.now();
  if (now - this.lastShotAt < this.shootCooldownMs) return;
  ...
}

handleRestartKey(event) {
  if (event.key.toLowerCase() !== 'r' || !this.caught) return;
  this.resetRound();
}
```

- `E` fires a basketball projectile in the player's facing direction with a 5-second cooldown; `R` manually restarts the round after being caught

```js
keypress: { up: 87, left: 65, down: 83, right: 68 }
```

- WASD movement (W=87, A=65, S=83, D=68) is configured directly in the player sprite data object and handled by the engine's `Player` class through its own inherited input system

---

<a id="canvas-rendering"></a>
### Canvas Rendering

**Project Evidence Required:** Draw sprites, backgrounds, platforms using Canvas API.  
**Assessment Method:** Code review of `draw()` method implementations.

The **Canvas 2D API** is a stateful drawing surface — `ctx` is the 2D rendering context. Every projectile is drawn onto its own offscreen canvas using `arc`, `fill`, and `quadraticCurveTo` to produce a recognizable basketball with seam lines.

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

- `arc()` draws the main orange ball body; `quadraticCurveTo()` draws curved seam lines by bending a path between two points through a control point
- Each projectile canvas is positioned absolutely in the DOM and updated every frame through its `style.left` and `style.top` properties
- This is the visual piece that makes the shot look like a basketball moving across the court

---

<a id="gameenv-configuration"></a>
### GameEnv Configuration

**Project Evidence Required:** Set canvas size, difficulty levels, game settings.  
**Assessment Method:** Code review of `GameEnv.create()` and `GameSetup.js`.

The **level constructor** is the engine's single configuration hook. Reading dimensions from `gameEnv` instead of hardcoding them makes the level adapt to any canvas size. `this.classes` tells the engine which base objects to instantiate on startup.

```js
constructor(gameEnv) {
  this.gameEnv = gameEnv;
  const width = gameEnv.innerWidth;
  const height = gameEnv.innerHeight;
  this.playerStart = { x: Math.round(width * 0.12), y: Math.round(height * 0.68) };
  this.chaserStart = { x: Math.round(width * 0.72), y: Math.round(height * 0.55) };
  ...
  this.classes = [
    { class: GameEnvBackground, data: image_data_court },
    { class: Player, data: sprite_data_player },
    { class: Npc, data: sprite_data_chaser },
  ];
}
```

- All positions are computed as percentages of `gameEnv.innerWidth` and `gameEnv.innerHeight` rather than fixed pixel values — the level scales correctly to any screen size without any changes to the code

---

<a id="api-integration"></a>
### API Integration

**Project Evidence Required:** Implement Leaderboard API with POST/GET scores.  
**Assessment Method:** Code review of fetch calls with error handling.

A **REST API** uses HTTP verbs: POST sends data to the server; GET retrieves data. The `Leaderboard` class handles the actual fetch calls, and `GameLevelBasketball` drives it by calling `submitScore()` with the computed round result.

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

- `scoreSubmittedThisRound` prevents duplicate submissions in the same round — set to `true` before the async call so even a slow network response can't trigger a second submit
- Score is calculated from survival time (10 pts/sec) and coins collected (50 pts each) before being posted

---

<a id="asynchronous-io"></a>
### Asynchronous I/O

**Project Evidence Required:** Use `async/await` or promises for API calls.  
**Assessment Method:** Code review of `async/await` or `.then()` chains.

**Asynchronous** code runs outside the current execution frame — it schedules work to happen later without blocking the game loop. A Promise represents a value that will be available in the future; `.catch()` handles any failure without crashing the caller.

```js
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));
```

- `submitScore` returns a Promise; `.catch()` is attached so any network failure, CORS block, or server error is caught and logged without stopping the game

```js
try {
  window.dispatchEvent(new CustomEvent('characters:concept-focus', {
    detail: { level: 'basketball', trigger: 'first-steal' }
  }));
} catch (err) {
  console.warn('Failed to emit basketball concept focus event:', err);
}
```

- Custom events are dispatched asynchronously — the game loop doesn't wait for any listener to respond, keeping the frame rate unaffected

---

<a id="json-parsing"></a>
### JSON Parsing

**Project Evidence Required:** Parse API responses such as leaderboard data and AI responses.  
**Assessment Method:** Code review of `JSON.parse()`, object destructuring.

**JSON** (JavaScript Object Notation) is the standard text format for API data. Its syntax is identical to JS object literals — the same dot notation accesses properties in both. `res.json()` parses the response body text into a plain JS object.

```js
import Leaderboard from '@assets/js/GameEnginev1.1/essentials/Leaderboard.js';
...
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));
```

- The `Leaderboard` class internally calls `res.json()` and destructures the API response — `GameLevelBasketball` consumes the result by calling `submitScore()` and handles errors through `.catch()`
- The `detail` payload on custom events also follows JSON-serializable object structure: `{ level: 'basketball', trigger: 'first-steal' }` — read back with identical dot notation on the receiving end

---

<a id="code-comments"></a>
### Code Comments

**Project Evidence Required:** JSDoc comments for classes and methods (>10% comment density).  
**Assessment Method:** Code review of comment density.

A good comment explains **why** something works the way it does — the non-obvious constraint or trade-off that would cause a bug if removed. Comments that just restate what the code does add no value; comments that explain a speed cap, a backwards loop, or a stun override save the next reader from breaking it.

```js
// Speed curve -> LeBron gets slightly faster over time but has a cap to keep the game fair
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);

// Clamp LeBron's position so he can't leave the visible game area
lebron.position.x = Math.max(0, Math.min(lebron.position.x, this.gameEnv.innerWidth - (lebron.width || 0)));

// Draws the main orange circle (the ball body)
ctx.beginPath();
ctx.arc(cx, cy, r, 0, Math.PI * 2);

// Calculate the direction from LeBron to the player so LeBron can chase
const dx = player.position.x - lebron.position.x;

// Keep the DOM hitbox aligned with the new spawn immediately.
if (typeof coin.setupCanvas === 'function') {
  coin.setupCanvas();
}
```

- Each comment explains the intent behind non-obvious logic — the speed cap exists to keep the game fair, the clamp exists to prevent LeBron from escaping the visible area, and the `setupCanvas` call keeps the DOM collision boundary in sync after a position change

---


<a id="console-debugging"></a>
### Console Debugging

**Project Evidence Required:** Use `console.log` to track game state, variables, method calls.  
**Assessment Method:** Code review of strategic logging in update/collision methods.

**Console logging** traces execution by printing values at key moments — game start, collision events, state transitions. Never log inside the animation loop (`update()` runs 60×/sec and will flood the console in seconds); place logs at one-time state transitions instead.

```js
// Log once when the round resets — confirm all state was cleared correctly
resetRound() {
  console.log('[Basketball] Round reset — caught=false, time=0, coins=0');
  this.caught = false;
  this.caughtAt = 0;
  this.startTime = performance.now();
  this.currentTime = 0;
  ...
}
```

```js
// Log at catch event — confirm collision fired and values are correct
if (this.isHitboxCollision(player, lebron)) {
  console.log(`[Basketball] Caught at t=${this.currentTime.toFixed(2)}s, coins=${this.getCoinsCollected()}`);
  this.caught = true;
  this.caughtAt = now;
  ...
}
```

```js
// Log at level complete — confirm timer hit target and event fired
completeLevel() {
  console.log(`[Basketball] Level complete — survived ${this.currentTime.toFixed(1)}s`);
  ...
}
```

```js
// Error path logging — surface API and event failures without crashing the loop
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));

try {
  window.dispatchEvent(new CustomEvent('characters:level-complete', {
    detail: { level: 'basketball' }
  }));
} catch (err) {
  console.warn('Failed to emit basketball completion event:', err);
}
```

- Logs placed at `resetRound()`, the catch collision, and `completeLevel()` trace the three major state transitions without running every frame
- Each log uses a `[Basketball]` prefix so it's easy to filter in the DevTools console alongside engine logs

---

<a id="hit-box-visualization"></a>
### Hit Box Visualization

**Project Evidence Required:** Draw/visualize collision boundaries to refine detection.  
**Assessment Method:** Demo — toggle hit box display, adjust collision rectangles.

**Hit box visualization** exposes the invisible collision geometry, revealing whether the hit zone actually matches the visible sprite. The same values used in the collision math are reused in the visualization — if they diverge, the visualization is wrong too.

```js
getHitboxRect(obj) {
  const width  = obj.width  || 0;
  const height = obj.height || 0;
  const pos = obj.position || { x: 0, y: 0 };
  const widthReduction  = width  * 0.2;
  const heightReduction = height * 0.2;

  return {
    left:   pos.x + widthReduction,
    right:  pos.x + width - widthReduction,
    top:    pos.y + heightReduction,
    bottom: pos.y + height - heightReduction
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

- `getHitboxRect` computes the shrunk collision boundary by reducing 20% from each side — this makes the collision feel tighter than the sprite so near-misses feel fair
- `isHitboxCollision` performs a standard AABB (axis-aligned bounding box) overlap check using those computed rectangles; to visualize these during debugging, draw a `ctx.strokeRect` using the returned `left`, `top`, `right`, and `bottom` values

---
<a id="source-debugging"></a>
### Source-Level Debugging

**Project Evidence Required:** Set breakpoints in DevTools, step through code execution.  
**Assessment Method:** Demo — use Sources tab to pause and inspect code flow.

A **breakpoint** pauses execution at a specific line so you can inspect every live variable at that exact moment. This is more precise than console logs because you step through code one line at a time and watch values change in real time.

1. Open DevTools → **Sources** tab
2. Navigate to `assets/js/projects/kirby-minigames/levels/GameLevelBasketball.js`
3. Click the line number next to `const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);` inside `update()`
4. Move Astro close to LeBron — execution pauses on that line
5. Inspect `player.position`, `lebron.position`, `dx`, `dy`, `dist`, and `speed` in the **Scope** panel on the right
6. Press **F10** (step over) to advance one line at a time and watch `lebron.position.x` and `lebron.position.y` update

```js
// Good breakpoint targets in GameLevelBasketball.js:

// 1. Chase math — inspect direction vector and speed scaling
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);
lebron.position.x += (dx / dist) * speed;

// 2. Collision detection — inspect hitbox rects at the moment of catch
if (this.isHitboxCollision(player, lebron)) {
  this.caught = true; // pause here to confirm caught flips correctly
}

// 3. Stun application — confirm stun timestamp is set correctly
if (lebron && this.isCircleHittingObject(projectile, lebron)) {
  this.lebronStunUntil = Math.max(this.lebronStunUntil, now + this.lebronStunDurationMs);
}
```

- Setting a breakpoint on the collision check lets you inspect both hitbox rectangles side by side in the Scope panel and confirm the overlap math is working correctly before the state changes

---

<a id="network-debugging"></a>
### Network Debugging

**Project Evidence Required:** Examine Network tab for API calls, CORS errors, response status.  
**Assessment Method:** Demo — inspect fetch requests, response data, error messages.

The **Network tab** records every HTTP request the page makes. CORS (Cross-Origin Resource Sharing) errors appear as blocked requests with a missing `Access-Control-Allow-Origin` header — the fix is always on the server, not in the JavaScript.

1. Open DevTools → **Network** tab → filter by **Fetch/XHR**
2. Get caught by LeBron to trigger `submitRoundScore()`
3. **Headers** tab: confirm POST to the leaderboard endpoint, `Content-Type: application/json`
4. **Payload** tab: confirm JSON body contains `username`, `score`, and `'Basketball'` as the game name
5. **Response** tab: confirm a success response or read the server error message
6. CORS error? `Access-Control-Allow-Origin` is missing from response headers — add it server-side

```js
// This is the call that appears in the Network tab
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err));

// The score being posted — visible in the Payload tab
const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50));
const username = (this.gameEnv?.game?.uid && String(this.gameEnv.game.uid)) || 'Player';
```

- If the request shows a 401 Unauthorized, the session cookie is expired — check the Application tab
- If the request never appears at all, `scoreSubmittedThisRound` may already be `true` from a previous round — check the boolean flag in the Sources panel

---

<a id="application-debugging"></a>
### Application / Storage Debugging

**Project Evidence Required:** Examine cookies, localStorage, session data for login/state.  
**Assessment Method:** Demo — Application tab inspection of stored data.

The **Application tab** exposes cookies, localStorage, and sessionStorage. `GameLevelBasketball` uses `localStorage` to persist best time and best coins between sessions — this tab lets you confirm those values were written correctly and read them back without running the game.

```js
// Best time and coins are saved to localStorage at the end of each round
saveBestTime() {
  try {
    localStorage.setItem('basketball_best_time', String(this.bestTime));
  } catch (_) {}
}

saveBestCoins() {
  try {
    localStorage.setItem('basketball_best_coins', String(this.bestCoins));
  } catch (_) {}
}

// Loaded back when the level initializes
loadBestTime() {
  try {
    return Number(localStorage.getItem('basketball_best_time') || 0);
  } catch (_) {
    return 0;
  }
}
```

- Open DevTools → **Application** tab → **Local Storage** → select the site origin
- Confirm `basketball_best_time` and `basketball_best_coins` keys exist after completing a round
- If `bestTime` is showing 0 at the start of a new session, check here first — the key may be missing, expired, or written as `NaN` if `this.bestTime` was never updated before `saveBestTime()` ran
- Under **Cookies**: confirm the session token exists if the leaderboard is returning 401 errors on score submit

---

<a id="element-inspection"></a>
### Element Inspection

**Project Evidence Required:** Use Element Viewer to inspect canvas, DOM elements, styles.  
**Assessment Method:** Demo — inspect element properties and game object state.

The **Element Inspector** shows the live DOM tree. Because `GameLevelBasketball` creates its HUD elements and projectile canvases dynamically at runtime, this is the only way to confirm they were appended to the right parent with the correct CSS.

```js
// HUD elements created and appended at runtime — inspect these in the Elements tab
this.timeHud = document.createElement('div');
this.timeHud.id = 'basketball-time-hud';
Object.assign(this.timeHud.style, {
  position: 'fixed',
  top: `${safeTop}px`,
  left: '16px',
  zIndex: '20000',
  ...
});
container.appendChild(this.timeHud);

// Each projectile gets its own canvas appended to the game container
projectile.canvas = document.createElement('canvas');
Object.assign(projectile.canvas.style, {
  position: 'absolute',
  width: `${projectile.radius * 2}px`,
  height: `${projectile.radius * 2}px`,
  left: `${projectile.x - projectile.radius}px`,
  top: `${(this.gameEnv.top || 0) + projectile.y - projectile.radius}px`,
  zIndex: '1002',
  pointerEvents: 'none'
});
container.appendChild(projectile.canvas);
```

1. Right-click the HUD timer → **Inspect Element**
2. Confirm `id="basketball-time-hud"`, `position: fixed`, `z-index: 20000` in computed styles
3. Select `#basketball-time-hud` — its `textContent` should update live as time passes
4. Fire a basketball with `E` — a new `<canvas>` element should appear inside the game container and disappear when it expires or hits LeBron
5. HUD not showing? Check that `container` resolved correctly — if `this.gameEnv.container` and `this.gameEnv.gameContainer` are both undefined, `createHud()` returns early and nothing is appended

---

<a id="gameplay-testing"></a>
### Gameplay Testing

**Project Evidence Required:** Test level completion, character interactions, collision detection.  
**Assessment Method:** Live demo — play through level without critical bugs.

**Gameplay testing** verifies that the game behaves correctly as a player — not just that it compiles. Each mechanic below has a specific code path that drives it; all should be verified in a live playthrough before submission.

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

- Level completion triggers after surviving 20 seconds — `completeLevel()` fires a custom event and halts the loop
- Character interaction is tested through the catch collision between the player and LeBron each frame
- Projectile stun is tested by shooting `E` at LeBron and verifying he freezes for 3 seconds
- These are the main gameplay checks a teacher can point to when asking how the round actually works

---

<a id="integration-testing"></a>
### Integration Testing

**Project Evidence Required:** Test API integration (Leaderboard, NPC AI) with live backend.  
**Assessment Method:** Demo — successful score saving and AI responses.

**Integration testing** checks that two separate systems — the game and the leaderboard server — work correctly together. Each item below requires a live backend and tests both the success path and the failure path.

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

- POST: getting caught triggers `submitRoundScore()`; the Network tab in DevTools should show a successful request with the correct JSON body containing username, score, and game name
- Error path: with the server stopped, `submitScore()` falls into `.catch()` and logs a warning instead of crashing the game
- This is the code that saves the round score to the leaderboard backend

---

<a id="api-error-handling"></a>
### API Error Handling

**Project Evidence Required:** Try/catch blocks for API calls and network error handling.  
**Assessment Method:** Code review of error handling for fetch failures.

A **try/catch** block wraps risky code so any failure — network loss, CORS block, or bad response — is caught in one place. The game must keep running when the backend is unreachable.

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

- Both `try/catch` and `.catch()` are present — `try/catch` handles synchronous event dispatch failures and `.catch()` handles async Promise rejections from the leaderboard API
- Failures are logged as warnings so the game loop continues unaffected even when the API or event system is unavailable
- The game stays playable even if the backend or custom event system fails

---

## Final CS 111 Alignment

`GameLevelBasketball` demonstrates every major CS 111 and CSSE objective in a direct, playable way.

- It uses classes, objects, inheritance-based engine components, and constructor chaining through imported `Player`, `Npc`, and `Coin` classes
- It uses numbers, strings, booleans, arrays, and object literals throughout the level configuration and game loop
- It uses conditionals, nested logic, and loops for real gameplay systems: chase AI, stun mechanics, and coin spawning
- It uses keyboard input, canvas rendering, DOM output, and `GameEnv` configuration for all player interaction and visuals
- It uses API integration, asynchronous score submission, and local storage persistence for leaderboard tracking
- It shows state management, collision systems, debugging evidence, and testable game behavior across every phase of the round

## Project Checklist

- 2+ custom character classes extending base classes — `Player extends Character` and `Npc extends Character`, both used in `GameLevelBasketball`
- 5+ methods with parameters — `isCircleHittingObject(projectile, obj)`, `updateProjectiles(now, lebron)`, `drawProjectileSprite(ctx, width, height)`, `isHitboxCollision(a, b)`, `spawnProjectileFromPlayer(player, now)`
- GameLevel configuration via Object Literals — `sprite_data_player`, `sprite_data_chaser`, `image_data_court`, `this.classes`
- Code comments explaining WHY — speed cap, backwards iteration, LeBron clamp, `setupCanvas` alignment
- API Integration — `submitRoundScore()` with `.catch()` error handling, `initLeaderboard()` with `Leaderboard` class
- Debugging — Console logging at state transitions, hit box math via `getHitboxRect`, Source-Level breakpoints, Network tab for score POST, Application tab for `localStorage`, Element inspection for HUD and projectile canvases
- Mini-lesson documentation — embedded runtime GameRunner at the top of this page with WASD and E controls
- Code highlights — reference table mapping every CS 111 category to its location across the basketball codebase
- Complete, playable level — LeBron chase AI, basketball projectile stun, coin collection, 20-second survival win condition, leaderboard score submission

What makes the file especially strong is that every concept connects to something the player can actually see and test — the physics move LeBron, the booleans freeze the round, the canvas draws the basketball, and the API saves the score.
