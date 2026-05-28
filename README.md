# game
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
            background: url('https://media.discordapp.net/attachments/6e46f401-fa81-48d9-b946-f742e1be7b31.png') center/cover fixed;
            background-attachment: fixed;
            font-family: 'Arial', sans-serif;
        }

        body::before {
            content: '';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.35);
            z-index: -1;
            pointer-events: none;
        }

        .container {
            text-align: center;
        }

        h1 {
            color: white;
            margin-bottom: 20px;
            font-size: 2.5em;
            text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.5);
            letter-spacing: 2px;
        }

        #gameCanvas {
            border: 5px solid white;
            background-color: #1a1a2e;
            display: block;
            box-shadow: 0 0 30px rgba(0, 0, 0, 0.7);
            border-radius: 10px;
        }

        .scoreboard {
            display: flex;
            justify-content: space-around;
            width: 600px;
            margin-top: 20px;
            background-color: rgba(255, 255, 255, 0.15);
            padding: 20px;
            border-radius: 15px;
            backdrop-filter: blur(10px);
            border: 2px solid rgba(255, 255, 255, 0.3);
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
            font-size: 2.5em;
            color: #ffd700;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .controls {
            color: white;
            margin-top: 20px;
            font-size: 0.95em;
            background-color: rgba(255, 255, 255, 0.15);
            padding: 20px;
            border-radius: 15px;
            max-width: 600px;
            backdrop-filter: blur(10px);
            border: 2px solid rgba(255, 255, 255, 0.3);
        }

        .controls h3 {
            margin-bottom: 15px;
            text-transform: uppercase;
            letter-spacing: 1px;
            font-size: 1.1em;
        }

        .controls p {
            margin: 8px 0;
        }

        .button-group {
            margin-top: 20px;
            display: flex;
            gap: 15px;
            justify-content: center;
        }

        button {
            background-color: #667eea;
            color: white;
            border: 3px solid white;
            padding: 14px 35px;
            font-size: 1.1em;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.3s ease;
            font-weight: bold;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
        }

        button:hover {
            background-color: #764ba2;
            transform: scale(1.08);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.4);
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
                font-size: 0.85em;
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
                <h2>📊 Score</h2>
                <div class="score" id="score">0</div>
            </div>
            <div class="score-section">
                <h2>📏 Độ Dài</h2>
                <div class="score" id="length">1</div>
            </div>
            <div class="score-section">
                <h2>⚡ Tốc Độ</h2>
                <div class="score" id="speed">Chậm</div>
            </div>
        </div>

        <div class="controls">
            <h3>🎮 Điều Khiển</h3>
            <p>⬆️⬇️⬅️➡️ Mũi tên hoặc W/A/S/D để điều khiển</p>
            <p>💩 Ăn cục cứt để mọc thêm thân và tăng điểm</p>
            <p>⚡ Rắn càng dài, chạy càng nhanh!</p>
            <p>🌀 Chạy xuyên tường sẽ sang phía bên kia</p>
            <p>❌ Chỉ chết khi cắn vào chính mình!</p>
        </div>

        <div class="button-group">
            <button onclick="startGame()">▶️ Start Game</button>
            <button onclick="resetGame()">🔄 Reset</button>
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
        let baseGameSpeed = 120;

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

        // Tính toán tốc độ theo độ dài rắn
        function getGameSpeed() {
            // Rắn càng dài, tốc độ càng nhanh
            // length = 1: 120ms, length = 10: 80ms, length = 20+: 40ms
            const speedReduction = Math.min((snake.length - 1) * 4, 80);
            return baseGameSpeed - speedReduction;
        }

        // Cập nhật hiển thị tốc độ
        function updateSpeedDisplay() {
            const speed = getGameSpeed();
            let speedText = 'Chậm';
            if (speed < 80) speedText = '🔥 Cực Nhanh!';
            else if (speed < 100) speedText = '⚡ Rất Nhanh';
            else if (speed < 110) speedText = '💨 Nhanh';
            else speedText = '🐢 Chậm';
            document.getElementById('speed').textContent = speedText;
        }

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

                // Chèm chữ "KIÊN" vào rắn (mỗi 5 đoạn)
                if (index > 0 && index % 5 === 0) {
                    ctx.fillStyle = '#ffd700';
                    ctx.font = 'bold 12px Arial';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText('K', segment.x * gridSize + gridSize / 2, segment.y * gridSize + gridSize / 2);
                }
            });

            // Draw food (poop emoji style)
            ctx.fillStyle = '#8B4513';
            ctx.font = '18px Arial';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText('💩', food.x * gridSize + gridSize / 2, food.y * gridSize + gridSize / 2);

            // Draw scores
            ctx.fillStyle = 'white';
            ctx.font = 'bold 18px Arial';
            ctx.textAlign = 'left';
            ctx.fillText('Score: ' + score, 20, 30);
            ctx.fillText('Length: ' + snake.length, 20, 60);

            // Draw game status
            if (!gameRunning && !gameOver) {
                ctx.font = 'bold 30px Arial';
                ctx.fillStyle = 'rgba(255, 255, 255, 0.7)';
                ctx.textAlign = 'center';
                ctx.fillText('Press Start to Begin', canvas.width / 2, canvas.height / 2 - 20);
                ctx.font = '16px Arial';
                ctx.fillText('🐍 KIÊN 🐍', canvas.width / 2, canvas.height / 2 + 30);
            }

            if (gameOver) {
                // Draw semi-transparent overlay
                ctx.fillStyle = 'rgba(0, 0, 0, 0.6)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                ctx.font = 'bold 50px Arial';
                ctx.fillStyle = '#ff0055';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('KIÊN NGU! 😂', canvas.width / 2, canvas.height / 2 - 60);

                ctx.font = 'bold 35px Arial';
                ctx.fillStyle = '#ffd700';
                ctx.fillText('GAME OVER!', canvas.width / 2, canvas.height / 2);

                ctx.font = '20px Arial';
                ctx.fillStyle = 'white';
                ctx.fillText('Score: ' + score + ' | Length: ' + snake.length, canvas.width / 2, canvas.height / 2 + 50);
                ctx.fillText('Press Reset to Play Again', canvas.width / 2, canvas.height / 2 + 90);
            }
        }

        function updateGame() {
            if (!gameRunning || gameOver) return;

            direction = nextDirection;

            // Move snake
            let newX = snake[0].x + direction.x;
            let newY = snake[0].y + direction.y;

            // WRAPAROUND MODE - Chạy xuyên tường
            // Nếu vượt ra ngoài, sẽ xuất hiện ở phía bên kia
            if (newX < 0) newX = tileCount - 1;
            if (newX >= tileCount) newX = 0;
            if (newY < 0) newY = tileCount - 1;
            if (newY >= tileCount) newY = 0;

            const head = {x: newX, y: newY};

            // Check self collision - CHỈ CHẾT KHI CẮN VÀO CHÍNH MÌNH
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
                updateSpeedDisplay();
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
            document.getElementById('speed').textContent = 'Chậm';
        }

        let lastUpdate = 0;

        function optimizedGameLoop(timestamp) {
            const currentSpeed = getGameSpeed();
            if (timestamp - lastUpdate > currentSpeed) {
                updateGame();
                lastUpdate = timestamp;
            }
            drawGame();
            requestAnimationFrame(optimizedGameLoop);
        }

        // Start
        drawGame();
        updateSpeedDisplay();
        requestAnimationFrame(optimizedGameLoop);
    </script>
</body>
</html>
