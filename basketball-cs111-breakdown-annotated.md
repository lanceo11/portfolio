---
layout: post
title: GameLevelBasketball CS 111 Breakdown
description: In-depth, non-table breakdown of how GameLevelBasketball demonstrates CS 111 and CSSE concepts with a runnable GameRunner.
permalink: /basketball-cs111-breakdown2
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
  this.gameEnv = gameEnv;                                                        // save the game environment for later use
  this.playerStart = { x: Math.round(width * 0.12), y: Math.round(height * 0.68) }; // player spawn: 12% from left, 68% from top
  this.chaserStart = { x: Math.round(width * 0.72), y: Math.round(height * 0.55) }; // Kirby spawn: 72% from left, 55% from top

  this.caught = false;           // tracks whether the player has been caught this round
  this.projectiles = [];         // live list of basketball shots currently flying on screen
  this.shootCooldownMs = 5000;   // player must wait 5 seconds between shots
  this.targetSurvivalSeconds = 20; // survive this long to win the round
}
```

- Every number here directly shapes what the player experiences: the spawn positions decide how much running room you start with, the 5-second cooldown forces you to choose when to shoot, and 20 seconds is the win target the timer counts toward.

---

<a id="writing-classes"></a>
### Writing Classes

**Project Evidence Required:** Create a minimum of 2 custom character classes extending base classes.  
**Assessment Method:** Code review of `Player.js`, `NPC.js`, `Enemy.js`-style class definitions.

A **class** is a blueprint for creating objects — it defines their properties and behavior. `extends` sets up an inheritance chain so one class can reuse another's code. `this.classes` tells the engine which objects to create at startup.

```js
// From Player.js — first custom character class
class Player extends Character {                                       // Player is a Character with extra keyboard features
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);                                              // runs Character's setup first
    this.id = data?.id ? data.id.toLowerCase() : `player${Player.playerCount}`; // unique name for finding this object
    this.keypress = data?.keypress || { up: 87, left: 65, down: 83, right: 68 }; // WASD key codes
    this.pressedKeys = {};                                             // remembers which keys are held right now
    this._boundHandleKeyDown = this.handleKeyDown.bind(this);         // locks 'this' so the listener works correctly
    this._boundHandleKeyUp = this.handleKeyUp.bind(this);             // same fix for key-up listener
    this.bindMovementKeyListners();                                    // starts listening for WASD input
    this.gravity = data.GRAVITY || false;                             // whether gravity pulls this character down
    this.acceleration = 0.001;                                        // how fast falling speed builds up
    this.time = 0;                                                    // counts frames of falling
    this.moved = false;                                               // true when player is actively moving
  }
}
```

- `extends Character` means Player automatically inherits `Character`'s canvas setup, velocity, sprite drawing, and collision detection without rewriting any of it.
- Player adds keyboard input, gravity, and touch controls on top of what it inherits.

```js
// From Npc.js — second custom character class
class Npc extends Character {                                         // Npc is a Character that patrols and talks
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);                                             // runs Character's setup first
    this.interact = data?.interact;                                   // function to call when player talks to this NPC
    this.currentQuestionIndex = 0;                                    // tracks which dialogue line comes next
    this.isInteracting = false;                                       // true while a dialogue box is open
    this.walkingArea = data?.walkingArea || null;                     // boundary box the NPC patrols inside
    this.speed = data?.speed || 1;                                    // how many pixels per frame the NPC moves
    this.moveDirection = data?.moveDirection || { x: 1, y: 1 };      // current travel direction
    this.dialogueSystem = new DialogueSystem({                        // creates the pop-up dialogue box
      dialogues: data.dialogues,
      id: this.uniqueId
    });
  }
}
```

- `Npc extends Character` is the second custom class — it inherits the same sprite and canvas system as `Player` but adds patrol movement, dialogue, and interaction handling instead of keyboard input.
- Both `Player` and `Npc` sit at the end of a three-level chain: `GameObject → Character → Player` and `GameObject → Character → Npc`.

```js
// From GameLevelBasketball.js — both classes are used here
{ class: Player, data: sprite_data_player },  // spawns Astro as the keyboard-controlled character
{ class: Npc,    data: sprite_data_chaser },  // spawns Kirby as the auto-chasing NPC
```

- `GameLevelBasketball` orchestrates both classes by placing them in `this.classes` — the engine reads this array and calls `new Player(...)` and `new Npc(...)` to bring them into the level.

---

<a id="methods-and-parameters"></a>
### Methods & Parameters

**Project Evidence Required:** Implement methods with parameters, such as `collisionHandler(other, direction)`.  
**Assessment Method:** Code review of method signatures with 2 or more parameters.

A **method** is a function that belongs to a class; it uses `this` to access the instance's own data. **Parameters** are the named inputs declared in parentheses — they let one method body handle many different callers without duplicating code.

```js
isCircleHittingObject(projectile, obj) {              // checks if a flying ball has hit a rectangular target
  const rect = this.getHitboxRect(obj);               // shrinks obj's bounding box to its collision zone
  const nearestX = Math.max(rect.left, Math.min(projectile.x, rect.right));  // closest point on rect to ball (x-axis)
  const nearestY = Math.max(rect.top,  Math.min(projectile.y, rect.bottom)); // closest point on rect to ball (y-axis)
  const dx = projectile.x - nearestX;                 // horizontal gap between ball center and nearest rect point
  const dy = projectile.y - nearestY;                 // vertical gap between ball center and nearest rect point
  return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius); // true = ball overlaps rect (a hit)
}
```

- **This method has nothing to do with the sprite image** — it works entirely with invisible numbers (positions and a radius). The sprite is just the orange circle drawn on screen; this method is the math that decides whether that circle's center is close enough to Kirby's hitbox rectangle to count as a hit. You could delete the sprite entirely and the collision would still work.
- This is the method that makes shooting Kirby feel accurate — if the ball's center lands within its own radius of Kirby's box, it registers as a stun.

```js
updateProjectiles(now, lebron) {                         // moves every flying ball and checks for hits each frame
  for (let i = this.projectiles.length - 1; i >= 0; i -= 1) { // loop backwards so splicing doesn't skip items
    const projectile = this.projectiles[i];              // grab the current shot
    projectile.x += projectile.vx;                      // move ball horizontally by its velocity
    projectile.y += projectile.vy;                      // move ball vertically by its velocity
    ...
  }
}
```

- `now` is used to expire old projectiles (each shot has a `bornAt` timestamp and a `projectileLifeMs` limit), and `lebron` is passed in so the method can immediately check `isCircleHittingObject` — both parameters are doing real work every frame.
- This is the method that makes every shot actually travel across the court and disappear when it misses or hits.

```js
drawProjectileSprite(ctx, width, height) {         // paints a basketball onto an offscreen canvas
  const cx = width / 2;                            // horizontal center of the canvas
  const cy = height / 2;                           // vertical center of the canvas
  const r = Math.min(width, height) * 0.42;        // radius: 42% of the smaller canvas dimension
  ctx.beginPath();                                  // start a new drawing path
  ctx.arc(cx, cy, r, 0, Math.PI * 2);              // trace a full circle
  ctx.fillStyle = '#f68b1f';                        // orange ball color
  ctx.fill();                                       // fill the circle
}
```

- **This is the sprite method** — unlike `isCircleHittingObject`, this one is purely about what the player sees. It draws the orange ball and its seam lines. It never checks positions or radii for collision; it only paints pixels.
- Every projectile gets its own mini-canvas, and this method is what makes that canvas look like a basketball instead of a blank square.

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
for (let descriptor of this.gameObjectClasses) {           // loop through every entry in this.classes
  const expandedDescriptors = this.expandDescriptor(descriptor) // handle any repeated/template entries
  for (let gameObjectClass of expandedDescriptors) {
    if (!gameObjectClass.data) gameObjectClass.data = {}    // give empty data if none was provided
    let gameObject = new gameObjectClass.class(gameObjectClass.data, this.gameEnv) // CREATE the actual object
    this.gameEnv.gameObjects.push(gameObject)              // add it to the live game world
  }
}
```

- `new gameObjectClass.class(...)` is the exact line where Astro, Kirby, and every coin come to life — before this line they are just blueprints; after it they exist in the game and start running their own `update()` each frame.

```js
// From GameLevelBasketball.js — the level's class manifest that GameLevel.js reads
this.classes = [
  { class: GameEnvBackground, data: image_data_court },  // the court image behind everything
  { class: Player,            data: sprite_data_player },// Astro — keyboard-controlled
  { class: Npc,               data: sprite_data_chaser },// Kirby — auto-chasing
  { class: Coin,              data: coin_1 },             // first coin, top-left area
  { class: Coin,              data: coin_2 },             // second coin, center area
  { class: Coin,              data: coin_3 },             // third coin, top-right area
  { class: Barrier,           data: barrier_bench_top },  // invisible wall along the top bench
  { class: Barrier,           data: barrier_bench_bottom },// invisible wall along the bottom bench
  { class: Barrier,           data: barrier_gatorade_left },// invisible wall left sideline
  { class: Barrier,           data: barrier_gatorade_right } // invisible wall right sideline
];
```

- The same `Coin` class appears three times with different `data` objects — each `new Coin(...)` call produces a completely separate coin at a different position, proving that one blueprint can make many independent objects.

---

<a id="inheritance"></a>
### Inheritance (Basic)

**Project Evidence Required:** Create a class hierarchy with 2+ levels.  
**Assessment Method:** Code review of `extends` keyword and inheritance chain.

**Inheritance** lets a child class automatically get all the properties and methods of its parent without rewriting them. The chain here is `GameObject → Character → Player` and `GameObject → Character → Npc`. Each level adds only what it specifically needs.

```js
// From Character.js — middle of the chain
class Character extends GameObject {          // Character is a GameObject that can also draw sprites
  constructor(data = null, gameEnv = null) {
    super(gameEnv);                           // registers this object with the engine (GameObject's job)
    this.canvas = document.createElement("canvas"); // creates the drawing surface for the sprite
    this.canvas.id = data.id || "default";   // gives the canvas a unique HTML id
    this.velocity = { x: 0, y: 0 };          // starts stationary
    this.position = { ...data.INIT_POSITION }; // places character at its configured starting spot
    this.spriteData = data;                   // stores the full config object for later use
    this.spriteSheet = new Image();           // creates an image element to load the sprite PNG
    this.spriteSheet.src = data.src;          // points the image at the correct PNG file
  }
}
```

- `Character` is the shared middle layer — it's the reason both Astro and Kirby can draw themselves on screen without each needing their own canvas and image-loading code.

```js
// From Player.js — leaf of the chain
class Player extends Character {             // Player is a Character that responds to the keyboard
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);                    // runs Character's full setup first
    this.keypress = data?.keypress || { up: 87, left: 65, down: 83, right: 68 }; // WASD bindings
    this.pressedKeys = {};                   // tracks which keys are currently held
    this.bindMovementKeyListners();          // attaches keydown/keyup listeners
  }
}

// From Npc.js — separate leaf of the chain
class Npc extends Character {               // Npc is a Character that moves on its own and talks
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);                   // runs Character's full setup first
    this.walkingArea = data?.walkingArea || null; // optional boundary for patrol
    this.speed = data?.speed || 1;          // how many pixels per frame it travels
    this.dialogueSystem = new DialogueSystem({ dialogues: data.dialogues }); // pop-up chat box
  }
}
```

- Both `Player` and `Npc` use `extends Character` — because of this, both automatically have `draw()`, `move()`, `resize()`, and `collisionChecks()` without writing those methods themselves. The `extends` keyword is the whole reason the basketball level only needs to configure data, not rebuild the rendering engine.

---

<a id="method-overriding"></a>
### Method Overriding

**Project Evidence Required:** Override parent methods such as `update()`, `draw()`, `handleCollision()`.  
**Assessment Method:** Code review of polymorphic implementations.

**Overriding** means a child class defines a method with the same name as a parent method, replacing its behavior. Calling `super.methodName()` runs the parent's version first, then the child adds its own logic on top — so you extend rather than replace.

```js
// From Player.js — Player overrides update() from Character
update() {
  super.update();                          // runs Character's update: steps sprite animation and calls draw()
  if (!this.moved) {                       // if the player isn't pressing any movement key...
    if (this.gravity) {                    // ...and gravity is enabled for this character...
      this.time += 1;                      // increment the fall timer each frame
      this.velocity.y += 0.5 + this.acceleration * this.time; // pull character downward, accelerating
    }
  } else {
    this.time = 0;                         // reset fall timer the moment the player moves
  }
}
```

- `super.update()` preserves the sprite animation so Astro keeps walking — then the gravity block adds falling on top. Without `super.update()`, the character would fall but the animation would freeze.

```js
// From Player.js — Player completely overrides handleCollisionReaction()
handleCollisionReaction(other) {
  try {
    const touchPoints = this.collisionData?.touchPoints?.this; // which sides are currently touching
    if (touchPoints) {
      if (touchPoints.left || touchPoints.right) {  // if hitting a wall on the left or right...
        this.velocity.x = 0;                        // stop horizontal movement
      }
      if (touchPoints.top || touchPoints.bottom) {  // if hitting a floor or ceiling...
        this.velocity.y = 0;                        // stop vertical movement
      }
    }
  } catch (_) {}
  super.handleCollisionReaction(other);             // let parent handle anything else
}
```

- This override is what stops Astro from sliding through barriers — it zeros out velocity along whichever axis is blocked before passing control back to the parent.

```js
// From GameLevelBasketball.js — the level overrides randomizePosition on each coin at runtime
coin.randomizePosition = () => {
  coin.position.x = xMin + Math.random() * Math.max(1, xMax - xMin); // random x within court bounds
  coin.position.y = yMin + Math.random() * Math.max(1, yMax - yMin); // random y within court bounds
  if (typeof coin.setupCanvas === 'function') {
    coin.setupCanvas(); // re-align the invisible hitbox with the new position immediately
  }
};
```

- This dynamically replaces the coin's default `randomizePosition` so coins only ever respawn inside the playable court area, never in the bleachers or out of bounds.

---

<a id="constructor-chaining"></a>
### Constructor Chaining

**Project Evidence Required:** Use `super()` to chain constructors.  
**Assessment Method:** Code review of `super(data, gameEnv)` calls.

`super()` inside a constructor calls the parent class's constructor, passing along any arguments it needs. JavaScript requires `super()` before you can use `this` in a child constructor — without it the engine throws a `ReferenceError`. Every level of the hierarchy initializes itself in order before the child adds its own properties.

```js
// Chain visualized top to bottom: Player → Character → GameObject

// From Player.js
class Player extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);                        // ① triggers Character's constructor with the full data object
    // ② these lines only run after Character (and GameObject above it) fully complete:
    this.keypress = data?.keypress || { up: 87, left: 65, down: 83, right: 68 }; // WASD config
    this.pressedKeys = {};                       // starts with no keys pressed
    this.gravity = data.GRAVITY || false;        // gravity setting from the data object
  }
}

// From Character.js
class Character extends GameObject {
  constructor(data = null, gameEnv = null) {
    super(gameEnv);                              // ① triggers GameObject's constructor — engine registration
    // ② Character's own setup runs after GameObject completes:
    this.canvas = document.createElement("canvas"); // drawing surface
    this.velocity = { x: 0, y: 0 };                // starts still
    this.position = { ...data.INIT_POSITION };      // placed at configured start point
  }
}
```

- The chain runs in this order every time: `GameObject` finishes first (engine registration), then `Character` (canvas + physics), then `Player` (keyboard). Each level waits for the level above it to finish before it can use `this`.

```js
// From GameLevelBasketball.js — the data object that travels through every level of the chain
const sprite_data_player = {
  id: 'BasketballPlayer',                        // used by Player to set its unique ID
  INIT_POSITION: { ...this.playerStart },        // used by Character to place the sprite
  pixels: { height: 770, width: 513 },           // used by Character to slice the sprite sheet
  orientation: { rows: 4, columns: 4 },          // used by Character to set animation grid
  hitbox: { widthPercentage: 0.45, heightPercentage: 0.5 }, // used by Character for collision box
  keypress: { up: 87, left: 65, down: 83, right: 68 }       // used by Player for WASD input
};
```

- One plain object, defined in `GameLevelBasketball`, carries all the configuration that every level of the constructor chain needs — each level just reads the keys it cares about and ignores the rest.

```js
// Npc uses the same chain pattern
class Npc extends Character {
  constructor(data = null, gameEnv = null) {
    super(data, gameEnv);                        // ① chains up through Character → GameObject
    // ② Npc-specific properties only after chain completes:
    this.walkingArea = data?.walkingArea || null; // patrol boundary
    this.speed = data?.speed || 1;               // movement speed in px/frame
    this.isInteracting = false;                  // no dialogue open at start
  }
}
```

- Both `Player` and `Npc` follow the identical chaining pattern — `super(data, gameEnv)` is always the first line, before a single `this.` property is set.

---

<a id="conditionals"></a>
### Conditionals

**Project Evidence Required:** Implement collision detection, state transitions.  
**Assessment Method:** Code review of `if/else`, nested conditions.

An `if/else` **conditional** picks one of two execution paths based on a boolean test. In the basketball level, every major game state — running, caught, stunned, complete — is controlled by a chain of conditionals that check flags and timestamps each frame.

```js
if (this.isHitboxCollision(player, lebron)) { // true = the two hitbox rectangles overlap right now
  this.caught = true;                          // flip the caught flag to freeze the game loop
  this.caughtAt = now;                         // record the exact moment of capture for the reset timer
  this.bestTime = Math.max(this.bestTime, this.currentTime); // keep the longer survival time
  this.showCaughtMessage();                    // put "Kirby stole the ball!" on screen
  this.updateHud();                            // refresh the HUD with the new best time
}
```

- This `if` block is the game's main consequence — the single check that turns a near-miss into a game-over and triggers score saving, the on-screen message, and the countdown to reset.

```js
if (this.caught) {                                      // are we in the "just got caught" state?
  if (now - this.caughtAt >= this.roundResetDelayMs) { // has 1.4 seconds passed since being caught?
    this.resetRound();                                  // yes — reset everyone back to spawn
  }
  return;                                               // skip all other update logic while waiting
}
```

- The outer `if` and `return` together freeze LeBron's chase, the timer, and all collision checks the moment the player is caught — one boolean gates an entire branch of the game loop.

---

<a id="nested-conditions"></a>
### Nested Conditions

**Project Evidence Required:** Complex game logic combining multiple conditions.  
**Assessment Method:** Code review of multi-level conditionals.

**Nested conditions** layer multiple independent checks — the outer test must pass before the inner test even runs. Each level enforces a real game rule: is the game active, is the projectile in bounds, is it actually hitting LeBron.

```js
// Level 1: skip all projectile logic while the game is locked or the round is over
if (!this.preGameLocked && !this.caught) {                    // game must be active and player must be alive
  for (let i = this.projectiles.length - 1; i >= 0; i -= 1) {
    const projectile = this.projectiles[i];                   // grab one flying basketball

    // Level 2: is the projectile still alive and in bounds?
    if (this.isProjectileOutOfBounds(projectile) || now - projectile.bornAt > this.projectileLifeMs) {
      this.removeProjectileAt(i);                             // gone — remove from array and DOM
      continue;                                               // skip to next projectile
    }

    // Level 3: is it hitting LeBron specifically?
    if (lebron && this.isCircleHittingObject(projectile, lebron)) { // circle-vs-rect math says yes
      this.lebronStunUntil = Math.max(this.lebronStunUntil, now + this.lebronStunDurationMs); // extend stun window
      lebron.velocity.x = 0;                                 // stop Kirby moving horizontally
      lebron.velocity.y = 0;                                 // stop Kirby moving vertically
      this.removeProjectileAt(i);                            // ball is used up — remove it
    }
  }
}
```

- Level 1 prevents any projectile logic from running during the intro screen or after being caught — no wasted work when nothing can happen.
- Level 2 cleans up expired or out-of-bounds shots before they ever reach the collision check — saves time and keeps the array clean.
- Level 3 is where `isCircleHittingObject` is called — this is the exact moment the basketball sprite's position is checked against Kirby's hitbox rectangle (again: no sprites involved, just numbers).

```js
if (now < this.lebronStunUntil) { // is the stun timer still counting down?
  lebron.velocity.x = 0;          // keep Kirby frozen horizontally
  lebron.velocity.y = 0;          // keep Kirby frozen vertically
  return;                          // skip the entire chase block this frame
}
```

- A separate guard inside `update()` — this is what actually makes Kirby stand still for 3 seconds after being hit. Without this check, the stun timer would be set but Kirby would keep chasing anyway.

---

<a id="numbers"></a>
### Numbers

**Project Evidence Required:** Position, velocity, score tracking.  
**Assessment Method:** Code review of numeric properties.

JavaScript has one number type covering both integers and floats. **Integers** count discrete things like coins collected. **Floats** power physics: positions, velocities, and timestamps update in sub-pixel or sub-millisecond increments each frame for smooth motion.

```js
this.projectileSpeed = 9;             // pixels per frame the basketball travels
this.projectileRadius = 10;           // size of the ball in pixels (also the collision circle's radius)
this.projectileLifeMs = 2200;         // shots disappear after 2.2 seconds if they miss
this.shootCooldownMs = 5000;          // 5 second wait between shots
this.lastShotAt = -Infinity;          // guarantees the very first shot is always allowed immediately
this.lebronStunUntil = 0;            // timestamp when the stun expires (0 = not stunned)
this.lebronStunDurationMs = 3000;    // Kirby freezes for 3 seconds after being hit
this.targetSurvivalSeconds = 20;     // survive this many seconds to complete the level
```

- `lastShotAt = -Infinity` is a clever trick — any real timestamp minus negative infinity is always a huge positive number, which is always greater than the 5-second cooldown, so the first shot is never blocked.

```js
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8); // Kirby starts at 2.1 px/frame, slowly climbs to max 2.8
lebron.position.x += (dx / dist) * speed;                    // move Kirby toward player on the x-axis
lebron.position.y += (dy / dist) * speed;                    // move Kirby toward player on the y-axis
```

- Speed scales with survival time but is capped at 2.8 so the game stays winnable — at the 20-second mark Kirby would reach exactly 2.7 px/frame, just under the cap.

---

<a id="strings"></a>
### Strings

**Project Evidence Required:** Character names, sprite paths, game states.  
**Assessment Method:** Code review of string manipulation.

A **string** is a sequence of characters in quotes. Strings are used here for asset IDs, file paths, and filtering game objects by name — the same dot notation accesses them whether they come from a local object or an API response.

```js
const sprite_src_player = getKirbyImageUrl('astro.png'); // builds the full URL to Astro's sprite PNG
const sprite_src_chaser = getKirbyImageUrl('kirby.png'); // builds the full URL to Kirby's sprite PNG
```

- These string filenames tell `getKirbyImageUrl` which image to load — change `'astro.png'` to any other filename and the engine immediately loads that sprite instead.

```js
const coins = this.gameEnv.gameObjects.filter(
  (obj) => String(obj?.spriteData?.id || '').startsWith('coin_') // keep objects whose id begins with "coin_"
);
```

- `String()` safely converts the id to a string even if it's `null`, and `.startsWith('coin_')` finds all three coins (coin_1, coin_2, coin_3) in one filter — no separate coin list needed.

---

<a id="booleans"></a>
### Booleans

**Project Evidence Required:** Flags such as `isJumping`, `isPaused`, `isVulnerable`.  
**Assessment Method:** Code review of boolean logic.

A **boolean** is either `true` or `false` — a single-bit decision flag. Booleans guard state transitions and prevent a single event (one catch, one score submit) from triggering twice in the same round.

```js
this.caught = false;                    // false = round is active; true = player was just caught
this.preGameLocked = true;              // true = intro screen is showing; blocks all movement and timers
this.scoreSubmittedThisRound = false;   // prevents sending the score to the leaderboard twice
this.levelCompleted = false;            // true once the 20-second win condition is met
this.completionTriggered = false;       // prevents completeLevel() from firing more than once
this.firstStealScrollTriggered = false; // ensures the "concept focus" event fires only on the first catch
```

- Six flags cover every major event in the round — each one acts as a one-way switch that trips once and stays tripped until `resetRound()` resets them all to `false`.

```js
if (this.preGameLocked) return; // stop here — the Start button hasn't been clicked yet
```

- A single boolean check freezes LeBron, the survival timer, and all collision detection simultaneously — one line does the work of six `if` checks.

---

<a id="arrays"></a>
### Arrays

**Project Evidence Required:** Game object collections, level data.  
**Assessment Method:** Code review of array operations.

An **array** is an ordered list that can hold any number of values. The game stores every live projectile in an array that grows via `push()` as shots are fired and shrinks via `splice()` as they expire — the loop never needs to know how many there are in advance.

```js
this.projectiles = [];                   // starts empty — no shots fired yet
...
this.projectiles.push(projectile);       // adds a new basketball to the live list when E is pressed
...
this.projectiles.splice(index, 1);       // removes one shot at 'index' when it expires or hits Kirby
```

- The backwards `for` loop in `updateProjectiles` (`i = length - 1; i >= 0; i -= 1`) is specifically because `splice` shifts every element after the removed one — iterating backwards means earlier indices are never skipped.

```js
this.classes = [
  { class: GameEnvBackground, data: image_data_court }, // court image
  { class: Player,            data: sprite_data_player },// Astro
  { class: Npc,               data: sprite_data_chaser },// Kirby
  { class: Coin,              data: coin_1 },             // coin 1
  { class: Coin,              data: coin_2 },             // coin 2
  { class: Coin,              data: coin_3 },             // coin 3
  ...
];
```

- The engine iterates this array in order — the first entry (background) is drawn first so everything else appears on top of it.

---

<a id="objects-json"></a>
### Objects (JSON)

**Project Evidence Required:** Configuration objects, sprite data.  
**Assessment Method:** Code review of object literals.

An **object literal** `{ key: value }` groups related values under one name. JSON uses the same syntax — objects in code and API responses have identical structure and access patterns.

```js
const sprite_data_player = {
  id: 'BasketballPlayer',                    // used by findById() to locate Astro in the game object list
  greeting: 'Ball handler ready.',           // shown if the player interacts with this character
  src: sprite_src_player,                    // URL of the PNG sprite sheet
  SCALE_FACTOR: 11,                          // how many times bigger to draw vs the raw pixel size
  STEP_FACTOR: 800,                          // controls movement speed
  ANIMATION_RATE: 110,                       // milliseconds between animation frames
  INIT_POSITION: { ...this.playerStart },    // starting x/y position on the court
  pixels: { height: 770, width: 513 },       // total size of the sprite sheet PNG
  orientation: { rows: 4, columns: 4 },      // how the sprite sheet is divided (4 directions × 4 frames)
  down:  { row: 0, start: 0, columns: 4 },   // which row plays when walking down
  left:  { row: 1, start: 0, columns: 4 },   // which row plays when walking left
  right: { row: 2, start: 0, columns: 4 },   // which row plays when walking right
  up:    { row: 3, start: 0, columns: 4 },   // which row plays when walking up
  hitbox: { widthPercentage: 0.45, heightPercentage: 0.5 }, // collision box is 45% × 50% of the sprite
  keypress: { up: 87, left: 65, down: 83, right: 68 }       // WASD key codes
};
```

- Every nested object here (`pixels`, `orientation`, `hitbox`, `keypress`) is accessed with dot notation in the constructor chain — `data.hitbox.widthPercentage` is the same syntax whether `data` came from this object literal or a parsed API response.

---

<a id="mathematical-operators"></a>
### Mathematical Operators

**Project Evidence Required:** Physics calculations such as gravity, velocity, collision.  
**Assessment Method:** Code review of `+`, `-`, `*`, `/` in physics.

Math operators power all physics. `-` finds direction vectors. `/` normalizes them. `*` scales speed. `+` updates position. `Math.hypot` computes straight-line distance without manually squaring and square-rooting.

```js
const dx = player.position.x - lebron.position.x; // how far right the player is from Kirby
const dy = player.position.y - lebron.position.y; // how far down the player is from Kirby
const dist = Math.hypot(dx, dy);                   // straight-line distance between them
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8); // speed climbs with time but caps at 2.8
lebron.position.x += (dx / dist) * speed;          // move Kirby one step toward player on x-axis
lebron.position.y += (dy / dist) * speed;          // move Kirby one step toward player on y-axis
```

- `dx / dist` normalizes the direction vector to length 1, then `* speed` rescales it to the correct step size — this is what makes Kirby always chase at a consistent speed regardless of angle.

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
const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50)); // time score + coin bonus, rounded to whole number
```

- Survival time is worth 10 pts/sec and each coin is worth 50 pts — `Math.round` converts the float to a clean integer before it gets sent to the leaderboard API.

---

<a id="string-operations"></a>
### String Operations

**Project Evidence Required:** Path concatenation, text display.  
**Assessment Method:** Code review of template literals and concatenation.

**Template literals** (backtick strings with `${}`) replace concatenation for building display strings. They embed any live JavaScript expression inline, so the HUD updates in one readable line instead of several joined strings.

```js
this.timeHud.textContent =
  `Time: ${this.currentTime.toFixed(1)}s/${this.targetSurvivalSeconds}s | Best: ${this.bestTime.toFixed(1)}s | ` + // current and best time
  `Coins: ${this.getCoinsCollected()} | Best Coins: ${this.bestCoins}`;                                             // current and best coin count
```

- `.toFixed(1)` formats the float (e.g., `7.3472`) down to one decimal place (e.g., `7.3`) so the HUD stays narrow and readable. This runs every frame so the timer is always live.

```js
const basePath = (this.gameEnv?.path || '').replace(/\/$/, ''); // strip trailing slash if present
const aquaticUrl = `${basePath}/games/aquatic.html`;            // build the full URL for the Aquatic level
```

- `replace(/\/$/, '')` uses a regex to clean the path before the template literal appends the route — without it you'd get double slashes like `/games//aquatic.html`.

---

<a id="boolean-expressions"></a>
### Boolean Expressions

**Project Evidence Required:** Compound conditions in game logic.  
**Assessment Method:** Code review of `&&`, `||`, `!`.

**Boolean operators**: `||` (OR) is true if either side is true; `&&` (AND) requires both sides. Short-circuit evaluation means `||` stops as soon as it finds a truthy value — used here as a safe default to avoid null errors.

```js
if (event.key.toLowerCase() !== 'e' || event.repeat) return; // block if wrong key OR key is being held down
if (this.preGameLocked || this.caught) return;                 // block if intro is showing OR player is caught
```

- Four separate reasons to cancel a shot, all checked in two lines — if any single one is true, the shot is blocked. Short-circuit means if the key isn't 'e', none of the other checks even run.

```js
return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius); // true = ball is close enough to count as a hit
```

- Uses squared distance instead of `Math.sqrt` (which is slower) — mathematically identical result, no sprites involved, just pure coordinate math returning a yes/no answer.

---

<a id="keyboard-input"></a>
### Keyboard Input

**Project Evidence Required:** Arrow keys, space, WASD controls using event listeners.  
**Assessment Method:** Testing that key event handlers respond correctly.

An **event listener** registers a callback function that runs whenever a named browser event fires. Because the game loop can't poll the keyboard directly, listeners write input state at the moment keys are pressed, and the loop reads that state every frame.

```js
document.addEventListener('keydown', this.handleRestartKey); // 'R' key handler — resets the round
document.addEventListener('keydown', this.handleShootKey);   // 'E' key handler — fires a basketball
```

- Both are registered in `initialize()` and removed in `destroy()` — if `destroy()` didn't call `removeEventListener`, these handlers would keep firing even after leaving the level.

```js
handleShootKey(event) {
  if (event.key.toLowerCase() !== 'e' || event.repeat) return; // only 'e', only once per press
  if (this.preGameLocked || this.caught) return;                // not during intro or after being caught
  const now = performance.now();                                // get the current timestamp in ms
  if (now - this.lastShotAt < this.shootCooldownMs) return;    // enforce the 5-second cooldown
  ...
}

handleRestartKey(event) {
  if (event.key.toLowerCase() !== 'r' || !this.caught) return; // only 'r' and only when caught
  this.resetRound();                                            // put everyone back at spawn
}
```

- `event.repeat` being `true` means the key is held down — without this check, holding E would fire dozens of shots per second instead of one clean shot per press.

```js
keypress: { up: 87, left: 65, down: 83, right: 68 } // W=87, A=65, S=83, D=68 (WASD key codes)
```

- These are the ASCII key codes stored in the player's data object — `Player.js` reads them in its constructor and maps them to movement directions.

---

<a id="canvas-rendering"></a>
### Canvas Rendering

**Project Evidence Required:** Draw sprites, backgrounds, platforms using Canvas API.  
**Assessment Method:** Code review of `draw()` method implementations.

The **Canvas 2D API** is a stateful drawing surface — `ctx` is the 2D rendering context. Every projectile is drawn onto its own offscreen canvas using `arc`, `fill`, and `quadraticCurveTo` to produce a recognizable basketball with seam lines.

```js
drawProjectileSprite(ctx, width, height) {   // paints a basketball onto a canvas — purely visual, no collision math
  const cx = width / 2;                      // center x of the canvas
  const cy = height / 2;                     // center y of the canvas
  const r = Math.min(width, height) * 0.42;  // radius: 42% of the shorter canvas dimension

  ctx.beginPath();                           // start a new drawing path
  ctx.arc(cx, cy, r, 0, Math.PI * 2);       // trace a complete circle
  ctx.fillStyle = '#f68b1f';                 // orange fill color
  ctx.fill();                                // fill the circle with orange
  ctx.lineWidth = 4;                         // border thickness
  ctx.strokeStyle = '#8a3d00';               // dark brown outline color
  ctx.stroke();                              // draw the outline

  ctx.beginPath();                           // start the horizontal seam line
  ctx.moveTo(cx - r, cy);                    // begin at the left edge of the ball
  ctx.quadraticCurveTo(cx, cy - 8, cx + r, cy); // curve slightly upward across the center
  ctx.strokeStyle = '#8a3d00';               // same dark brown
  ctx.lineWidth = 3;
  ctx.stroke();                              // draw the horizontal seam

  ctx.beginPath();                           // start the vertical seam line
  ctx.moveTo(cx, cy - r);                    // begin at the top of the ball
  ctx.quadraticCurveTo(cx - 8, cy, cx, cy + r); // curve slightly leftward down the center
  ctx.stroke();                              // draw the vertical seam
}
```

- This entire method is purely visual — it only paints pixels. The collision radius used in `isCircleHittingObject` is a separate number (`projectile.radius = 10`) stored on the projectile object itself, not derived from anything drawn here. The sprite and the hitbox are completely independent.

---

<a id="gameenv-configuration"></a>
### GameEnv Configuration

**Project Evidence Required:** Set canvas size, difficulty levels, game settings.  
**Assessment Method:** Code review of `GameEnv.create()` and `GameSetup.js`.

The **level constructor** is the engine's single configuration hook. Reading dimensions from `gameEnv` instead of hardcoding them makes the level adapt to any canvas size. `this.classes` tells the engine which base objects to instantiate on startup.

```js
constructor(gameEnv) {
  this.gameEnv = gameEnv;                                         // save reference to the whole game environment
  const width = gameEnv.innerWidth;                               // current canvas width in pixels
  const height = gameEnv.innerHeight;                             // current canvas height in pixels
  this.playerStart = { x: Math.round(width * 0.12), y: Math.round(height * 0.68) }; // 12% in, 68% down
  this.chaserStart = { x: Math.round(width * 0.72), y: Math.round(height * 0.55) }; // 72% in, 55% down
  ...
  this.classes = [
    { class: GameEnvBackground, data: image_data_court }, // court background
    { class: Player, data: sprite_data_player },           // Astro
    { class: Npc, data: sprite_data_chaser },              // Kirby
  ];
}
```

- Using percentages (`width * 0.12`) instead of fixed pixel values means spawn points scale correctly whether the game runs in a 400px embed or a 1200px full-screen window.

---

<a id="api-integration"></a>
### API Integration

**Project Evidence Required:** Implement Leaderboard API with POST/GET scores.  
**Assessment Method:** Code review of fetch calls with error handling.

A **REST API** uses HTTP verbs: POST sends data to the server; GET retrieves data. The `Leaderboard` class handles the actual fetch calls, and `GameLevelBasketball` drives it by calling `submitScore()` with the computed round result.

```js
initLeaderboard() {
  if (this.leaderboard) return;           // already initialized — don't create a second one
  this.leaderboard = new Leaderboard(this.gameEnv.gameControl, {
    gameName: 'Basketball',               // identifies which leaderboard table to use
    initiallyHidden: false                // show the leaderboard widget immediately
  });
}

submitRoundScore() {
  if (!this.leaderboard || this.scoreSubmittedThisRound) return; // skip if no leaderboard or already submitted
  const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50)); // compute final score
  const username = (this.gameEnv?.game?.uid && String(this.gameEnv.game.uid)) || 'Player'; // logged-in user or fallback
  this.scoreSubmittedThisRound = true;    // flip the flag BEFORE the async call to prevent double-submit

  this.leaderboard.submitScore(username, score, 'Basketball') // POST to the leaderboard API
    .catch((err) => console.warn('Leaderboard score submit failed:', err)); // log failures without crashing
}
```

- `scoreSubmittedThisRound` is set to `true` before the async `submitScore` call — this is intentional. If it were set after, a fast second trigger (e.g., lag causing two collision checks in one frame) could slip through before the first request finished.

---

<a id="asynchronous-io"></a>
### Asynchronous I/O

**Project Evidence Required:** Use `async/await` or promises for API calls.  
**Assessment Method:** Code review of `async/await` or `.then()` chains.

**Asynchronous** code runs outside the current execution frame — it schedules work to happen later without blocking the game loop. A Promise represents a value that will be available in the future; `.catch()` handles any failure without crashing the caller.

```js
this.leaderboard.submitScore(username, score, 'Basketball') // returns a Promise — doesn't block the frame
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // handles any network or server error
```

- The game loop keeps running at 60fps while this request is in flight — the player can keep moving while the score is being saved in the background.

```js
try {
  window.dispatchEvent(new CustomEvent('characters:concept-focus', { // fires a custom browser event
    detail: { level: 'basketball', trigger: 'first-steal' }          // payload: which level and why
  }));
} catch (err) {
  console.warn('Failed to emit basketball concept focus event:', err); // catch any dispatch error
}
```

- Custom events are asynchronous — listeners on the other end respond in their own time and the game doesn't wait for them, keeping the frame rate smooth.

---

<a id="json-parsing"></a>
### JSON Parsing

**Project Evidence Required:** Parse API responses such as leaderboard data and AI responses.  
**Assessment Method:** Code review of `JSON.parse()`, object destructuring.

**JSON** (JavaScript Object Notation) is the standard text format for API data. Its syntax is identical to JS object literals — the same dot notation accesses properties in both. `res.json()` parses the response body text into a plain JS object.

```js
import Leaderboard from '@assets/js/GameEnginev1.1/essentials/Leaderboard.js'; // import the leaderboard helper class
...
this.leaderboard.submitScore(username, score, 'Basketball') // sends username+score as JSON to the API
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // handles parse or network failures
```

- The `Leaderboard` class internally calls `res.json()` to parse the server's JSON response — `GameLevelBasketball` only needs to call `submitScore()` and handle errors; the parsing is abstracted away.

```js
detail: { level: 'basketball', trigger: 'first-steal' } // JSON-serializable payload on a custom event
```

- This object literal follows JSON structure — any listener reads `event.detail.level` and `event.detail.trigger` with the same dot notation used to read parsed API responses.

---

<a id="code-comments"></a>
### Code Comments

**Project Evidence Required:** JSDoc comments for classes and methods (>10% comment density).  
**Assessment Method:** Code review of comment density.

A good comment explains **why** something works the way it does — the non-obvious constraint or trade-off that would cause a bug if removed.

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

- Each comment explains the intent behind non-obvious logic — the speed cap exists to keep the game fair, the clamp prevents Kirby escaping the screen, and the `setupCanvas()` call keeps the invisible collision boundary in sync after a coin moves.

---

<a id="console-debugging"></a>
### Console Debugging

**Project Evidence Required:** Use `console.log` to track game state, variables, method calls.  
**Assessment Method:** Code review of strategic logging in update/collision methods.

**Console logging** traces execution by printing values at key moments — game start, collision events, state transitions. Never log inside the animation loop (`update()` runs 60×/sec and will flood the console in seconds); place logs at one-time state transitions instead.

```js
// Log once when the round resets — confirm all state was cleared correctly
resetRound() {
  console.log('[Basketball] Round reset — caught=false, time=0, coins=0'); // one-time log at reset
  this.caught = false;             // clear the caught flag
  this.caughtAt = 0;               // clear the catch timestamp
  this.startTime = performance.now(); // restart the survival timer
  this.currentTime = 0;            // reset the displayed time to 0
  ...
}
```

```js
// Log at catch event — confirm collision fired and values are correct
if (this.isHitboxCollision(player, lebron)) {
  console.log(`[Basketball] Caught at t=${this.currentTime.toFixed(2)}s, coins=${this.getCoinsCollected()}`); // one-time log on catch
  this.caught = true;
  this.caughtAt = now;
  ...
}
```

```js
// Log at level complete — confirm timer hit target and event fired
completeLevel() {
  console.log(`[Basketball] Level complete — survived ${this.currentTime.toFixed(1)}s`); // one-time log on win
  ...
}
```

```js
// Error path logging — surface API and event failures without crashing the loop
this.leaderboard.submitScore(username, score, 'Basketball')
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // warn instead of throwing

try {
  window.dispatchEvent(new CustomEvent('characters:level-complete', {
    detail: { level: 'basketball' }
  }));
} catch (err) {
  console.warn('Failed to emit basketball completion event:', err); // warn instead of throwing
}
```

- All logs use a `[Basketball]` prefix — in DevTools you can type `[Basketball]` in the console filter to see only this level's messages alongside the engine's own logs.

---

<a id="hit-box-visualization"></a>
### Hit Box Visualization

**Project Evidence Required:** Draw/visualize collision boundaries to refine detection.  
**Assessment Method:** Demo — toggle hit box display, adjust collision rectangles.

**Hit box visualization** exposes the invisible collision geometry, revealing whether the hit zone actually matches the visible sprite.

```js
getHitboxRect(obj) {
  const width  = obj.width  || 0;            // sprite's rendered pixel width (0 if not loaded yet)
  const height = obj.height || 0;            // sprite's rendered pixel height
  const pos = obj.position || { x: 0, y: 0 }; // current top-left position on screen
  const widthReduction  = width  * 0.2;      // shave 20% off the left side and 20% off the right
  const heightReduction = height * 0.2;      // shave 20% off the top and 20% off the bottom

  return {
    left:   pos.x + widthReduction,          // inner left edge
    right:  pos.x + width - widthReduction,  // inner right edge
    top:    pos.y + heightReduction,         // inner top edge
    bottom: pos.y + height - heightReduction // inner bottom edge
  };
}

isHitboxCollision(a, b) {
  const ar = this.getHitboxRect(a);          // get shrunk box for object A
  const br = this.getHitboxRect(b);          // get shrunk box for object B
  return (
    ar.left   < br.right  &&                 // A's left is inside B's right
    ar.right  > br.left   &&                 // A's right is inside B's left
    ar.top    < br.bottom &&                 // A's top is inside B's bottom
    ar.bottom > br.top                       // A's bottom is inside B's top
  );
}
```

- Shrinking each hitbox by 20% on every side makes near-misses feel fair — the sprite images overlap slightly before a catch registers, which matches player intuition better than pixel-perfect collision.
- To visualize during debugging, call `ctx.strokeRect(rect.left, rect.top, rect.right - rect.left, rect.bottom - rect.top)` using the values returned by `getHitboxRect`.

---

<a id="source-debugging"></a>
### Source-Level Debugging

**Project Evidence Required:** Set breakpoints in DevTools, step through code execution.  
**Assessment Method:** Demo — use Sources tab to pause and inspect code flow.

A **breakpoint** pauses execution at a specific line so you can inspect every live variable at that exact moment.

1. Open DevTools → **Sources** tab
2. Navigate to `assets/js/projects/kirby-minigames/levels/GameLevelBasketball.js`
3. Click the line number next to `const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);` inside `update()`
4. Move Astro close to LeBron — execution pauses on that line
5. Inspect `player.position`, `lebron.position`, `dx`, `dy`, `dist`, and `speed` in the **Scope** panel on the right
6. Press **F10** (step over) to advance one line at a time and watch `lebron.position.x` and `lebron.position.y` update

```js
// Good breakpoint targets in GameLevelBasketball.js:

// 1. Chase math — inspect direction vector and speed scaling
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8); // pause here to see dx, dy, dist, speed all at once
lebron.position.x += (dx / dist) * speed;                    // step over to watch Kirby's position change live

// 2. Collision detection — inspect hitbox rects at the moment of catch
if (this.isHitboxCollision(player, lebron)) {
  this.caught = true; // pause here to confirm both hitbox rectangles overlapped and caught flipped to true
}

// 3. Stun application — confirm stun timestamp is set correctly
if (lebron && this.isCircleHittingObject(projectile, lebron)) {
  this.lebronStunUntil = Math.max(this.lebronStunUntil, now + this.lebronStunDurationMs); // pause to verify timestamp math
}
```

- Setting a breakpoint on the collision check lets you open the Scope panel and read both hitbox rectangles side by side — you can confirm the four overlap conditions are all true before `this.caught` flips.

---

<a id="network-debugging"></a>
### Network Debugging

**Project Evidence Required:** Examine Network tab for API calls, CORS errors, response status.  
**Assessment Method:** Demo — inspect fetch requests, response data, error messages.

The **Network tab** records every HTTP request the page makes.

1. Open DevTools → **Network** tab → filter by **Fetch/XHR**
2. Get caught by LeBron to trigger `submitRoundScore()`
3. **Headers** tab: confirm POST to the leaderboard endpoint, `Content-Type: application/json`
4. **Payload** tab: confirm JSON body contains `username`, `score`, and `'Basketball'` as the game name
5. **Response** tab: confirm a success response or read the server error message
6. CORS error? `Access-Control-Allow-Origin` is missing from response headers — add it server-side

```js
// This is the call that appears in the Network tab
this.leaderboard.submitScore(username, score, 'Basketball') // the POST that shows up under Fetch/XHR
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // network errors land here

// The score being posted — visible in the Payload tab
const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50)); // computed value that gets sent
const username = (this.gameEnv?.game?.uid && String(this.gameEnv.game.uid)) || 'Player'; // sender identity
```

- If the request never appears at all in the Network tab, `scoreSubmittedThisRound` is already `true` — check it in the Sources panel's Scope view during a breakpoint at `submitRoundScore`.

---

<a id="application-debugging"></a>
### Application / Storage Debugging

**Project Evidence Required:** Examine cookies, localStorage, session data for login/state.  
**Assessment Method:** Demo — Application tab inspection of stored data.

The **Application tab** exposes cookies, localStorage, and sessionStorage. `GameLevelBasketball` uses `localStorage` to persist best time and best coins between sessions.

```js
// Best time and coins are saved to localStorage at the end of each round
saveBestTime() {
  try {
    localStorage.setItem('basketball_best_time', String(this.bestTime)); // write best time as a string
  } catch (_) {} // silently ignore if storage is blocked (e.g., private browsing)
}

saveBestCoins() {
  try {
    localStorage.setItem('basketball_best_coins', String(this.bestCoins)); // write best coin count as a string
  } catch (_) {}
}

// Loaded back when the level initializes
loadBestTime() {
  try {
    return Number(localStorage.getItem('basketball_best_time') || 0); // convert stored string back to a number
  } catch (_) {
    return 0; // return 0 if storage read fails
  }
}
```

- Open DevTools → **Application** tab → **Local Storage** → select the site origin. Confirm `basketball_best_time` and `basketball_best_coins` appear after completing a round. If `bestTime` shows 0 at the start of a new session, check here first — the key may be missing or stored as `NaN`.

---

<a id="element-inspection"></a>
### Element Inspection

**Project Evidence Required:** Use Element Viewer to inspect canvas, DOM elements, styles.  
**Assessment Method:** Demo — inspect element properties and game object state.

The **Element Inspector** shows the live DOM tree. Because `GameLevelBasketball` creates its HUD elements and projectile canvases dynamically at runtime, this is the only way to confirm they were appended to the right parent with the correct CSS.

```js
// HUD elements created and appended at runtime — inspect these in the Elements tab
this.timeHud = document.createElement('div');          // creates the timer bar element in memory
this.timeHud.id = 'basketball-time-hud';               // gives it an id for easy selection in DevTools
Object.assign(this.timeHud.style, {
  position: 'fixed',                                   // stays in the corner even if the page scrolls
  top: `${safeTop}px`,                                 // positioned below the nav bar
  left: '16px',                                        // 16px from the left edge
  zIndex: '20000',                                     // sits on top of all game elements
  ...
});
container.appendChild(this.timeHud);                   // adds it to the live DOM

// Each projectile gets its own canvas appended to the game container
projectile.canvas = document.createElement('canvas');  // new canvas element for this one basketball
Object.assign(projectile.canvas.style, {
  position: 'absolute',                                // positioned relative to the game container
  width: `${projectile.radius * 2}px`,                 // canvas visible width = diameter
  height: `${projectile.radius * 2}px`,                // canvas visible height = diameter
  left: `${projectile.x - projectile.radius}px`,       // centered on the projectile's x position
  top: `${(this.gameEnv.top || 0) + projectile.y - projectile.radius}px`, // centered on y, offset by game top
  zIndex: '1002',                                      // above the court background
  pointerEvents: 'none'                                // clicks pass through the ball canvas to the game below
});
container.appendChild(projectile.canvas);              // adds the ball canvas to the live DOM
```

- Press `E` to fire a shot — a new `<canvas>` element should appear inside the game container in the Elements panel and disappear the moment it expires or hits Kirby. If HUD elements are missing, check that `container` isn't `null` — the method returns early if both `gameEnv.container` and `gameEnv.gameContainer` are undefined.

---

<a id="gameplay-testing"></a>
### Gameplay Testing

**Project Evidence Required:** Test level completion, character interactions, collision detection.  
**Assessment Method:** Live demo — play through level without critical bugs.

**Gameplay testing** verifies that the game behaves correctly as a player — not just that it compiles.

```js
if (this.currentTime >= this.targetSurvivalSeconds) { // has the player survived 20 seconds?
  this.completeLevel();                               // yes — trigger the win sequence
  return;                                             // stop the update loop
}
```

```js
if (this.isHitboxCollision(player, lebron)) { // do the two hitbox rectangles overlap?
  this.caught = true;                          // freeze the round
  this.caughtAt = now;                         // record catch time for the reset delay
  this.showCaughtMessage();                    // display "Kirby stole the ball!" on screen
}
```

```js
if (lebron && this.isCircleHittingObject(projectile, lebron)) { // did the ball's circle touch Kirby's box?
  this.lebronStunUntil = Math.max(this.lebronStunUntil, now + this.lebronStunDurationMs); // extend stun timer
  lebron.velocity.x = 0;                                        // stop Kirby's horizontal movement
  lebron.velocity.y = 0;                                        // stop Kirby's vertical movement
  this.removeProjectileAt(i);                                    // remove the ball that landed the hit
}
```

- Three testable gameplay checks: survive 20 seconds, get caught by Kirby, shoot Kirby with E. Each maps directly to one code path.

---

<a id="integration-testing"></a>
### Integration Testing

**Project Evidence Required:** Test API integration (Leaderboard, NPC AI) with live backend.  
**Assessment Method:** Demo — successful score saving and AI responses.

**Integration testing** checks that two separate systems — the game and the leaderboard server — work correctly together.

```js
submitRoundScore() {
  if (!this.leaderboard || this.scoreSubmittedThisRound) return; // guard: one submit per round
  const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50)); // compute score
  const username = (this.gameEnv?.game?.uid && String(this.gameEnv.game.uid)) || 'Player'; // get username
  this.scoreSubmittedThisRound = true;                           // lock before async call

  this.leaderboard.submitScore(username, score, 'Basketball')   // POST to leaderboard backend
    .catch((err) => console.warn('Leaderboard score submit failed:', err)); // failure path
}
```

- To test the error path: stop the backend server, get caught, and confirm the `.catch()` fires a `console.warn` instead of an uncaught error — the game should keep running normally with no visible crash.

---

<a id="api-error-handling"></a>
### API Error Handling

**Project Evidence Required:** Try/catch blocks for API calls and network error handling.  
**Assessment Method:** Code review of error handling for fetch failures.

A **try/catch** block wraps risky code so any failure — network loss, CORS block, or bad response — is caught in one place.

```js
try {
  window.dispatchEvent(new CustomEvent('characters:concept-focus', { // fire the event
    detail: { level: 'basketball', trigger: 'first-steal' }          // payload
  }));
} catch (err) {
  console.warn('Failed to emit basketball concept focus event:', err); // catch any synchronous dispatch failure
}
```

```js
this.leaderboard.submitScore(username, score, 'Basketball') // async POST
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // catch any async network failure
```

- `try/catch` catches synchronous failures (event dispatch errors); `.catch()` catches asynchronous failures (network errors, server rejections). Both log warnings instead of throwing, so the game loop keeps running at full speed even when the backend is down.

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
