---
layout: opencs
title: Background with Object
description: Use JavaScript to have an in motion background.
sprite: /images/platformer/sprites/flying-ufo.png
background: /images/platformer/backgrounds/alien_planet2.jpg
permalink: /background
---

<canvas id="world"></canvas>

<script>
  // Get canvas and context
  const canvas = document.getElementById("world");
  const ctx = canvas.getContext('2d');

  // Create image objects for background and sprite
  const backgroundImg = new Image();
  const spriteImg = new Image();

  // Hardcoded image paths (update if needed)
  backgroundImg.src = '/images/platformer/backgrounds/alien_planet2.jpg';
  spriteImg.src = '/images/platformer/sprites/flying-ufo.png';

  // Wait for both images to load before starting
  let bgLoaded = false, spriteLoaded = false;
  backgroundImg.onload = function() {
    bgLoaded = true;
    if (bgLoaded && spriteLoaded) startGameWorld();
  };
  spriteImg.onload = function() {
    spriteLoaded = true;
    if (bgLoaded && spriteLoaded) startGameWorld();
  };

  function startGameWorld() {
    // Base class for all game objects
    class GameObject {
      constructor(image, width, height, x = 0, y = 0, speedRatio = 0) {
        this.image = image;
        this.width = width;
        this.height = height;
        this.x = x;
        this.y = y;
        this.speedRatio = speedRatio;
        this.speed = GameWorld.gameSpeed * this.speedRatio;
      }
      update() {}
      draw(ctx) {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
      }
    }

    // Background class, scrolls horizontally
    class Background extends GameObject {
      constructor(image, gameWorld) {
        super(image, gameWorld.width, gameWorld.height, 0, 0, 0.1);
      }
      update() {
        this.x = (this.x - this.speed) % this.width;
      }
      draw(ctx) {
        ctx.drawImage(this.image, this.x, this.y, this.width, this.height);
        ctx.drawImage(this.image, this.x + this.width, this.y, this.width, this.height);
      }
    }

    // Player class, moves in a sine wave pattern
    class Player extends GameObject {
      constructor(image, gameWorld) {
        const width = image.naturalWidth / 2;
        const height = image.naturalHeight / 2;
        const x = (gameWorld.width - width) / 2;
        const y = (gameWorld.height - height) / 2;
        super(image, width, height, x, y);
        this.baseY = y;
        this.baseX = x;
        this.frame = 0;
      }
      update() {
        this.y = this.baseY + Math.sin(this.frame * 0.05) * 20;
        this.x = this.baseX + Math.sin(this.frame * 0.05) * 20;
        this.frame++;
      }
    }

    // Main game world class
    class GameWorld {
      static gameSpeed = 5;
      constructor(backgroundImg, spriteImg) {
        this.canvas = document.getElementById("world");
        this.ctx = this.canvas.getContext('2d');
        this.width = window.innerWidth;
        this.height = window.innerHeight;
        this.canvas.width = this.width;
        this.canvas.height = this.height;
        this.gameObjects = [
          new Background(backgroundImg, this),
          new Player(spriteImg, this)
        ];
      }
      gameLoop() {
        this.ctx.clearRect(0, 0, this.width, this.height);
        for (const obj of this.gameObjects) {
          obj.update();
          obj.draw(this.ctx);
        }
        requestAnimationFrame(this.gameLoop.bind(this));
      }
      start() {
        this.gameLoop();
      }
    }

    const world = new GameWorld(backgroundImg, spriteImg);
    world.start();
  }
</script>
