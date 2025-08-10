<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" />
<title>Speed Master - Realistic Cars Dodge</title>
<style>
  body, html {
    margin:0; padding:0; background:#111; color:#eee; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    overflow: hidden;
    user-select:none;
  }
  #gameCanvas {
    display:block;
    margin: 10px auto;
    background: linear-gradient(to top, #444, #222);
    border-radius: 12px;
    box-shadow: 0 0 15px #0f0;
    touch-action:none;
  }
  #startScreen, #gameOverScreen {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.9);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 10;
    color: #0f0;
    text-align: center;
  }
  input, button {
    padding: 12px 18px;
    font-size: 1.2em;
    border-radius: 10px;
    border: none;
    margin-top: 15px;
    background: #0a0;
    color: #eee;
    font-weight: bold;
    cursor: pointer;
    user-select:none;
    transition: background 0.3s ease;
  }
  input:focus, button:hover {
    background: #0f0;
    color: #000;
    outline: none;
  }
  #leaderboardList {
    margin-top: 25px;
    width: 280px;
    max-height: 160px;
    overflow-y: auto;
    background: #111;
    border: 2px solid #0f0;
    border-radius: 10px;
    padding: 10px;
    text-align: left;
  }
  #leaderboardList li {
    margin: 4px 0;
    font-weight: bold;
  }
  #scoreDisplay {
    position: fixed;
    top: 5px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 1.8em;
    font-weight: bold;
    color: #0f0;
    user-select:none;
    z-index: 5;
    text-shadow: 0 0 5px #0f0;
  }
  #instructions {
    position: fixed;
    bottom: 10px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 1em;
    color: #0f0a0a;
    user-select:none;
  }
</style>
</head>
<body>

<div id="startScreen">
  <h1>Speed Master</h1>
  <label for="playerName">Enter your name:</label>
  <input id="playerName" type="text" maxlength="12" placeholder="Your name" autocomplete="off" />
  <button id="startBtn">Start Race</button>

  <h2>Leaderboard</h2>
  <ol id="leaderboardList"></ol>
  <p style="margin-top:10px; font-size:0.85em; color:#080;">Tap left/right side to move your car</p>
</div>

<canvas id="gameCanvas" width="320" height="480" tabindex="0"></canvas>
<div id="scoreDisplay" style="display:none;">Score: 0</div>
<div id="instructions">Tap left or right side to move</div>

<div id="gameOverScreen" style="display:none;">
  <h2>Game Over!</h2>
  <p id="finalScore"></p>
  <button id="restartBtn">Play Again</button>
  <h3>Leaderboard</h3>
  <ol id="gameOverLeaderboard"></ol>
</div>

<audio id="bgMusic" loop src="https://cdn.pixabay.com/download/audio/2022/03/29/audio_ebff882d41.mp3?filename=retro-game-loop-10870.mp3" crossorigin="anonymous"></audio>
<audio id="crashSound" src="https://cdn.pixabay.com/download/audio/2022/03/23/audio_75bb4940e7.mp3?filename=explosion-2-6590.mp3" crossorigin="anonymous"></audio>

<script>
// --- Car sprites as base64 images (small realistic-ish car icons, you can replace these with your own images or better sprites later)
const carSprites = [
  // Basic small cars (score 0+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAtCAYAAAANf0eBAAAAAXNSR0IArs4c6QAAAMFJREFUWAnN1LENwjAQRNFkxKZI3Eg1rNJcJcvshW8QOsDcshFyCErbxqpyQmbqDjmxG8Mfc7v6vzEq0DoMTv7+R46ZP3s2bOYKwbi5e6L6kR3ZPKPnoPS08zYQ+wc89XtGj9IgHW12GghPMDRCksThhpWIOZlIFpN2n4bCv5dWko6eYy3+wrCH3xUuDg+XfoAv0YXP+Bx/kHYDj+RH+H2nC4TQ/A6gXj6Vx0RJ6E0P9AjCQ7pt1V1BgAAAABJRU5ErkJggg==",
  // Slightly better cars (score 10+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAvCAYAAACxoYX3AAAAXElEQVR42u3SsQ0AMQwE0PTQnDjZr3yHgEvQHX7f0LlF0kKi1GgSl6u7mH/fx3djraaq7jM6c3Wb2Jp2NN+q8YXTpF8k3jY1R48DbtJm8vB7tHtgZDAAAAAElFTkSuQmCC",
  // More realistic cars (score 25+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAArCAYAAAAlxK4kAAAAXklEQVR42u3VwQ0AMAgDQPa/6ccNDsogK8FvUVZ+Xk8CzqEweuS3h5x+XzDGOnjL9QFi7eq6z8w+V6rYyYzD3XyJIQAAAAASUVORK5CYII=",
  // Realistic sport car (score 50+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAtCAYAAAC8kvlXAAAAVUlEQVR42u3VsQ0AMAwEsD3F3aT8SnkSQKDXcd+ZjPgsLzERwJbMLJp+HjMF6H+gLQ8SjeZBvT9gMwBUjpZoDlf5NJoZ6DCV1K1KQhvJuQkqpHQa01C1LUKw2YqNfvRw2ryq9AoykgYAAABl1hKwtxv25FAAAAABJRU5ErkJggg==",
  // Realistic luxury car (score 75+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAvCAYAAACQhZ7zAAAAVklEQVR42u3VsQ0AMAwEsD3F3aT8SnkSQKDXcd+ZjPgsLzERwJbMLJp+HjMF6H+gLQ8SjeZBvT9gMwBUjpZoDlf5NJoZ6DCV1K1KQhvJuQkqpHQa01C1LUKw2YqNfvRw2ryq9AoykgYAAABm0Rn2nXzN3QAAAABJRU5ErkJggg==",
  // Realistic racing car (score 150+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAArCAYAAAAlxK4kAAAAYklEQVR42u3UQQ0AMAwDwZ3/3Uq6YAEsUqJu7ngHhBbpG5ty7nBwN8wA3rmCuc/6juu76ZuD1+ZbA4x6MAAAAAElFTkSuQmCC",
  // Sportscar with spoilers (score 300+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAsCAYAAABMEdGqAAAAXklEQVR42u3VsQ0AMAwEsD3F3aT8SnkSQKDXcd+ZjPgsLzERwJbMLJp+HjMF6H+gLQ8SjeZBvT9gMwBUjpZoDlf5NJoZ6DCV1K1KQhvJuQkqpHQa01C1LUKw2YqNfvRw2ryq9AoykgYAAABM0xjC4x60AAAAASUVORK5CYII=",
  // Super realistic car (score 500+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAtCAYAAAC8kvlXAAAAVUlEQVR42u3VsQ0AMAwEsD3F3aT8SnkSQKDXcd+ZjPgsLzERwJbMLJp+HjMF6H+gLQ8SjeZBvT9gMwBUjpZoDlf5NJoZ6DCV1K1KQhvJuQkqpHQa01C1LUKw2YqNfvRw2ryq9AoykgYAAABl1hKwtxv25FAAAAABJRU5ErkJggg==",
  // Ultimate coolest car (score 1000+)
  "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACgAAAAvCAYAAACQhZ7zAAAAVklEQVR42u3VsQ0AMAwEsD3F3aT8SnkSQKDXcd+ZjPgsLzERwJbMLJp+HjMF6H+gLQ8SjeZBvT9gMwBUjpZoDlf5NJoZ6DCV1K1KQhvJuQkqpHQa01C1LUKw2YqNfvRw2ryq9AoykgYAAABm0Rn2nXzN3QAAAABJRU5ErkJggg=="
];

// Milestone scores for unlocking better cars
const unlockMilestones = [0,10,25,50,75,150,300,500,1000];

// Car dimensions
const carWidth = 40;
const carHeight = 70;

// Game constants
const laneCount = 3;
const laneWidth = 320 / laneCount;
const playerY = 480 - carHeight - 15;

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const startScreen = document.getElementById('startScreen');
const gameOverScreen = document.getElementById('gameOverScreen');
const leaderboardList = document.getElementById('leaderboardList');
const gameOverLeaderboard = document.getElementById('gameOverLeaderboard');
const playerNameInput = document.getElementById('playerName');
const startBtn = document.getElementById('startBtn');
const restartBtn = document.getElementById('restartBtn');
const scoreDisplay = document.getElementById('scoreDisplay');
const finalScoreText = document.getElementById('finalScore');
const bgMusic = document.getElementById('bgMusic');
const crashSound = document.getElementById('crashSound');

let traffic = [];
let playerLane = 1;
let targetLane = 1;
let playerX = laneToX(playerLane);
let speed = 2;
let score = 0;
let gameOver = false;
let explosion = null;

class Particle {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.radius = Math.random() * 5 + 2;
    this.color = `hsl(${Math.random() * 60 + 30}, 100%, 50%)`;
    this.speedX = (Math.random() - 0.5) * 8;
    this.speedY = (Math.random() - 0.5) * 8;
    this.alpha = 1;
    this.decay = Math.random() * 0.03 + 0.015;
  }
  update() {
    this.x += this.speedX;
    this.y += this.speedY;
    this.alpha -= this.decay;
    this.radius *= 0.96;
  }
  draw() {
    ctx.save();
    ctx.globalAlpha = this.alpha;
    ctx.fillStyle = this.color;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
    ctx.fill();
    ctx.restore();
  }
}

class Explosion {
  constructor(x, y) {
    this.particles = [];
    for (let i = 0; i < 25; i++) {
      this.particles.push(new Particle(x, y));
    }
    this.done = false;
  }
  update() {
    this.particles.forEach(p => p.update());
    this.particles = this.particles.filter(p => p.alpha > 0.05 && p.radius > 0.5);
    if (this.particles.length === 0) this.done = true;
  }
  draw() {
    this.particles.forEach(p => p.draw());
  }
}

function laneToX(lane) {
  return lane * laneWidth + laneWidth / 2 - carWidth / 2;
}

// Load car images
const loadedCars = carSprites.map(src => {
  const img = new Image();
  img.src = src;
  return img;
});

function getUnlockedCarIndex(score) {
  let idx = 0;
  for(let i = 0; i < unlockMilestones.length; i++) {
    if(score >= unlockMilestones[i]) idx = i;
  }
  return Math.min(idx, loadedCars.length - 1);
}

function drawCarImage(img, x, y, width, height) {
  ctx.drawImage(img, x, y, width, height);
}

// Spawn traffic with spacing and gap logic so lanes aren't blocked
function spawnTraffic() {
  // We'll spawn 1 or 2 cars per spawn, with guaranteed gap lane(s)
  const lanes = [0,1,2];
  // Filter out lanes currently too crowded near top
  const safeLanes = lanes.filter(l => !traffic.some(c => c.lane === l && c.y < 150));
  
  // To create gaps, pick 1 or 2 lanes to fill, leave at least 1 gap lane empty
  let numCars = Math.random() < 0.3 ? 2 : 1; // 30% chance 2 cars, else 1
  numCars = Math.min(numCars, safeLanes.length);

  // Shuffle safe lanes
  for(let i = safeLanes.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [safeLanes[i], safeLanes[j]] = [safeLanes[j], safeLanes[i]];
  }
  
  // Add cars in those lanes
  for(let i = 0; i < numCars; i++) {
    const lane = safeLanes[i];
    traffic.push({
      lane,
      x: laneToX(lane),
      y: -carHeight - Math.random()*150,
      width: carWidth,
      height: carHeight,
      // Random car color index based on score
      colorIndex: Math.floor(Math.random() * getUnlockedCarIndex(score + 5)), // Slight randomness
      passed: false,
    });
  }
}

// Rect collision detection
function rectsCollide(r1, r2) {
  return !(r1.x > r2.x + r2.width ||
           r1.x + r1.width < r2.x ||
           r1.y > r2.y + r2.height ||
           r1.y + r1.height < r2.y);
}

// Update game logic
function update() {
  if(gameOver) {
    if(explosion) {
      explosion.update();
      if(explosion.done) {
        showGameOverScreen();
      }
    }
    return;
  }

  // Smooth lane switching
  let desiredX = laneToX(targetLane);
  if(Math.abs(playerX - desiredX) < 5) {
    playerX = desiredX;
    playerLane = targetLane;
  } else {
    playerX += (desiredX - playerX) * 0.3;
  }

  //
