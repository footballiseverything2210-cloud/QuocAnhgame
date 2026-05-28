# game
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kiên Săn Cục Cứt - Snake Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            font-family: 'Arial', sans-serif;
        }

        .container {
            text-align: center;
        }

        h1 {
            color: white;
            margin-bottom: 20px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        #gameCanvas {
            border: 3px solid white;
            background-color: #1a1a2e;
            display: block;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
        }

        .scoreboard {
            display: flex;
            justify-content: space-around;
            width: 600px;
            margin-top: 20px;
            background-color: rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 10px;
            backdrop-filter: blur(10px);
        }

        .score-section {
            color: white;
            font-size: 1.5em;
            font-weight: bold;
        }

        .score-section h2 {
            font-size: 1.2em;
            margin-bottom: 10px;
            text-transform: uppercase;
            letter-spacing: 2px;
        }

        .score {
            font-size: 2em;
            color: #ffd700;
        }

        .controls {
            color: white;
            margin-top: 20px;
            font-size: 0.9em;
            background-color: rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 10px;
            max-width: 600px;
            backdrop-filter: blur(10px);
        }

        .button-group {
            margin-top: 20px;
            display: flex;
            gap: 10px;
            justify-content: center;
        }

        button {
            background-color: #667eea;
            color: white;
            border: 2px solid white;
            padding: 12px 30px;
            font-size: 1em;
            border-radius: 5px;
            cursor: pointer;
            transition: all 0.3s ease;
            font-weight: bold;
        }

        button:hover {
            background-color: #764ba2;
            transform: scale(1.05);
        }

        button:active {
            transform: scale(0.95);
        }

        @media (max-width: 768px) {
            h1 {
                font-size: 1.8em;
            }

            #gameCanvas {
                max-width: 100vw;
            }

            .scoreboard {
                width: 90vw;
                max-width: 600px;
            }

            .controls {
                width: 90vw;
                max-width: 600px;
                font-size: 0.8em;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🐍 KIÊN SĂN CỤC CỨT 💩</h1>
        <canvas id="gameCanvas" width="600" height="600"></canvas>

        <div class="scoreboard">
            <div class="score-section">
                <h2>Score</h2>
                <div class="score" id="score">0</div>
            </div>
            <div class="score-section">
                <h2>Độ Dài Rắn</h2>
                <div class="score" id="length">1</div>
            </div>
        </div>

        <div class="controls">
            <h3>Điều Khiển</h3>
            <p>⬆️⬇️⬅️➡️ Mũi tên hoặc W/A/S/D để điều khiển rắn Kiên</p>
            <p>Ăn 💩 để mọc thêm thân và tăng điểm</p>
            <p>Đừng va vào tường hoặc chính mình!</p>
        </div>

        <div class="button-group">
            <button onclick="startGame()">Start Game</button>
            <button onclick="resetGame()">Reset</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        const gridSize = 20;
        const tileCount = canvas.width / gridSize;

        let snake = [
            {x: Math.floor(tileCount / 2), y: Math.floor(tileCount / 2)}
        ];

        let food = {
            x: Math.floor(Math.random() * tileCount),
            y: Math.floor(Math.random() * tileCount)
        };

        let direction = {x: 1, y: 0};
        let nextDirection = {x: 1, y: 0};
        let score = 0;
        let gameRunning = false;
        let gameOver = false;

        const keys = {};

        window.addEventListener('keydown', (e) => {
            keys[e.key] = true;

            // Arrow keys
            if (e.key === 'ArrowUp' || e.key === 'w' || e.key === 'W') {
                if (direction.y === 0) nextDirection = {x: 0, y: -1};
            } else if (e.key === 'ArrowDown' || e.key === 's' || e.key === 'S') {
                if (direction.y === 0) nextDirection = {x: 0, y: 1};
            } else if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') {
                if (direction.x === 0) nextDirection = {x: -1, y: 0};
            } else if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') {
                if (direction.x === 0) nextDirection = {x: 1, y: 0};
            }
        });

        function drawGame() {
            // Clear canvas
            ctx.fillStyle = '#1a1a2e';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw grid
            ctx.strokeStyle = 'rgba(255, 255, 255, 0.1)';
            ctx.lineWidth = 1;
            for (let i = 0; i <= tileCount; i++) {
                ctx.beginPath();
                ctx.moveTo(i * gridSize, 0);
                ctx.lineTo(i * gridSize, canvas.height);
                ctx.stroke();

                ctx.beginPath();
                ctx.moveTo(0, i * gridSize);
                ctx.lineTo(canvas.width, i * gridSize);
                ctx.stroke();
            }

            // Draw snake
            snake.forEach((segment, index) => {
                if (index === 0) {
                    // Head - màu xanh sáng
                    ctx.fillStyle = '#00ff88';
                } else {
                    // Body - màu xanh đậm hơn
                    ctx.fillStyle = '#00cc66';
                }
                ctx.fillRect(
                    segment.x * gridSize + 1,
                    segment.y * gridSize + 1,
                    gridSize - 2,
                    gridSize - 2
                );

                // Draw eyes on head
                if (index === 0) {
                    ctx.fillStyle = 'black';
                    const eyeSize = 3;
                    if (direction.x === 1) { // Moving right
                        ctx.fillRect(segment.x * gridSize + 12, segment.y * gridSize + 6, eyeSize, eyeSize);
                        ctx.fillRect(segment.x * gridSize + 12, segment.y * gridSize + 12, eyeSize, eyeSize);
                    } else if (direction.x === -1) { // Moving left
                        ctx.fillRect(segment.x * gridSize + 5, segment.y * gridSize + 6, eyeSize, eyeSize);
                        ctx.fillRect(segment.x * gridSize + 5, segment.y * gridSize + 12, eyeSize, eyeSize);
                    } else if (direction.y === -1) { // Moving up
                        ctx.fillRect(segment.x * gridSize + 6, segment.y * gridSize + 5, eyeSize, eyeSize);
                        ctx.fillRect(segment.x * gridSize + 12, segment.y * gridSize + 5, eyeSize, eyeSize);
                    } else if (direction.y === 1) { // Moving down
                        ctx.fillRect(segment.x * gridSize + 6, segment.y * gridSize + 12, eyeSize, eyeSize);
                        ctx.fillRect(segment.x * gridSize + 12, segment.y * gridSize + 12, eyeSize, eyeSize);
                    }
                }
            });

            // Draw food (poop emoji style)
            ctx.fillStyle = '#8B4513';
            ctx.font = '16px Arial';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('💩', food.x * gridSize + gridSize / 2, food.y * gridSize + gridSize / 2);

            // Draw scores
            ctx.fillStyle = 'white';
            ctx.font = '20px Arial';
            ctx.textAlign = 'left';
            ctx.fillText('Score: ' + score, 20, 30);
            ctx.fillText('Length: ' + snake.length, 20, 60);

            // Draw game status
            if (!gameRunning && !gameOver) {
                ctx.font = 'bold 30px Arial';
                ctx.fillStyle = 'rgba(255, 255, 255, 0.7)';
                ctx.textAlign = 'center';
                ctx.fillText('Press Start to Begin', canvas.width / 2, canvas.height / 2);
            }

            if (gameOver) {
                ctx.font = 'bold 40px Arial';
                ctx.fillStyle = '#ff0055';
                ctx.textAlign = 'center';
                ctx.fillText('GAME OVER!', canvas.width / 2, canvas.height / 2);
                ctx.font = '20px Arial';
                ctx.fillStyle = 'white';
                ctx.fillText('Score: ' + score, canvas.width / 2, canvas.height / 2 + 40);
                ctx.fillText('Press Reset to Play Again', canvas.width / 2, canvas.height / 2 + 80);
            }
        }

        function updateGame() {
            if (!gameRunning || gameOver) return;

            direction = nextDirection;

            // Move snake
            const head = {x: snake[0].x + direction.x, y: snake[0].y + direction.y};

            // Check wall collision
            if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
                gameOver = true;
                return;
            }

            // Check self collision
            for (let segment of snake) {
                if (head.x === segment.x && head.y === segment.y) {
                    gameOver = true;
                    return;
                }
            }

            snake.unshift(head);

            // Check food collision
            if (head.x === food.x && head.y === food.y) {
                score += 10;
                generateFood();
                document.getElementById('score').textContent = score;
                document.getElementById('length').textContent = snake.length;
            } else {
                snake.pop();
            }
        }

        function generateFood() {
            let newFood;
            let isOnSnake;
            do {
                isOnSnake = false;
                newFood = {
                    x: Math.floor(Math.random() * tileCount),
                    y: Math.floor(Math.random() * tileCount)
                };
                for (let segment of snake) {
                    if (newFood.x === segment.x && newFood.y === segment.y) {
                        isOnSnake = true;
                        break;
                    }
                }
            } while (isOnSnake);
            food = newFood;
        }

        function gameLoop() {
            updateGame();
            drawGame();
            requestAnimationFrame(gameLoop);
        }

        function startGame() {
            if (!gameRunning && !gameOver) {
                gameRunning = true;
            }
        }

        function resetGame() {
            snake = [{x: Math.floor(tileCount / 2), y: Math.floor(tileCount / 2)}];
            direction = {x: 1, y: 0};
            nextDirection = {x: 1, y: 0};
            score = 0;
            gameRunning = false;
            gameOver = false;
            generateFood();
            document.getElementById('score').textContent = '0';
            document.getElementById('length').textContent = '1';
        }

        // Set game speed (mili giây giữa mỗi bước)
        let gameSpeed = 100;
        let lastUpdate = 0;

        function optimizedGameLoop(timestamp) {
            if (timestamp - lastUpdate > gameSpeed) {
                updateGame();
                lastUpdate = timestamp;
            }
            drawGame();
            requestAnimationFrame(optimizedGameLoop);
        }

        // Start
        drawGame();
        requestAnimationFrame(optimizedGameLoop);
    </script>
</body>
</html>
