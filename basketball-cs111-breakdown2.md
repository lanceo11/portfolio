---
layout: post
title: GameLevelBasketball CS 111 Breakdown
description: In-depth breakdown of how GameLevelBasketball demonstrates CS 111 and CSSE concepts with a runnable GameRunner.
permalink: /basketball-cs111-breakdown3
hide: true
toc: true
toc_history: true
codemirror: true
---

<a href="#top" id="back-to-top-btn" style="position:fixed;bottom:24px;right:24px;background:#0d4a8a;color:#fff;border:none;border-radius:8px;padding:10px 18px;font-size:14px;cursor:pointer;z-index:99999;text-decoration:none;">▲ Top</a>

<a id="top"></a>

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

---

<div style="background:#010a18; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:18px 0;">

## 📋 CS 111 LEARNING OBJECTIVES — JUMP TO SECTION

<table style="width:100%; border-collapse:collapse;">
<tr>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top; width:50%;">
<strong style="color:#4a9eff;">1 → Object-Oriented Programming</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#writing-classes" style="color:#aaa;">Writing classes</a> • <a href="#inheritance" style="color:#aaa;">Inheritance</a> • <a href="#method-overriding" style="color:#aaa;">Method overriding</a> • <a href="#constructor-chaining" style="color:#aaa;">Constructor chaining</a> • <a href="#instantiation-and-objects" style="color:#aaa;">Instantiation</a></span>
</td>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top; width:50%;">
<strong style="color:#4a9eff;">2 → Methods & Parameters</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#methods-and-parameters" style="color:#aaa;">Parameters</a> • Return values • Single responsibility</span>
</td>
</tr>
<tr>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top;">
<strong style="color:#4a9eff;">3 → Control Structures</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#conditionals" style="color:#aaa;">Conditionals</a> • <a href="#nested-conditions" style="color:#aaa;">Nested conditions</a> • Iteration</span>
</td>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top;">
<strong style="color:#4a9eff;">4 → Data Types & Operators</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#numbers" style="color:#aaa;">Numbers</a> • <a href="#strings" style="color:#aaa;">Strings</a> • <a href="#booleans" style="color:#aaa;">Booleans</a> • <a href="#arrays" style="color:#aaa;">Arrays</a> • <a href="#objects-json" style="color:#aaa;">Objects</a> • <a href="#mathematical-operators" style="color:#aaa;">Math</a> • <a href="#boolean-expressions" style="color:#aaa;">Boolean expressions</a></span>
</td>
</tr>
<tr>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top;">
<strong style="color:#4a9eff;">5 → Input / Output</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#keyboard-input" style="color:#aaa;">Keyboard input</a> • <a href="#canvas-rendering" style="color:#aaa;">Canvas rendering</a> • <a href="#gameenv-configuration" style="color:#aaa;">GameEnv config</a></span>
</td>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top;">
<strong style="color:#4a9eff;">6 → API, Async I/O & JSON</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#api-integration" style="color:#aaa;">API calls</a> • <a href="#asynchronous-io" style="color:#aaa;">async/await</a> • <a href="#json-parsing" style="color:#aaa;">JSON.parse</a> • localStorage</span>
</td>
</tr>
<tr>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top;">
<strong style="color:#4a9eff;">7 → Debugging</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#console-debugging" style="color:#aaa;">Console</a> • <a href="#source-debugging" style="color:#aaa;">DevTools</a> • <a href="#network-debugging" style="color:#aaa;">Network</a> • <a href="#application-debugging" style="color:#aaa;">Storage</a> • <a href="#element-inspection" style="color:#aaa;">Elements</a></span>
</td>
<td style="padding:12px; border:1px solid #0d4a8a; border-radius:6px; background:#020f22; vertical-align:top;">
<strong style="color:#4a9eff;">8 → Testing & Verification</strong><br>
<span style="color:#aaa; font-size:13px;"><a href="#gameplay-testing" style="color:#aaa;">Gameplay checklist</a> • <a href="#integration-testing" style="color:#aaa;">Live demo</a></span>
</td>
</tr>
</table>

</div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:18px 0;">

### CS111 Rubric Map

Every required objective mapped to the section where the evidence lives.

| Learning Objective | Required Evidence | Section |
|---|---|---|
| Writing classes | Minimum 2 custom classes | [Section 1.1](#writing-classes) |
| Methods and parameters | Methods with 2+ parameters | [Section 2.1](#methods-and-parameters) |
| Instantiation and objects | Game objects created with new | [Section 1.3](#instantiation-and-objects) |
| Inheritance | 2+ level class hierarchy | [Section 1.2](#inheritance) |
| Method overriding | Override lifecycle methods | [Section 1.4](#method-overriding) |
| Constructor chaining | super(data, gameEnv) | [Section 1.5](#constructor-chaining) |
| Conditions | if/else branching | [Section 3.1](#conditionals) |
| Nested conditions | Multi-level logic | [Section 3.2](#nested-conditions) |
| Numbers | Position, velocity, score | [Section 4.1](#numbers) |
| Strings | Names, paths, states | [Section 4.2](#strings) |
| Booleans | Flags | [Section 4.3](#booleans) |
| Arrays | Collections | [Section 4.4](#arrays) |
| Objects / JSON | Object literals and parsed data | [Section 4.5](#objects-json) |
| Math operators | Physics and scoring | [Section 4.6](#mathematical-operators) |
| String operations | Paths and display text | [Section 4.7](#string-operations) |
| Boolean expressions | Compound conditions | [Section 4.8](#boolean-expressions) |
| Keyboard input | Event listeners and movement | [Section 5.1](#keyboard-input) |
| Canvas rendering | Draw sprites/assets | [Section 5.2](#canvas-rendering) |
| GameEnv config | Canvas size and settings | [Section 5.3](#gameenv-configuration) |
| API integration | Leaderboard and AI NPC | [Section 6.1](#api-integration) |
| Async I/O | async/await | [Section 6.2](#asynchronous-io) |
| JSON parsing | JSON.parse / JSON.stringify | [Section 6.3](#json-parsing) |
| Debugging | DevTools and logs | [Section 7](#console-debugging) |
| Testing | Gameplay verification | [Section 8](#gameplay-testing) |

</div>

---

## Overview

`GameLevelBasketball.js` is a strong CS 111 capstone example because it combines:
- Object-oriented design
- State management
- Collision logic
- Keyboard input
- Canvas rendering
- Local storage
- Leaderboard API usage
- Debugging-friendly structure

**Core gameplay loop:**
- Astro survives on the court
- Kirby chases Astro around the map
- Coins spawn around the arena
- The player can shoot a basketball projectile to stun the chaser

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 1 — Object-Oriented Programming

</div>

---

<a id="writing-classes"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 1.1 — Writing Classes

**Project Evidence Required:** Create a minimum of 2 custom character classes extending base classes.  
**Assessment Method:** Code review of `Player.js`, `NPC.js`, `Enemy.js`-style class definitions.

**What it is:**
- A **class** is a blueprint for creating objects — you define it once, then use `new` to make as many objects from it as you need
- `extends` sets up an inheritance chain so one class can reuse another's code
- `this.classes` tells the engine which objects to create at startup

</div>

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

**What this code does:**
- `extends Character` means Player automatically inherits `Character`'s canvas setup, velocity, sprite drawing, and collision detection without rewriting any of it
- Player adds keyboard input, gravity, and touch controls on top of what it inherits

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

**What this code does:**
- `Npc extends Character` is the second custom class — it inherits the same sprite and canvas system as `Player` but adds patrol movement, dialogue, and interaction handling instead of keyboard input
- Both `Player` and `Npc` sit at the end of a three-level chain: `GameObject → Character → Player` and `GameObject → Character → Npc`

```js
// From GameLevelBasketball.js — both classes are used here
{ class: Player, data: sprite_data_player },  // spawns Astro as the keyboard-controlled character
{ class: Npc,    data: sprite_data_chaser },  // spawns Kirby as the auto-chasing NPC
```

**What this code does:**
- `GameLevelBasketball` orchestrates both classes by placing them in `this.classes`
- The engine reads this array and calls `new Player(...)` and `new Npc(...)` to bring them into the level

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="inheritance"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 1.2 — Inheritance (Basic)

**Project Evidence Required:** Create a class hierarchy with 2+ levels.  
**Assessment Method:** Code review of `extends` keyword and inheritance chain.

**What it is:**
- **Inheritance** lets a child class automatically get all the properties and methods of its parent without rewriting them
- The chain here is `GameObject → Character → Player` and `GameObject → Character → Npc`
- Each level adds only what it specifically needs

</div>

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

**What this code does:**
- `Character` is the shared middle layer
- Because of it, both Astro and Kirby can draw themselves on screen without each needing their own canvas and image-loading code

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

**What this code does:**
- Both `Player` and `Npc` use `extends Character`
- Because of this, both automatically have `draw()`, `move()`, `resize()`, and `collisionChecks()` without writing those methods themselves
- The `extends` keyword is the whole reason the basketball level only needs to configure data, not rebuild the rendering engine

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="method-overriding"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 1.3 — Method Overriding

**Project Evidence Required:** Override parent methods such as `update()`, `draw()`, `handleCollision()`.  
**Assessment Method:** Code review of polymorphic implementations.

**What it is:**
- **Overriding** means a child class defines a method with the same name as a parent method, replacing its behavior
- Calling `super.methodName()` runs the parent's version first, then the child adds its own logic on top — so you extend rather than replace

</div>

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

**What this code does:**
- `super.update()` preserves the sprite animation so Astro keeps walking — then the gravity block adds falling on top
- Without `super.update()`, the character would fall but the animation would freeze

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

**What this code does:**
- This override is what stops Astro from sliding through barriers
- It zeros out velocity along whichever axis is blocked before passing control back to the parent

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

**What this code does:**
- Dynamically replaces the coin's default `randomizePosition`
- Coins only ever respawn inside the playable court area, never in the bleachers or out of bounds

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="constructor-chaining"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 1.4 — Constructor Chaining

**Project Evidence Required:** Use `super()` to chain constructors.  
**Assessment Method:** Code review of `super(data, gameEnv)` calls.

**What it is:**
- `super()` inside a constructor calls the parent class's constructor, passing along any arguments it needs
- JavaScript requires `super()` before you can use `this` in a child constructor — without it the engine throws a `ReferenceError`
- Every level of the hierarchy initializes itself in order before the child adds its own properties

</div>

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

**What this code does:**
- The chain runs in this order every time: `GameObject` finishes first (engine registration), then `Character` (canvas + physics), then `Player` (keyboard)
- Each level waits for the level above it to finish before it can use `this`

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

**What this code does:**
- One plain object, defined in `GameLevelBasketball`, carries all the configuration that every level of the constructor chain needs
- Each level just reads the keys it cares about and ignores the rest

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="instantiation-and-objects"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 1.5 — Instantiation & Objects

**Project Evidence Required:** Instantiate game objects in GameLevel configuration.  
**Assessment Method:** Code review of GameLevel setup objects.

**What it is:**
- **Instantiation** means calling `new ClassName()` to create an independent object from a class blueprint
- Each object owns its own copy of the class's properties — changing one coin's position doesn't affect any other
- The engine does this inside `GameLevel.js` by reading `this.classes` from the level and calling `new` on each entry

</div>

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

**What this code does:**
- `new gameObjectClass.class(...)` is the exact line where Astro, Kirby, and every coin come to life
- Before this line they are just blueprints; after it they exist in the game and start running their own `update()` each frame

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

**What this code does:**
- The same `Coin` class appears three times with different `data` objects
- Each `new Coin(...)` call produces a completely separate coin at a different position
- Proves that one blueprint can make many independent objects

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 2 — Methods & Parameters

</div>

---

<a id="methods-and-parameters"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 2.1 — Methods with Parameters

**Project Evidence Required:** Implement methods with parameters, such as `collisionHandler(other, direction)`.  
**Assessment Method:** Code review of method signatures with 2 or more parameters.

**What it is:**
- A **method** is a function that belongs to a class; it uses `this` to access the instance's own data
- **Parameters** are the named inputs declared in parentheses — they let one method body handle many different callers without duplicating code

</div>

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

**What this code does:**
- Works entirely with invisible numbers (positions and a radius) — the sprite is just the orange circle drawn on screen
- This method is the math that decides whether that circle's center is close enough to Kirby's hitbox rectangle to count as a hit
- You could delete the sprite entirely and the collision would still work
- This is the method that makes shooting Kirby feel accurate

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

**What this code does:**
- `now` is used to expire old projectiles (each shot has a `bornAt` timestamp and a `projectileLifeMs` limit)
- `lebron` is passed in so the method can immediately check `isCircleHittingObject` — both parameters are doing real work every frame
- This is the method that makes every shot actually travel across the court and disappear when it misses or hits

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

**What this code does:**
- Unlike `isCircleHittingObject`, this one is purely about what the player sees
- It draws the orange ball and its seam lines — never checks positions or radii for collision; it only paints pixels
- Every projectile gets its own mini-canvas, and this method is what makes that canvas look like a basketball instead of a blank square

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 3 — Control Structures

</div>

---

<a id="conditionals"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 3.1 — Conditionals

**Project Evidence Required:** Implement collision detection, state transitions.  
**Assessment Method:** Code review of `if/else`, nested conditions.

**What it is:**
- An `if/else` **conditional** picks one of two execution paths based on a boolean test
- In the basketball level, every major game state — running, caught, stunned, complete — is controlled by a chain of conditionals that check flags and timestamps each frame

</div>

```js
if (this.isHitboxCollision(player, lebron)) { // true = the two hitbox rectangles overlap right now
  this.caught = true;                          // flip the caught flag to freeze the game loop
  this.caughtAt = now;                         // record the exact moment of capture for the reset timer
  this.bestTime = Math.max(this.bestTime, this.currentTime); // keep the longer survival time
  this.showCaughtMessage();                    // put "Kirby stole the ball!" on screen
  this.updateHud();                            // refresh the HUD with the new best time
}
```

**What this code does:**
- This `if` block is the game's main consequence
- The single check that turns a near-miss into a game-over and triggers score saving, the on-screen message, and the countdown to reset

```js
if (this.caught) {                                      // are we in the "just got caught" state?
  if (now - this.caughtAt >= this.roundResetDelayMs) { // has 1.4 seconds passed since being caught?
    this.resetRound();                                  // yes — reset everyone back to spawn
  }
  return;                                               // skip all other update logic while waiting
}
```

**What this code does:**
- The outer `if` and `return` together freeze LeBron's chase, the timer, and all collision checks the moment the player is caught
- One boolean gates an entire branch of the game loop

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="nested-conditions"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 3.2 — Nested Conditions

**Project Evidence Required:** Complex game logic combining multiple conditions.  
**Assessment Method:** Code review of multi-level conditionals.

**What it is:**
- **Nested conditions** layer multiple independent checks — the outer test must pass before the inner test even runs
- Each level enforces a real game rule: is the game active, is the projectile in bounds, is it actually hitting LeBron

</div>

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

**What this code does:**
- Level 1: prevents any projectile logic from running during the intro screen or after being caught — no wasted work when nothing can happen
- Level 2: cleans up expired or out-of-bounds shots before they ever reach the collision check — saves time and keeps the array clean
- Level 3: is where `isCircleHittingObject` is called — this is the exact moment the basketball sprite's position is checked against Kirby's hitbox rectangle

```js
if (now < this.lebronStunUntil) { // is the stun timer still counting down?
  lebron.velocity.x = 0;          // keep Kirby frozen horizontally
  lebron.velocity.y = 0;          // keep Kirby frozen vertically
  return;                          // skip the entire chase block this frame
}
```

**What this code does:**
- A separate guard inside `update()` — this is what actually makes Kirby stand still for 3 seconds after being hit
- Without this check, the stun timer would be set but Kirby would keep chasing anyway

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 4 — Data Types & Operators

</div>

---

<a id="numbers"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.1 — Numbers

**Project Evidence Required:** Position, velocity, score tracking.  
**Assessment Method:** Code review of numeric properties.

**What it is:**
- JavaScript has one number type covering both integers and floats
- **Integers** count discrete things like coins collected
- **Floats** power physics: positions, velocities, and timestamps update in sub-pixel or sub-millisecond increments each frame for smooth motion

</div>

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

**What this code does:**
- `lastShotAt = -Infinity` is a clever trick — any real timestamp minus negative infinity is always a huge positive number, which is always greater than the 5-second cooldown, so the first shot is never blocked

```js
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8); // Kirby starts at 2.1 px/frame, slowly climbs to max 2.8
lebron.position.x += (dx / dist) * speed;                    // move Kirby toward player on the x-axis
lebron.position.y += (dy / dist) * speed;                    // move Kirby toward player on the y-axis
```

**What this code does:**
- Speed scales with survival time but is capped at 2.8 so the game stays winnable
- At the 20-second mark Kirby would reach exactly 2.7 px/frame, just under the cap

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="strings"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.2 — Strings

**Project Evidence Required:** Character names, sprite paths, game states.  
**Assessment Method:** Code review of string manipulation.

**What it is:**
- A **string** is a sequence of characters in quotes
- Strings are used here for asset IDs, file paths, and filtering game objects by name
- The same dot notation accesses them whether they come from a local object or an API response

</div>

```js
const sprite_src_player = getKirbyImageUrl('astro.png'); // builds the full URL to Astro's sprite PNG
const sprite_src_chaser = getKirbyImageUrl('kirby.png'); // builds the full URL to Kirby's sprite PNG
```

**What this code does:**
- These string filenames tell `getKirbyImageUrl` which image to load
- Change `'astro.png'` to any other filename and the engine immediately loads that sprite instead

```js
const coins = this.gameEnv.gameObjects.filter(
  (obj) => String(obj?.spriteData?.id || '').startsWith('coin_') // keep objects whose id begins with "coin_"
);
```

**What this code does:**
- `String()` safely converts the id to a string even if it's `null`
- `.startsWith('coin_')` finds all three coins (coin_1, coin_2, coin_3) in one filter — no separate coin list needed

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="booleans"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.3 — Booleans

**Project Evidence Required:** Flags such as `isJumping`, `isPaused`, `isVulnerable`.  
**Assessment Method:** Code review of boolean logic.

**What it is:**
- A **boolean** is either `true` or `false` — a single-bit decision flag
- Booleans guard state transitions and prevent a single event (one catch, one score submit) from triggering twice in the same round

</div>

```js
this.caught = false;                    // false = round is active; true = player was just caught
this.preGameLocked = true;              // true = intro screen is showing; blocks all movement and timers
this.scoreSubmittedThisRound = false;   // prevents sending the score to the leaderboard twice
this.levelCompleted = false;            // true once the 20-second win condition is met
this.completionTriggered = false;       // prevents completeLevel() from firing more than once
this.firstStealScrollTriggered = false; // ensures the "concept focus" event fires only on the first catch
```

**What this code does:**
- Six flags cover every major event in the round
- Each one acts as a one-way switch that trips once and stays tripped until `resetRound()` resets them all to `false`

```js
if (this.preGameLocked) return; // stop here — the Start button hasn't been clicked yet
```

**What this code does:**
- A single boolean check freezes LeBron, the survival timer, and all collision detection simultaneously
- One line does the work of six `if` checks

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="arrays"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.4 — Arrays

**Project Evidence Required:** Game object collections, level data.  
**Assessment Method:** Code review of array operations.

**What it is:**
- An **array** is an ordered list that can hold any number of values
- The game stores every live projectile in an array that grows via `push()` as shots are fired and shrinks via `splice()` as they expire
- The loop never needs to know how many there are in advance

</div>

```js
this.projectiles = [];                   // starts empty — no shots fired yet
...
this.projectiles.push(projectile);       // adds a new basketball to the live list when E is pressed
...
this.projectiles.splice(index, 1);       // removes one shot at 'index' when it expires or hits Kirby
```

**What this code does:**
- The backwards `for` loop in `updateProjectiles` (`i = length - 1; i >= 0; i -= 1`) is specifically because `splice` shifts every element after the removed one
- Iterating backwards means earlier indices are never skipped

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

**What this code does:**
- The engine iterates this array in order — the first entry (background) is drawn first so everything else appears on top of it

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="objects-json"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.5 — Objects (JSON)

**Project Evidence Required:** Configuration objects, sprite data.  
**Assessment Method:** Code review of object literals.

**What it is:**
- An **object literal** `{ key: value }` groups related values under one name
- JSON uses the same syntax — objects in code and API responses have identical structure and access patterns

</div>

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

**What this code does:**
- Every nested object here (`pixels`, `orientation`, `hitbox`, `keypress`) is accessed with dot notation in the constructor chain
- `data.hitbox.widthPercentage` is the same syntax whether `data` came from this object literal or a parsed API response

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="mathematical-operators"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.6 — Mathematical Operators

**Project Evidence Required:** Physics calculations such as gravity, velocity, collision.  
**Assessment Method:** Code review of `+`, `-`, `*`, `/` in physics.

**What it is:**
- Math operators power all physics
- `-` finds direction vectors; `/` normalizes them; `*` scales speed; `+` updates position
- `Math.hypot` computes straight-line distance without manually squaring and square-rooting

</div>

```js
const dx = player.position.x - lebron.position.x; // how far right the player is from Kirby
const dy = player.position.y - lebron.position.y; // how far down the player is from Kirby
const dist = Math.hypot(dx, dy);                   // straight-line distance between them
const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8); // speed climbs with time but caps at 2.8
lebron.position.x += (dx / dist) * speed;          // move Kirby one step toward player on x-axis
lebron.position.y += (dy / dist) * speed;          // move Kirby one step toward player on y-axis
```

**What this code does:**
- `dx / dist` normalizes the direction vector to length 1, then `* speed` rescales it to the correct step size
- This is what makes Kirby always chase at a consistent speed regardless of angle

```js
const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50)); // time score + coin bonus, rounded to whole number
```

**What this code does:**
- Survival time is worth 10 pts/sec and each coin is worth 50 pts
- `Math.round` converts the float to a clean integer before it gets sent to the leaderboard API

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="string-operations"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.7 — String Operations

**Project Evidence Required:** Path concatenation, text display.  
**Assessment Method:** Code review of template literals and concatenation.

**What it is:**
- **Template literals** (backtick strings with `${}`) replace concatenation for building display strings
- They embed any live JavaScript expression inline, so the HUD updates in one readable line instead of several joined strings

</div>

```js
this.timeHud.textContent =
  `Time: ${this.currentTime.toFixed(1)}s/${this.targetSurvivalSeconds}s | Best: ${this.bestTime.toFixed(1)}s | ` + // current and best time
  `Coins: ${this.getCoinsCollected()} | Best Coins: ${this.bestCoins}`;                                             // current and best coin count
```

**What this code does:**
- `.toFixed(1)` formats the float (e.g., `7.3472`) down to one decimal place (e.g., `7.3`) so the HUD stays narrow and readable
- This runs every frame so the timer is always live

```js
const basePath = (this.gameEnv?.path || '').replace(/\/$/, ''); // strip trailing slash if present
const aquaticUrl = `${basePath}/games/aquatic.html`;            // build the full URL for the Aquatic level
```

**What this code does:**
- `replace(/\/$/, '')` uses a regex to clean the path before the template literal appends the route
- Without it you'd get double slashes like `/games//aquatic.html`

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="boolean-expressions"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 4.8 — Boolean Expressions

**Project Evidence Required:** Compound conditions in game logic.  
**Assessment Method:** Code review of `&&`, `||`, `!`.

**What it is:**
- **Boolean operators**: `||` (OR) is true if either side is true; `&&` (AND) requires both sides
- Short-circuit evaluation means `||` stops as soon as it finds a truthy value — used here as a safe default to avoid null errors

</div>

```js
if (event.key.toLowerCase() !== 'e' || event.repeat) return; // block if wrong key OR key is being held down
if (this.preGameLocked || this.caught) return;                 // block if intro is showing OR player is caught
```

**What this code does:**
- Four separate reasons to cancel a shot, all checked in two lines
- If any single one is true, the shot is blocked
- Short-circuit means if the key isn't 'e', none of the other checks even run

```js
return (dx * dx + dy * dy) <= (projectile.radius * projectile.radius); // true = ball is close enough to count as a hit
```

**What this code does:**
- Uses squared distance instead of `Math.sqrt` (which is slower) — mathematically identical result
- No sprites involved, just pure coordinate math returning a yes/no answer

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 5 — Input / Output

</div>

---

<a id="keyboard-input"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 5.1 — Keyboard Input

**Project Evidence Required:** Arrow keys, space, WASD controls using event listeners.  
**Assessment Method:** Testing that key event handlers respond correctly.

**What it is:**
- An **event listener** registers a callback function that runs whenever a named browser event fires
- Because the game loop can't poll the keyboard directly, listeners write input state at the moment keys are pressed, and the loop reads that state every frame

</div>

```js
document.addEventListener('keydown', this.handleRestartKey); // 'R' key handler — resets the round
document.addEventListener('keydown', this.handleShootKey);   // 'E' key handler — fires a basketball
```

**What this code does:**
- Both are registered in `initialize()` and removed in `destroy()`
- If `destroy()` didn't call `removeEventListener`, these handlers would keep firing even after leaving the level

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

**What this code does:**
- `event.repeat` being `true` means the key is held down
- Without this check, holding E would fire dozens of shots per second instead of one clean shot per press

```js
keypress: { up: 87, left: 65, down: 83, right: 68 } // W=87, A=65, S=83, D=68 (WASD key codes)
```

**What this code does:**
- These are the ASCII key codes stored in the player's data object
- `Player.js` reads them in its constructor and maps them to movement directions

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="canvas-rendering"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 5.2 — Canvas Rendering

**Project Evidence Required:** Draw sprites, backgrounds, platforms using Canvas API.  
**Assessment Method:** Code review of `draw()` method implementations.

**What it is:**
- The **Canvas 2D API** is a stateful drawing surface — `ctx` is the 2D rendering context
- Every projectile is drawn onto its own offscreen canvas using `arc`, `fill`, and `quadraticCurveTo` to produce a recognizable basketball with seam lines

</div>

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

**What this code does:**
- This entire method is purely visual — it only paints pixels
- The collision radius used in `isCircleHittingObject` is a separate number (`projectile.radius = 10`) stored on the projectile object itself, not derived from anything drawn here
- The sprite and the hitbox are completely independent

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="gameenv-configuration"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 5.3 — GameEnv Configuration

**Project Evidence Required:** Set canvas size, difficulty levels, game settings.  
**Assessment Method:** Code review of `GameEnv.create()` and `GameSetup.js`.

**What it is:**
- The **level constructor** is the engine's single configuration hook
- Reading dimensions from `gameEnv` instead of hardcoding them makes the level adapt to any canvas size
- `this.classes` tells the engine which base objects to instantiate on startup

</div>

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

**What this code does:**
- Using percentages (`width * 0.12`) instead of fixed pixel values means spawn points scale correctly whether the game runs in a 400px embed or a 1200px full-screen window

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 6 — API, Async I/O & JSON

</div>

---

<a id="api-integration"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 6.1 — API Integration

**Project Evidence Required:** Implement Leaderboard API with POST/GET scores.  
**Assessment Method:** Code review of fetch calls with error handling.

**What it is:**
- A **REST API** uses HTTP verbs: POST sends data to the server; GET retrieves data
- The `Leaderboard` class handles the actual fetch calls
- `GameLevelBasketball` drives it by calling `submitScore()` with the computed round result

</div>

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

**What this code does:**
- `scoreSubmittedThisRound` is set to `true` before the async `submitScore` call — this is intentional
- If it were set after, a fast second trigger (e.g., lag causing two collision checks in one frame) could slip through before the first request finished

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="asynchronous-io"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 6.2 — Asynchronous I/O

**Project Evidence Required:** Use `async/await` or promises for API calls.  
**Assessment Method:** Code review of `async/await` or `.then()` chains.

**What it is:**
- **Asynchronous** code runs outside the current execution frame — it schedules work to happen later without blocking the game loop
- A Promise represents a value that will be available in the future
- `.catch()` handles any failure without crashing the caller

</div>

```js
this.leaderboard.submitScore(username, score, 'Basketball') // returns a Promise — doesn't block the frame
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // handles any network or server error
```

**What this code does:**
- The game loop keeps running at 60fps while this request is in flight
- The player can keep moving while the score is being saved in the background

```js
try {
  window.dispatchEvent(new CustomEvent('characters:concept-focus', { // fires a custom browser event
    detail: { level: 'basketball', trigger: 'first-steal' }          // payload: which level and why
  }));
} catch (err) {
  console.warn('Failed to emit basketball concept focus event:', err); // catch any dispatch error
}
```

**What this code does:**
- Custom events are asynchronous — listeners on the other end respond in their own time
- The game doesn't wait for them, keeping the frame rate smooth

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="json-parsing"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 6.3 — JSON Parsing

**Project Evidence Required:** Parse API responses such as leaderboard data and AI responses.  
**Assessment Method:** Code review of `JSON.parse()`, object destructuring.

**What it is:**
- **JSON** (JavaScript Object Notation) is the standard text format for API data
- Its syntax is identical to JS object literals — the same dot notation accesses properties in both
- `res.json()` parses the response body text into a plain JS object

</div>

```js
import Leaderboard from '@assets/js/GameEnginev1.1/essentials/Leaderboard.js'; // import the leaderboard helper class
...
this.leaderboard.submitScore(username, score, 'Basketball') // sends username+score as JSON to the API
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // handles parse or network failures
```

**What this code does:**
- The `Leaderboard` class internally calls `res.json()` to parse the server's JSON response
- `GameLevelBasketball` only needs to call `submitScore()` and handle errors; the parsing is abstracted away

```js
detail: { level: 'basketball', trigger: 'first-steal' } // JSON-serializable payload on a custom event
```

**What this code does:**
- This object literal follows JSON structure
- Any listener reads `event.detail.level` and `event.detail.trigger` with the same dot notation used to read parsed API responses

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 7 — Debugging

</div>

---

<a id="console-debugging"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 7.1 — Console Debugging

**Project Evidence Required:** Use `console.log` to track game state, variables, method calls.  
**Assessment Method:** Code review of strategic logging in update/collision methods.

**What it is:**
- **Console logging** traces execution by printing values at key moments — game start, collision events, state transitions
- Never log inside the animation loop (`update()` runs 60×/sec and will flood the console in seconds)
- Place logs at one-time state transitions instead

</div>

```js
// Log once when the round resets — confirm all state was cleared correctly
resetRound() {
  console.log('[Basketball] Round reset — caught=false, time=0, coins=0'); // one-time log at reset
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

**What this code does:**
- All logs use a `[Basketball]` prefix
- In DevTools you can type `[Basketball]` in the console filter to see only this level's messages alongside the engine's own logs

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="hit-box-visualization"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 7.2 — Hit Box Visualization

**Project Evidence Required:** Draw/visualize collision boundaries to refine detection.  
**Assessment Method:** Demo — toggle hit box display, adjust collision rectangles.

**What it is:**
- **Hit box visualization** exposes the invisible collision geometry
- Reveals whether the hit zone actually matches the visible sprite

</div>

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

**What this code does:**
- Shrinking each hitbox by 20% on every side makes near-misses feel fair — the sprite images overlap slightly before a catch registers, which matches player intuition better than pixel-perfect collision
- To visualize during debugging, call `ctx.strokeRect(rect.left, rect.top, rect.right - rect.left, rect.bottom - rect.top)` using the values returned by `getHitboxRect`

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="source-debugging"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 7.3 — Source-Level Debugging

**Project Evidence Required:** Set breakpoints in DevTools, step through code execution.  
**Assessment Method:** Demo — use Sources tab to pause and inspect code flow.

**What it is:**
- A **breakpoint** pauses execution at a specific line so you can inspect every live variable at that exact moment

**Steps:**
1. Open DevTools → **Sources** tab
2. Navigate to `assets/js/projects/kirby-minigames/levels/GameLevelBasketball.js`
3. Click the line number next to `const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8);` inside `update()`
4. Move Astro close to LeBron — execution pauses on that line
5. Inspect `player.position`, `lebron.position`, `dx`, `dy`, `dist`, and `speed` in the **Scope** panel on the right
6. Press **F10** (step over) to advance one line at a time and watch `lebron.position.x` and `lebron.position.y` update

</div>

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

**What this code does:**
- Setting a breakpoint on the collision check lets you open the Scope panel and read both hitbox rectangles side by side
- You can confirm the four overlap conditions are all true before `this.caught` flips

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="network-debugging"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 7.4 — Network Debugging

**Project Evidence Required:** Examine Network tab for API calls, CORS errors, response status.  
**Assessment Method:** Demo — inspect fetch requests, response data, error messages.

**What it is:**
- The **Network tab** records every HTTP request the page makes

**Steps:**
1. Open DevTools → **Network** tab → filter by **Fetch/XHR**
2. Get caught by LeBron to trigger `submitRoundScore()`
3. **Headers** tab: confirm POST to the leaderboard endpoint, `Content-Type: application/json`
4. **Payload** tab: confirm JSON body contains `username`, `score`, and `'Basketball'` as the game name
5. **Response** tab: confirm a success response or read the server error message
6. CORS error? `Access-Control-Allow-Origin` is missing from response headers — add it server-side

</div>

```js
// This is the call that appears in the Network tab
this.leaderboard.submitScore(username, score, 'Basketball') // the POST that shows up under Fetch/XHR
  .catch((err) => console.warn('Leaderboard score submit failed:', err)); // network errors land here

// The score being posted — visible in the Payload tab
const score = Math.round((this.currentTime * 10) + (this.getCoinsCollected() * 50)); // computed value that gets sent
const username = (this.gameEnv?.game?.uid && String(this.gameEnv.game.uid)) || 'Player'; // sender identity
```

**What this code does:**
- If the request never appears at all in the Network tab, `scoreSubmittedThisRound` is already `true`
- Check it in the Sources panel's Scope view during a breakpoint at `submitRoundScore`

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="application-debugging"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 7.5 — Application / Storage Debugging

**Project Evidence Required:** Examine cookies, localStorage, session data for login/state.  
**Assessment Method:** Demo — Application tab inspection of stored data.

**What it is:**
- The **Application tab** exposes cookies, localStorage, and sessionStorage
- `GameLevelBasketball` uses `localStorage` to persist best time and best coins between sessions

</div>

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

**What this code does:**
- Open DevTools → **Application** tab → **Local Storage** → select the site origin
- Confirm `basketball_best_time` and `basketball_best_coins` appear after completing a round
- If `bestTime` shows 0 at the start of a new session, check here first — the key may be missing or stored as `NaN`

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="element-inspection"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 7.6 — Element Inspection

**Project Evidence Required:** Use Element Viewer to inspect canvas, DOM elements, styles.  
**Assessment Method:** Demo — inspect element properties and game object state.

**What it is:**
- The **Element Inspector** shows the live DOM tree
- Because `GameLevelBasketball` creates its HUD elements and projectile canvases dynamically at runtime, this is the only way to confirm they were appended to the right parent with the correct CSS

</div>

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

**What this code does:**
- Press `E` to fire a shot — a new `<canvas>` element should appear inside the game container in the Elements panel and disappear the moment it expires or hits Kirby
- If HUD elements are missing, check that `container` isn't `null` — the method returns early if both `gameEnv.container` and `gameEnv.gameContainer` are undefined

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0 10px 0;">

## 8 — Testing & Verification

</div>

---

<a id="gameplay-testing"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 8.1 — Gameplay Testing

**Project Evidence Required:** Test level completion, character interactions, collision detection.  
**Assessment Method:** Live demo — play through level without critical bugs.

**What it is:**
- **Gameplay testing** verifies that the game behaves correctly as a player — not just that it compiles

</div>

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

**Three testable gameplay checks:**
- Survive 20 seconds → win condition triggers
- Get caught by Kirby → caught state, score submit, reset flow
- Shoot Kirby with E → stun applied, ball removed

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="integration-testing"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 8.2 — Integration Testing

**Project Evidence Required:** Test API integration (Leaderboard, NPC AI) with live backend.  
**Assessment Method:** Demo — successful score saving and AI responses.

**What it is:**
- **Integration testing** checks that two separate systems — the game and the leaderboard server — work correctly together

</div>

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

**What this code does:**
- To test the error path: stop the backend server, get caught, and confirm the `.catch()` fires a `console.warn` instead of an uncaught error
- The game should keep running normally with no visible crash

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<a id="api-error-handling"></a>
<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:10px 0;">

### 8.3 — API Error Handling

**Project Evidence Required:** Try/catch blocks for API calls and network error handling.  
**Assessment Method:** Code review of error handling for fetch failures.

**What it is:**
- A **try/catch** block wraps risky code so any failure — network loss, CORS block, or bad response — is caught in one place

</div>

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

**What this code does:**
- `try/catch` catches synchronous failures (event dispatch errors)
- `.catch()` catches asynchronous failures (network errors, server rejections)
- Both log warnings instead of throwing, so the game loop keeps running at full speed even when the backend is down

<div style="text-align:right;"><a href="#top" style="color:#4a9eff; font-size:13px;">▲ Top</a></div>

---

<div style="background:#020f22; border:1px solid #0d4a8a; border-radius:10px; padding:18px 26px; margin:28px 0;">

## ✅ CS111 Completion Summary

| ✅ | Objective | Section |
|---|---|---|
| ✅ | Writing classes (2+ custom classes) | Section 1.1 |
| ✅ | Methods with 2+ parameters | Section 2.1 |
| ✅ | Instantiation & objects | Section 1.5 |
| ✅ | Inheritance (extends) | Section 1.2 |
| ✅ | Method overriding | Section 1.3 |
| ✅ | Constructor chaining (super) | Section 1.4 |
| ✅ | Conditionals (if/else) | Section 3.1 |
| ✅ | Nested conditions | Section 3.2 |
| ✅ | Numbers | Section 4.1 |
| ✅ | Strings & template literals | Section 4.2 |
| ✅ | Booleans | Section 4.3 |
| ✅ | Arrays | Section 4.4 |
| ✅ | Objects / JSON | Section 4.5 |
| ✅ | Math operators | Section 4.6 |
| ✅ | String operations | Section 4.7 |
| ✅ | Boolean expressions | Section 4.8 |
| ✅ | Keyboard input | Section 5.1 |
| ✅ | Canvas rendering | Section 5.2 |
| ✅ | GameEnv configuration | Section 5.3 |
| ✅ | API integration | Section 6.1 |
| ✅ | Async I/O (async/await) | Section 6.2 |
| ✅ | JSON parsing | Section 6.3 |
| ✅ | Debugging (6 DevTools areas) | Section 7 |
| ✅ | Testing checklist | Section 8 |

</div>

**What makes this file especially strong:**
- Every concept connects to something the player can actually see and test
- The physics move LeBron
- The booleans freeze the round
- The canvas draws the basketball
- The API saves the score
