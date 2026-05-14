---
layout: post
title: Game Runner Sandbox
description: Blank editable game runner for experimenting with GameEngine code.
permalink: /game-runner-sandbox
codemirror: true
hide: true
toc: false
---

## Blank Game Runner

Use this page to paste in game code, edit it live, and test ideas in the browser.

{% capture sandbox_challenge %}
Paste GameEngine code into the editor, then press Start. This blank runner is meant for experiments, debugging, and quick demos.
{% endcapture %}

{% capture sandbox_code %}
import GameControl from '/assets/js/GameEnginev1.1/essentials/GameControl.js';
import GameLevelWater from '/assets/js/GameEnginev1.1/GameLevelWater.js';

export const gameLevelClasses = [GameLevelWater];
export { GameControl };
{% endcapture %}

{% include runners/game.html
   runner_id="lance-sandbox"
   challenge=sandbox_challenge
   code=sandbox_code
   height="420px"
   editor_height="320px"
%}
