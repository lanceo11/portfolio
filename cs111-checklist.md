---
layout: post
title: CS 111 Checklist
description: Completed checklist for CS 111 course alignment rubric evidence.
permalink: /cs111-checklist
hide: true
toc: true
---

## CS 111 Course Alignment Rubric

This chart focuses on how `GameLevelBasketball.js` demonstrates each concept in a concrete way.

| Concept | How I completed it in `GameLevelBasketball` | Complete |
|---|---|---|
| [Writing Classes](/basketball-cs111-breakdown#writing-classes) | I created the custom `GameLevelBasketball` class to control the full basketball minigame. | `✓` |
| [Methods & Parameters](/basketball-cs111-breakdown#methods-and-parameters) | I used methods like `updateProjectiles(now, lebron)`, `spawnProjectileFromPlayer(player, now)`, and `isHitboxCollision(a, b)`. | `✓` |
| [Instantiation & Objects](/basketball-cs111-breakdown#instantiation-and-objects) | I instantiate the level objects through `this.classes` with `GameEnvBackground`, `Player`, `Npc`, `Coin`, and `Barrier`. | `✓` |
| [Inheritance (Basic)](/basketball-cs111-breakdown#inheritance-basic) | The level uses engine classes that extend the game object hierarchy, including `Player`, `Npc`, `Coin`, and `Barrier`. | `✓` |
| [Method Overriding](/basketball-cs111-breakdown#method-overriding) | The level defines its own lifecycle methods such as `initialize()`, `update()`, and `destroy()` to customize runtime behavior. | `✓` |
| [Constructor Chaining](/basketball-cs111-breakdown#constructor-chaining) | The basketball level composes engine objects that use constructor chaining internally when created from `this.classes`. | `✓` |
| [Iteration](/basketball-cs111-breakdown#iteration) | I loop through projectile arrays in `updateProjectiles()` and through filtered coins in reset/spawn logic. | `✓` |
| [Conditionals](/basketball-cs111-breakdown#conditionals) | I use `if` conditions for caught state, pregame lock, stun, projectile cooldowns, collision checks, and win conditions. | `✓` |
| [Nested Conditions](/basketball-cs111-breakdown#nested-conditions) | The `update()` method layers multiple conditions like timer checks, caught state, stun state, and collision outcomes. | `✓` |
| [Numbers](/basketball-cs111-breakdown#numbers) | The code uses positions, cooldowns, speed, survival time, projectile radius, and score math. | `✓` |
| [Strings](/basketball-cs111-breakdown#strings) | The code uses sprite IDs, dialogue text, HUD text, localStorage keys, and direction names. | `✓` |
| [Booleans](/basketball-cs111-breakdown#booleans) | State flags include `caught`, `preGameLocked`, `scoreSubmittedThisRound`, `levelCompleted`, and `completionTriggered`. | `✓` |
| [Arrays](/basketball-cs111-breakdown#arrays) | Arrays are used for `this.classes`, `this.projectiles`, and filtered collections of coins or game objects. | `✓` |
| [Objects (JSON)](/basketball-cs111-breakdown#objects-json) | Player, chaser, barrier, and coin setup all use object literals for configuration. | `✓` |
| [Mathematical Operators](/basketball-cs111-breakdown#mathematical-operators) | The chase system uses subtraction, division, multiplication, and addition with `dx`, `dy`, `dist`, and speed scaling. | `✓` |
| [String Operations](/basketball-cs111-breakdown#string-operations) | Template literals build HUD text and dynamic CSS positions like `` `${projectile.x - projectile.radius}px` ``. | `✓` |
| [Boolean Expressions](/basketball-cs111-breakdown#boolean-expressions) | Compound comparisons and `&&` logic are used in hitbox collision and input/state checks. | `✓` |
| [Keyboard Input](/basketball-cs111-breakdown#keyboard-input) | `keydown` listeners handle restart with `R` and shooting with `E`. | `✓` |
| [Canvas Rendering](/basketball-cs111-breakdown#canvas-rendering) | `drawProjectileSprite()` renders the basketball projectile directly on a canvas with arc and curve drawing commands. | `✓` |
| [GameEnv Configuration](/basketball-cs111-breakdown#gameenv-configuration) | The level uses `gameEnv.innerWidth`, `gameEnv.innerHeight`, `gameEnv.path`, `gameEnv.container`, and `gameEnv.stats`. | `✓` |
| [API Integration](/basketball-cs111-breakdown#api-integration) | The level submits a score through the leaderboard system using `submitScore(...)`. | `✓` |
| [Asynchronous I/O](/basketball-cs111-breakdown#asynchronous-io) | Score submission is handled as an async promise with `.catch(...)`. | `✓` |
| [JSON Parsing](/basketball-cs111-breakdown#json-parsing) | The level works with structured game config objects and stats objects; API response handling is delegated through the leaderboard module. | `✓` |
| [Code Comments](/basketball-cs111-breakdown#code-comments) | The file includes explanatory comments around chase behavior, fairness, and hitbox alignment. | `✓` |
| [Mini-Lesson Documentation](/basketball-cs111-breakdown#mini-lesson-documentation) | I created the Basketball concepts breakdown page and linked it from my portfolio. | `✓` |
| [Code Highlights](/basketball-cs111-breakdown#code-highlights) | The portfolio now includes highlighted snippets and explanations for Basketball logic. | `✓` |
| [Console Debugging](/basketball-cs111-breakdown#console-debugging) | The Kirby project includes strategic logs in the broader lesson flow and can be debugged through stateful methods in this level. | `✓` |
| [Hit Box Visualization](/basketball-cs111-breakdown#hit-box-visualization) | The level uses custom hitbox rectangle logic in `getHitboxRect()` and `isHitboxCollision()` to refine collisions. | `✓` |
| [Source-Level Debugging](/basketball-cs111-breakdown#source-level-debugging) | The level is broken into named methods, which makes it straightforward to inspect with DevTools breakpoints. | `✓` |
| [Network Debugging](/basketball-cs111-breakdown#network-debugging) | Leaderboard submission can be inspected in the Network tab when scores are posted. | `✓` |
| [Application Debugging](/basketball-cs111-breakdown#application-debugging) | `localStorage` is used for best time and best coins with `loadBestTime()`, `saveBestTime()`, `loadBestCoins()`, and `saveBestCoins()`. | `✓` |
| [Element Inspection](/basketball-cs111-breakdown#element-inspection) | The HUD and projectile canvases are created dynamically in the DOM, making them inspectable in the Elements panel. | `✓` |
| [Gameplay Testing](/basketball-cs111-breakdown#gameplay-testing) | The level supports repeated playthroughs with reset, caught, and completion flows. | `✓` |
| [Integration Testing](/basketball-cs111-breakdown#integration-testing) | Leaderboard behavior can be tested by playing the level and submitting a score. | `✓` |
| [API Error Handling](/basketball-cs111-breakdown#api-error-handling) | `submitRoundScore()` catches failed leaderboard submissions with `.catch((err) => console.warn(...))`. | `✓` |

# GameLevelBasketball Final Project Rubric Alignment

| Concept / Rubric Requirement | How I Applied It in My Game | Actual Code Example |
|---|---|---|
| Object-Oriented Programming (OOP) | I created a custom game level class that controls the entire basketball level logic, state management, collisions, projectiles, HUDs, and gameplay systems. | ```js class GameLevelBasketball { constructor(gameEnv) { this.gameEnv = gameEnv; } } ``` |
| Classes Extending Base Classes | I used multiple classes that extend Game Engine base classes like Player, Npc, Coin, Barrier, and GameEnvBackground. | ```js import Player from '@assets/js/GameEnginev1.1/essentials/Player.js'; import Npc from '@assets/js/GameEnginev1.1/essentials/Npc.js'; ``` |
| Instantiation & Objects | I instantiated game objects using object literals inside the GameLevel configuration array. | ```js this.classes = [ { class: Player, data: sprite_data_player }, { class: Npc, data: sprite_data_chaser }, { class: Coin, data: coin_1 } ]; ``` |
| Inheritance | My game uses inheritance through Game Engine classes like Player, Npc, Coin, and Barrier which inherit from parent game object classes. | ```js import Barrier from '@assets/js/GameEnginev1.1/essentials/Barrier.js'; ``` |
| Methods & Parameters | I created methods with parameters to control gameplay systems like collisions, projectiles, and spawning. | ```js isHitboxCollision(a, b) { const ar = this.getHitboxRect(a); const br = this.getHitboxRect(b); } ``` |
| Method Overriding | The imported classes override engine lifecycle methods like update(), draw(), and handleCollision() internally. My GameLevelBasketball also manages update() behavior for the level. | ```js update() { const player = this.findById('BasketballPlayer'); const lebron = this.findById('LeBron'); } ``` |
| Constructor Chaining | The imported engine classes use constructor chaining with super(data, gameEnv) when extending engine objects. My GameLevelBasketball constructor initializes all level data. | ```js constructor(gameEnv) { this.gameEnv = gameEnv; } ``` |
| Numbers Data Type | I used numbers for positions, timers, projectile speed, score systems, cooldowns, and movement calculations. | ```js this.projectileSpeed = 9; this.shootCooldownMs = 5000; this.targetSurvivalSeconds = 20; ``` |
| Strings Data Type | I used strings for IDs, messages, directions, dialogue, and leaderboard names. | ```js id: 'BasketballPlayer', greeting: 'Ball handler ready.' ``` |
| Booleans Data Type | I used booleans to manage game states like caught, levelCompleted, preGameLocked, and completionTriggered. | ```js this.caught = false; this.preGameLocked = true; this.levelCompleted = false; ``` |
| Arrays Data Type | I used arrays for game objects, projectiles, dialogue lists, and level configuration. | ```js this.projectiles = []; dialogues: ['LeBron is in the gym.'] ``` |
| JSON Objects / Object Literals | I used object literals for sprites, hitboxes, positions, barriers, and coins. | ```js const coin_1 = { id: 'coin_1', INIT_POSITION: { x: 200, y: 300 }, value: 1 }; ``` |
| Mathematical Operators | I used math operators for movement physics, projectile motion, collision detection, and scaling difficulty. | ```js const dx = player.position.x - lebron.position.x; const dy = player.position.y - lebron.position.y; ``` |
| Boolean Expressions | I used boolean logic for cooldowns, collisions, and game conditions. | ```js if (event.key.toLowerCase() !== 'e' || event.repeat) return; ``` |
| String Operations | I used template literals and string concatenation for HUD displays and navigation URLs. | ```js this.timeHud.textContent = `Time: ${this.currentTime.toFixed(1)}s`; ``` |
| Conditionals | I used if statements throughout the game to manage collisions, cooldowns, resets, and win conditions. | ```js if (this.currentTime >= this.targetSurvivalSeconds) { this.completeLevel(); } ``` |
| Nested Conditions | I used nested conditionals for more advanced gameplay logic like collisions combined with stun states and cooldown systems. | ```js if (lebron && this.isCircleHittingObject(projectile, lebron)) { this.lebronStunUntil = now + this.lebronStunDurationMs; } ``` |
| Iteration / Loops | I used loops to update projectiles, coins, and game object arrays. | ```js for (let i = this.projectiles.length - 1; i >= 0; i -= 1) { projectile.x += projectile.vx; } ``` |
| Keyboard Input | I implemented keyboard controls for movement, restarting, and shooting basketball projectiles. | ```js document.addEventListener('keydown', this.handleShootKey); ``` |
| Canvas Rendering | I used the Canvas API to render custom basketball projectiles. | ```js const ctx = projectile.canvas.getContext('2d'); ctx.arc(cx, cy, r, 0, Math.PI * 2); ``` |
| GameEnv Configuration | I configured game environment dimensions, object spawning, and level setup using GameEnv. | ```js const width = gameEnv.innerWidth; const height = gameEnv.innerHeight; ``` |
| API Integration | I integrated the Leaderboard API to submit player scores to the backend. | ```js this.leaderboard.submitScore(username, score, 'Basketball') ``` |
| Asynchronous I/O | I used asynchronous promises with .catch() for leaderboard API requests. | ```js this.leaderboard.submitScore(username, score, 'Basketball').catch((err) => console.warn(err)); ``` |
| JSON Parsing / Object Access | I accessed and modified JSON-style game objects throughout the level. | ```js this.gameEnv.stats.coinsCollected = 0; ``` |
| Single Responsibility Principle | I separated gameplay systems into focused methods like updateProjectiles(), createHud(), completeLevel(), and resetRound(). | ```js updateProjectiles(now, lebron) { } createHud() { } resetRound() { } ``` |
| State Management | I managed game states like caught, stunned, completed, pregame lock, and timers. | ```js this.caught = true; this.lebronStunUntil = now + this.lebronStunDurationMs; ``` |
| Collision Detection | I created hitbox collision systems for player-vs-enemy and projectile-vs-enemy collisions. | ```js return ( ar.left < br.right && ar.right > br.left ); ``` |
| Hitbox Visualization Logic | I customized hitboxes using width and height percentages for accurate collision balancing. | ```js hitbox: { widthPercentage: 0.45, heightPercentage: 0.5 } ``` |
| Physics & Movement | I implemented directional movement and normalized chase movement using vector math. | ```js lebron.position.x += (dx / dist) * speed; ``` |
| Enemy AI / Chase Logic | I programmed LeBron to dynamically chase the player based on relative position. | ```js const dx = player.position.x - lebron.position.x; ``` |
| Dynamic Difficulty Scaling | I increased enemy speed over time to make survival progressively harder. | ```js const speed = Math.min(2.1 + this.currentTime * 0.03, 2.8); ``` |
| Projectile System | I created a custom projectile system where the player can shoot basketballs to stun the enemy. | ```js this.spawnProjectileFromPlayer(player, now); ``` |
| Cooldown System | I added cooldown timing to prevent projectile spam. | ```js if (now - this.lastShotAt < this.shootCooldownMs) return; ``` |
| DOM Manipulation | I dynamically created HUD elements and updated their styles during gameplay. | ```js this.timeHud = document.createElement('div'); ``` |
| HUD / UI System | I created a timer HUD and message system that updates during gameplay. | ```js this.timeHud.textContent = `Time: ${this.currentTime.toFixed(1)}s`; ``` |
| Local Storage Persistence | I used localStorage to save best survival time and coin records between sessions. | ```js localStorage.setItem('basketball_best_time', String(this.bestTime)); ``` |
| Event Systems | I used CustomEvent dispatching for level completion and gameplay progression triggers. | ```js window.dispatchEvent(new CustomEvent('characters:level-complete')); ``` |
| Dialogue System | I implemented an intro dialogue system before gameplay starts. | ```js this.introDialogue = new DialogueSystem({ dialogues: ['Foreign explorer?'] }); ``` |
| Randomization | I randomized coin spawning positions using Math.random(). | ```js coin.position.x = xMin + Math.random() * (xMax - xMin); ``` |
| Error Handling | I used try/catch and promise catch statements to prevent crashes from API or browser errors. | ```js try { localStorage.setItem(...) } catch (_) {} ``` |
| Debugging Practices | I used console warnings and logs for debugging events and API issues. | ```js console.warn('Leaderboard score submit failed:', err); ``` |
| Gameplay Testing | I tested player movement, collisions, timers, stun mechanics, and leaderboard integration to ensure the level was fully playable. | ```js if (this.isHitboxCollision(player, lebron)) { this.caught = true; } ``` |
| Complete Custom Level | I built a complete basketball survival game level with enemies, coins, barriers, UI, AI, projectiles, collisions, and win conditions. | ```js export default GameLevelBasketball; ``` |
