# Sanke
Snake The iconic Snake game was first introduced in 1997 on Nokia devices. Developed by Taneli Armanto, it was one of three games pre-installed on the Nokia 6110. The game's simple yet addictive gameplay led to widespread popularity and it became a staple of early mobile phones. Conveniently i need to make a game for a project so it a speed run when i made it.

<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Rắn Săn Mồi - Họ Tên: [Thay Tên Bạn] - MSSV: [Thay MSSV]</title>
    <style>
        canvas {
            border: 1px solid black;
        }
        #gameContainer {
            text-align: center;
            margin-top: 20px;
        }
        .info-box {
            display: inline-block;
            width: 100px;
            height: 50px;
            border: 2px solid black;
            margin: 10px;
            line-height: 50px;
            font-size: 18px;
        }
        #shop button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            font-size: 16px;
            cursor: pointer;
            margin: 5px;
        }
        #shop button:hover {
            background-color: #45a049;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div class="info-box">Điểm: <span id="score">0</span></div>
        <div class="info-box">Thời gian: <span id="gameTime">0</span></div>
        <br>
        <canvas id="gameCanvas" width="500" height="500"></canvas>
        <p id="timer" style="display: none;">Thời gian còn lại khi chạm tường: <span id="timeLeft">10</span></p>
        <div id="shop" style="margin-top: 10px;">
            <button onclick="buySpeedBoost()">Tăng tốc (10 điểm)</button>
            <button onclick="buyExtraTime()">Thêm thời gian (15 điểm)</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const gridSize = 20;
        const tileCount = canvas.width / gridSize;
        let snake = [{ x: 10, y: 10 }];
        let foods = [];
        const maxFoods = 3;
        let dx = 0;
        let dy = 0;
        let score = 0;
        let gameLoop = null;
        let gameStarted = false;
        let wallHit = false;
        let timeLeft = 10;
        let timerInterval = null;
        let gameTime = 0;
        let gameTimeInterval = null;
        let speedBoostActive = false;

        function spawnFood() {
            return {
                x: Math.floor(Math.random() * tileCount),
                y: Math.floor(Math.random() * tileCount)
            };
        }

        function initializeFoods() {
            while (foods.length < maxFoods) {
                foods.push(spawnFood());
            }
        }

        function draw() {
            ctx.fillStyle = 'black';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = 'green';
            snake.forEach(segment => {
                ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize - 2, gridSize - 2);
            });

            ctx.fillStyle = 'red';
            foods.forEach(food => {
                ctx.fillRect(food.x * gridSize, food.y * gridSize, gridSize - 2, gridSize - 2);
            });

            document.getElementById('score').innerText = score;
            document.getElementById('gameTime').innerText = gameTime;
            if (wallHit) {
                document.getElementById('timer').style.display = 'block';
                document.getElementById('timeLeft').innerText = timeLeft;
            }
            if (speedBoostActive) {
                ctx.fillStyle = 'yellow';
                ctx.font = '20px Arial';
                ctx.fillText('Tăng tốc!', 10, 30);
            }
        }

        function move() {
            if (!gameStarted) return;

            const head = { x: snake[0].x + dx, y: snake[0].y + dy };

            if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
                if (!wallHit) {
                    wallHit = true;
                    timeLeft = 1;
                    startTimer();
                }
                return;
            }

            wallHit = false;
            clearTimer();

            for (let i = 0; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    endGame();
                    return;
                }
            }

            snake.unshift(head);
            let ateFood = false;
            for (let i = 0; i < foods.length; i++) {
                if (head.x === foods[i].x && head.y === foods[i].y) {
                    score += 1;
                    foods.splice(i, 1);
                    foods.push(spawnFood());
                    ateFood = true;
                    break;
                }
            }

            if (!ateFood) {
                snake.pop();
            }

            draw();
        }

        function startTimer() {
            clearTimer();
            timerInterval = setInterval(() => {
                timeLeft--;
                document.getElementById('timeLeft').innerText = timeLeft;
                if (timeLeft <= 0) {
                    endGame();
                }
            }, 1000);
        }

        function clearTimer() {
            if (timerInterval) {
                clearInterval(timerInterval);
                timerInterval = null;
            }
            document.getElementById('timer').style.display = 'none';
        }

        function startGameTime() {
            gameTimeInterval = setInterval(() => {
                gameTime++;
                document.getElementById('gameTime').innerText = gameTime;
            }, 1000);
        }

        function endGame() {
            clearInterval(gameLoop);
            clearInterval(gameTimeInterval);
            clearTimer();
            gameStarted = false;
            alert('Game Over! Điểm của bạn: ' + score + ' - Thời gian chơi: ' + gameTime + ' giây');
            snake = [{ x: 10, y: 10 }];
            foods = [];
            initializeFoods();
            dx = 0;
            dy = 0;
            score = 0;
            gameTime = 0;
            wallHit = false;
            timeLeft = 10;
            speedBoostActive = false;
            draw();
        }

        function startGame() {
            if (gameStarted) return;
            gameStarted = true;
            if (gameLoop) clearInterval(gameLoop);
            gameLoop = setInterval(move, 500);
            startGameTime();
            draw();
        }

        function buySpeedBoost() {
            if (gameTime >= 10 && score >= 1 && !speedBoostActive) {
                gameTime -= 10;
                score -= 1;
                speedBoostActive = true;
                clearInterval(gameLoop);
                gameLoop = setInterval(move, 50);
                setTimeout(() => {
                    speedBoostActive = false;
                    clearInterval(gameLoop);
                    gameLoop = setInterval(move, 500);
                }, 5000);
                draw();
            } else if (speedBoostActive) {
                alert("Tăng tốc đang hoạt động!");
            } else {
                alert("Không đủ điểm! Cần 10 điểm.");
            }
        }

        function buyExtraTime() {
            if (gameTime >= 15 && score >= 2) {
                gameTime -= 15;
                score -= 2;
                timeLeft += 5;
                draw();
            } else {
                alert("Không đủ điểm! Cần 15 điểm.");
            }
        }

        document.addEventListener('keydown', (e) => {
            const key = e.key.toLowerCase();
            if (!gameStarted && (key === 'w' || key === 'a' || key === 's' || key === 'd')) {
                startGame();
            }
            if (!gameStarted) return;

            switch (key) {
                case 'w': if (dy !== 1) { dx = 0; dy = -1; } break;
                case 's': if (dy !== -1) { dx = 0; dy = 1; } break;
                case 'a': if (dx !== 1) { dx = -1; dy = 0; } break;
                case 'd': if (dx !== -1) { dx = 1; dy = 0; } break;
            }
        });

        initializeFoods();
        draw();
    </script>
</body>
</html>
