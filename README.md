<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Deadline Hehe</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #111;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
        }
        #gameCanvas {
            border: 1px solid #000;
            background: #333; /* Darker background for better trail visibility */
        }
        .hud {
            position: absolute;
            top: 10px;
            left: 10px;
            color: white;
            font-family: Arial, sans-serif;
            font-size: 14px;
            background: rgba(0, 0, 0, 0.5);
            padding: 10px;
            border-radius: 8px;
        }
        .restart-btn {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 15px 30px;
            background: #007bff;
            color: #fff;
            border: none;
            border-radius: 8px;
            font-size: 20px;
            cursor: pointer;
            display: none;
        }
        .game-over {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: red;
            font-size: 50px;
            font-family: Arial, sans-serif;
            display: none;
        }
        footer {
            position: absolute;
            bottom: 10px;
            left: 10px;
            color: white;
            font-size: 12px;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    <div class="hud">
        <div>Speed: <span id="speed">0</span> km/h</div>
        <div>Score: <span id="score">0</span></div>
        <div>Power-Up: <span id="powerUp">None</span></div>
        <div>Round: <span id="round">1</span></div>
        <div>Cars Remaining: <span id="remainingCars">3</span></div>
    </div>
    <div class="game-over" id="gameOverText">You Died!</div>
    <button class="restart-btn" id="restartBtn" onclick="restartGame()">Restart Game</button>
    <footer>
        Copyright © <a href="https://www.youtube.com/@Hehe-Playz29" target="_blank">https://www.youtube.com/@Hehe-Playz29</a>
    </footer>

    <script>
        // Canvas setup
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const WIDTH = window.innerWidth;
        const HEIGHT = window.innerHeight;
        canvas.width = WIDTH;
        canvas.height = HEIGHT;

        // Game variables
        const MAX_SPEED = 5;
        const TURN_RATE = 0.1;
        const LIGHT_TRAIL_LENGTH = 100;
        let speed = 0;
        let score = 0;
        let playerAngle = 0;
        let playerX = WIDTH / 2;
        let playerY = HEIGHT / 2;
        let round = 1;
        let powerUpActive =null;
        let powerUpTimer = 0;
        let carsRemaining = 3; // Starting number of AI cars
        let gameOver = false;

        const carsToEliminate = [3, 5, 7]; // Number of cars per round
        const aiCars = [];

        // Light trails
        let playerTrail = [];
        let powerUps = [];

        // Key states
        const keys = {};

        // Listen for key presses
        window.addEventListener('keydown', e => keys[e.code] = true);
        window.addEventListener('keyup', e => keys[e.code] = false);

        // Initialize AI cars for the first round
        spawnAICars(carsToEliminate[0]);

        // Game loop
        function gameLoop() {
            update();
            draw();

            if (!gameOver) requestAnimationFrame(gameLoop);
        }
        gameLoop();

        // Update game state
        function update() {
            if (gameOver) return;

            // Player controls
            if (keys['ArrowLeft']) playerAngle -= TURN_RATE;
            if (keys['ArrowRight']) playerAngle += TURN_RATE;
            if (keys['ArrowUp']) speed = Math.min(speed + 0.1, MAX_SPEED);
            if (keys['ArrowDown']) speed = Math.max(speed - 0.1, -MAX_SPEED / 2);

            // Update player position
            playerX += Math.cos(playerAngle) * speed;
            playerY += Math.sin(playerAngle) * speed;

            // Keep player within bounds
            playerX = Math.max(0, Math.min(WIDTH, playerX));
            playerY = Math.max(0, Math.min(HEIGHT, playerY));

            // Add to player trail
            playerTrail.push({ x: playerX, y: playerY });
            if (playerTrail.length > LIGHT_TRAIL_LENGTH) playerTrail.shift();

            // Update AI cars
            aiCars.forEach(ai => {
                ai.x += Math.cos(ai.angle) * ai.speed;
                ai.y += Math.sin(ai.angle) * ai.speed;

                // Turn around at borders
                if (ai.x < 0 || ai.x > WIDTH || ai.y < 0 || ai.y > HEIGHT) {
                    ai.angle += Math.PI;
                }

                // Add to AI trail
                ai.trail.push({ x: ai.x, y: ai.y });
                if (ai.trail.length > LIGHT_TRAIL_LENGTH) ai.trail.shift();

                // AI car collision with player trail
                if (playerTrail.some(trailPoint => distance(trailPoint, ai) < 10)) {
                    removeAICar(ai);
                }

                // AI car collision with player
                if (distance(ai, { x: playerX, y: playerY }) < 15) {
                    gameOver = true;
                    showGameOver();
                }
            });

            // Check if player collides with any AI trail
            aiCars.forEach(ai => {
                if (ai.trail.some(trailPoint => distance(trailPoint, { x: playerX, y: playerY }) < 10)) {
                    gameOver = true;
                    showGameOver();
                }
            });

            // Check for round completion
            if (carsRemaining <= 0) nextRound();

            // Update HUD
            document.getElementById('speed').textContent = Math.abs(speed * 10).toFixed(1);
            document.getElementById('score').textContent = score;
            document.getElementById('round').textContent = round;
            document.getElementById('remainingCars').textContent = carsRemaining;
        }

        // Draw game state
        function draw() {
            ctx.clearRect(0, 0, WIDTH, HEIGHT);

            // Draw player trail
            drawTrail(playerTrail, 'cyan');

            // Draw player car
            ctx.save();
            ctx.translate(playerX, playerY);
            ctx.rotate(playerAngle);
            ctx.fillStyle = 'cyan';
            ctx.fillRect(-10, -10, 20, 20);
            ctx.restore();

            // Draw AI cars and trails
            aiCars.forEach(ai => {
                drawTrail(ai.trail, 'red');

                ctx.save();
                ctx.translate(ai.x, ai.y);
                ctx.rotate(ai.angle);
                ctx.fillStyle = 'red';
                ctx.fillRect(-10, -10, 20, 20);
                ctx.restore();
            });
        }

        // Draw a trail
        function drawTrail(trail, color) {
            ctx.strokeStyle = color;
            ctx.lineWidth = 5;
            ctx.beginPath();
            trail.forEach((point, index) => {
                if (index === 0) ctx.moveTo(point.x, point.y);
                else ctx.lineTo(point.x, point.y);
            });
            ctx.stroke();
        }

        // Handle round completion
        function nextRound() {
            round++;
            if (round > 3) {
                showGameCompleted();
            } else {
                spawnAICars(carsToEliminate[round - 1]);
                carsRemaining = carsToEliminate[round - 1];
            }
        }

        // Spawn AI cars for the round
        function spawnAICars(count) {
            aiCars.length = 0;
            for (let i = 0; i < count; i++) {
                aiCars.push({
                    x: Math.random() * WIDTH,
                    y: Math.random() * HEIGHT,
                    angle: Math.random() * Math.PI * 2,
                    speed: Math.random() * 3 + 1,
                    trail: []
                });
            }
        }

        // Remove AI car
        function removeAICar(ai) {
            const index = aiCars.indexOf(ai);
            if (index !== -1) {
                aiCars.splice(index, 1);
                score += 100;
                carsRemaining--;
            }
        }

        // Show game over screen
        function showGameOver() {
            document.getElementById('gameOverText').style.display = 'block';
            document.getElementById('restartBtn').style.display = 'block';
        }

        // Show game completed screen
        function showGameCompleted() {
            document.getElementById('gameOverText').textContent = 'Game Completed!';
            document.getElementById('gameOverText').style.display = 'block';
            document.getElementById('restartBtn').style.display = 'block';
        }

        // Restart the game
        function restartGame() {
            document.getElementById('gameOverText').style.display = 'none';
            document.getElementById('restartBtn').style.display = 'none';
            score = 0;
            round = 1;
            carsRemaining = carsToEliminate[0];
            playerX = WIDTH / 2;
            playerY = HEIGHT / 2;
            playerTrail = [];
            gameOver = false;
            spawnAICars(carsToEliminate[0]);
        }

        // Utility function to calculate distance between two points
        function distance(a, b) {
            return Math.sqrt((a.x - b.x) ** 2 + (a.y - b.y) ** 2);
        }
    </script>
</body>
</html>
