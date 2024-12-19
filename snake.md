---
layout: page
title: Snake
search_exclude: true
permalink: /snake/
---

<style>
    body {}
    .wrap {
        margin-left: auto;
        margin-right: auto;
    }
    canvas {
        display: none;
        border-style: solid;
        border-width: 50px;
        border-color: rgb(30, 121, 44);
    }
    canvas:focus {
        outline: none;
    }
    #gameover p, #setting p, #menu p {
        font-size: 20px;
    }
    #gameover a, #setting a, #menu a {
        font-size: 30px;
        display: block;
    }
    #gameover a:hover, #setting a:hover, #menu a:hover {
        cursor: pointer;
    }
    #gameover a:hover::before, #setting a:hover::before, #menu a:hover::before {
        content: ">";
        margin-right: 10px;
    }
    #menu {
        display: block;
    }
    #gameover {
        display: none;
    }
    #setting {
        display: none;
    }
    #setting input {
        display: none;
    }
    #setting label {
        cursor: pointer;
    }
    #setting input:checked + label {
        background-color: #FFF;
        color: #000000
    }
</style>

<h2>Snake</h2>
<div class="container">
    <header class="pb-3 mb-4 border-bottom border-primary text-dark">
        <p class="fs-4">Score: <span id="score_value">0</span></p>
    </header>
    <div class="container bg-secondary" style="text-align:center;">
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: rgb(220,37,37); color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color:rgb(220, 37, 37); color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <canvas id="snake" class="wrap" width="800" height="800" tabindex="1"></canvas>
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color:rgb(220, 37, 37); color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="200" checked />
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="150" />
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="100" />
                <label for="speed3">Fast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked />
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0" />
                <label for="walloff">Off</label>
            </p>
        </div>
    </div>
</div>

<script>
(function () {
    const canvas = document.getElementById("snake");
    const ctx = canvas.getContext("2d");
    const SCREEN_SNAKE = 0;
    const screen_snake = document.getElementById("snake");
    const ele_score = document.getElementById("score_value");
    const speed_setting = document.getElementsByName("speed");
    const wall_setting = document.getElementsByName("wall");
    const SCREEN_MENU = -1, SCREEN_GAME_OVER = 1, SCREEN_SETTING = 2;
    const screen_menu = document.getElementById("menu");
    const screen_game_over = document.getElementById("gameover");
    const screen_setting = document.getElementById("setting");
    const button_new_game = document.getElementById("new_game");
    const button_new_game1 = document.getElementById("new_game1");
    const button_new_game2 = document.getElementById("new_game2");
    const button_setting_menu = document.getElementById("setting_menu");
    const button_setting_menu1 = document.getElementById("setting_menu1");

    const BLOCK = 50;
    let SCREEN = SCREEN_MENU;
    let snake;
    let snake_dir;
    let snake_next_dir;
    let snake_speed;
    let food = { x: 0, y: 0 };
    let score;
    let wall;

    // Load tennis racket image
    const tennisRacketImage = new Image();
    tennisRacketImage.src = "data:image/jpeg;base64,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Wx13mpJcd7idwCx3QfoKILk9paM43GOIULYyRS88jty037G7BxO+JCCIioIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgweX+ilkteM8WnukbovHDSG2nOq0K29CLfYnCSwymZjXXwzsuBAIqYzoPNCRXbiusotsfaL440iduk7w8Wx1tvPNyuw+0meGUjKFnLdFooG5twILiXESGhrUYB2FOazdj6b2GSBwdIYi57jccCTR0xIINLpqDWgJpU8FuVqsccrbssbXt4OaHD8itWt/s2yfISWsfCTtzTy0fkagfhRaceC/ipMe6fpPq8cOSOU6+/7M5JlCF8kAZNG6peQGuDq0jINKcnKRA4Z2U1GDYwcRUdo4jd2t655avY/HUugtkrD8zWP/WlVHHsttja3MoDHbUSCuFMbjxVck89mzoETfdbOCW192xqKGjozgdh2YU2qTI9vWG1c0ERPwJFaF7MacNGlVzT/AIZZQIDDbmXG0ujXkC72aDOUFKYK9/wpndjJlDHZhGXf/Y9ym6tvtHSOxxNtBfaI633aN4XqiNougHacNuzHatXyz7S4GRxNszC5wLO9BYDRuxorec6vpB2HapVg9k9mbTPWieQ/K4Qg/cRAH9VtGRuilisprZ7MxjvXS8/++6p/VNxzn9kZUyrIJZm5mIgC88Ojbdreo2Gt+XHEGS79iugdGeiNmsYrG0vlIAdK+heQPC2gpGz5GgBbAiuiCIioIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIgIiICIiAiIg/9k=";

    const tennisBallImage = new Image();
    tennisBallImage.src = "data:image/jpeg;base64,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";

    let showScreen = function (screen_opt) {
        SCREEN = screen_opt;
        switch (screen_opt) {
            case SCREEN_SNAKE:
                screen_snake.style.display = "block";
                screen_menu.style.display = "none";
                screen_setting.style.display = "none";
                screen_game_over.style.display = "none";
                break;
            case SCREEN_GAME_OVER:
                screen_snake.style.display = "block";
                screen_menu.style.display = "none";
                screen_setting.style.display = "none";
                screen_game_over.style.display = "block";
                break;
            case SCREEN_SETTING:
                screen_snake.style.display = "none";
                screen_menu.style.display = "none";
                screen_setting.style.display = "block";
                screen_game_over.style.display = "none";
                break;
        }
    };

    window.onload = function () {
        button_new_game.onclick = function () { newGame(); };
        button_new_game1.onclick = function () { newGame(); };
        button_new_game2.onclick = function () { newGame(); };
        button_setting_menu.onclick = function () { showScreen(SCREEN_SETTING); };
        button_setting_menu1.onclick = function () { showScreen(SCREEN_SETTING); };

        setSnakeSpeed(150);
        for (let i = 0; i < speed_setting.length; i++) {
            speed_setting[i].addEventListener("click", function () {
                for (let i = 0; i < speed_setting.length; i++) {
                    if (speed_setting[i].checked) {
                        setSnakeSpeed(speed_setting[i].value);
                    }
                }
            });
        }

        setWall(1);
        for (let i = 0; i < wall_setting.length; i++) {
            wall_setting[i].addEventListener("click", function () {
                for (let i = 0; i < wall_setting.length; i++) {
                    if (wall_setting[i].checked) {
                        setWall(wall_setting[i].value);
                    }
                }
            });
        }

        window.addEventListener("keydown", function (evt) {
            if (evt.code === "Space" && SCREEN !== SCREEN_SNAKE)
                newGame();
        }, true);
    };

    let mainLoop = function () {
        let _x = snake[0].x;
        let _y = snake[0].y;
        snake_dir = snake_next_dir;
        switch (snake_dir) {
            case 0: _y--; break;
            case 1: _x++; break;
            case 2: _y++; break;
            case 3: _x--; break;
        }
        snake.pop();
        snake.unshift({ x: _x, y: _y });

        if (wall === 1) {
            if (snake[0].x < 0 || snake[0].x === canvas.width / BLOCK || snake[0].y < 0 || snake[0].y === canvas.height / BLOCK) {
                showScreen(SCREEN_GAME_OVER);
                return;
            }
        } else {
            for (let i = 0, x = snake.length; i < x; i++) {
                if (snake[i].x < 0) {
                    snake[i].x = snake[i].x + (canvas.width / BLOCK);
                }
                if (snake[i].x === canvas.width / BLOCK) {
                    snake[i].x = snake[i].x - (canvas.width / BLOCK);
                }
                if (snake[i].y < 0) {
                    snake[i].y = snake[i].y + (canvas.height / BLOCK);
                }
                if (snake[i].y === canvas.height / BLOCK) {
                    snake[i].y = snake[i].y - (canvas.height / BLOCK);
                }
            }
        }

        for (let i = 1; i < snake.length; i++) {
            if (snake[0].x === snake[i].x && snake[0].y === snake[i].y) {
                showScreen(SCREEN_GAME_OVER);
                return;
            }
        }

        if (checkBlock(snake[0].x, snake[0].y, food.x, food.y)) {
            snake[snake.length] = { x: snake[0].x, y: snake[0].y };
            altScore(++score);
            addFood();
        }

        ctx.beginPath();
        ctx.fillStyle = "red";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Draw the head as a tennis racket
        ctx.drawImage(tennisRacketImage, snake[0].x * BLOCK, snake[0].y * BLOCK, BLOCK, BLOCK);

        // Draw the rest of the snake
        for (let i = 1; i < snake.length; i++) {
            activeDot(snake[i].x, snake[i].y);
        }

        ctx.drawImage(tennisBallImage, food.x * BLOCK, food.y * BLOCK, BLOCK, BLOCK);

        setTimeout(mainLoop, snake_speed);
    };

    let newGame = function () {
        showScreen(SCREEN_SNAKE);
        screen_snake.focus();
        score = 0;
        altScore(score);
        snake = [];
        snake.push({ x: 0, y: 15 });
        snake_next_dir = 1;
        addFood();
        canvas.onkeydown = function (evt) {
            changeDir(evt.keyCode);
        };
        mainLoop();
    };

    let changeDir = function (key) {
        switch (key) {
            case 37:
                if (snake_dir !== 1)
                    snake_next_dir = 3;
                break;
            case 38:
                if (snake_dir !== 2)
                    snake_next_dir = 0;
                break;
            case 39:
                if (snake_dir !== 3)
                    snake_next_dir = 1;
                break;
            case 40:
                if (snake_dir !== 0)
                    snake_next_dir = 2;
                break;
        }
    };

    let activeDot = function (x, y) {
        ctx.fillStyle = "#1E792C";
        ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
    };

    let addFood = function () {
        food.x = Math.floor(Math.random() * ((canvas.width / BLOCK) - 1));
        food.y = Math.floor(Math.random() * ((canvas.height / BLOCK) - 1));
        for (let i = 1; i < snake.length; i++) {
            if (checkBlock(food.x, food.y, snake[i].x, snake[i].y)) {
                addFood();
            }
        }
    };

    let checkBlock = function (x, y, _x, _y) {
        return (x === _x && y === _y);
    };

    let altScore = function (score_val) {
        ele_score.innerHTML = String(score_val);
    };

    let setSnakeSpeed = function (speed_value) {
        snake_speed = speed_value;
    };

    let setWall = function (wall_value) {
        wall = wall_value;
    };
})();
</script>   