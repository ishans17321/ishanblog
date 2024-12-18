<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake Game</title>
    <style>
        body {
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #000;
            color: #fff;
            font-family: Arial, sans-serif;
        }

        canvas {
            border: 10px solid #fff;
            display: block;
        }

        #menu {
            text-align: center;
        }

        .hidden {
            display: none;
        }

        button {
            margin: 10px;
            padding: 10px 20px;
            font-size: 1rem;
            cursor: pointer;
        }

        #score {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 1.5rem;
        }
    </style>
</head>
<body>

<div id="menu">
    <h1>Snake Game</h1>
    <button id="start">Start Game</button>
</div>

<div id="score" class="hidden">Score: <span id="scoreValue">0</span></div>
<canvas id="gameCanvas" class="hidden" width="400" height="400"></canvas>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const menu = document.getElementById('menu');
    const startButton = document.getElementById('start');
    const scoreDisplay = document.getElementById('score');
    const scoreValue = document.getElementById('scoreValue');

    const blockSize = 20;
    const canvasSize = 400;
    const rows = canvasSize / blockSize;
    const cols = canvasSize / blockSize;

    let snake = [{ x: 10, y: 10 }];
    let direction = { x: 0, y: 0 };
    let food = { x: Math.floor(Math.random() * cols), y: Math.floor(Math.random() * rows) };
    let score = 0;
    let gameInterval;

    function drawBlock(color, x, y) {
        ctx.fillStyle = color;
        ctx.fillRect(x * blockSize, y * blockSize, blockSize, blockSize);
    }

    function drawSnake() {
        snake.forEach(segment => drawBlock('#0f0', segment.x, segment.y));
    }

    function drawFood() {
        ctx.font = "20px Arial"; // Set font size for the emoji
        ctx.textAlign = "center"; // Center the emoji horizontally
        ctx.textBaseline = "middle"; // Center the emoji vertically
        ctx.fillText("🍎", (food.x + 0.5) * blockSize, (food.y + 0.5) * blockSize);
    }


    function updateSnake() {
        const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

        if (head.x < 0 || head.x >= cols || head.y < 0 || head.y >= rows || snake.some(segment => segment.x === head.x && segment.y === head.y)) {
            clearInterval(gameInterval);
            alert('Game Over! Your score: ' + score);
            resetGame();
            return;
        }

        snake.unshift(head);

        if (head.x === food.x && head.y === food.y) {
            score++;
            scoreValue.textContent = score;
            food = { x: Math.floor(Math.random() * cols), y: Math.floor(Math.random() * rows) };
        } else {
            snake.pop();
        }
    }

    function gameLoop() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        drawSnake();
        drawFood();
        updateSnake();
    }

    function resetGame() {
        snake = [{ x: 10, y: 10 }];
        direction = { x: 0, y: 0 };
        food = { x: Math.floor(Math.random() * cols), y: Math.floor(Math.random() * rows) };
        score = 0;
        scoreValue.textContent = score;
        menu.classList.remove('hidden');
        canvas.classList.add('hidden');
        scoreDisplay.classList.add('hidden');
    }

    startButton.addEventListener('click', () => {
        menu.classList.add('hidden');
        canvas.classList.remove('hidden');
        scoreDisplay.classList.remove('hidden');

        document.addEventListener('keydown', (e) => {
            switch (e.key) {
                case 'ArrowUp':
                    if (direction.y === 0) direction = { x: 0, y: -1 };
                    break;
                case 'ArrowDown':
                    if (direction.y === 0) direction = { x: 0, y: 1 };
                    break;
                case 'ArrowLeft':
                    if (direction.x === 0) direction = { x: -1, y: 0 };
                    break;
                case 'ArrowRight':
                    if (direction.x === 0) direction = { x: 1, y: 0 };
                    break;
            }
        });

        gameInterval = setInterval(gameLoop, 100);
    });
</script>

</body>
</html>
