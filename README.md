# index.html
index.html (для кода)
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>mifexc Mini App</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background: var(--tg-theme-bg-color, #ffffff); color: var(--tg-theme-text-color, #000000); height: 100vh; display: flex; flex-direction: column; overflow: hidden; }
        
        /* Основное содержимое */
        #main-content { flex: 1; display: flex; flex-direction: column; justify-content: center; align-items: center; gap: 20px; padding: 20px; }
        
        .main-btn {
            display: block; width: 80%; max-width: 300px; padding: 18px;
            text-align: center; text-decoration: none; border-radius: 12px;
            font-size: 18px; font-weight: 600;
            background: var(--tg-theme-button-color, #2481cc);
            color: var(--tg-theme-button-text-color, #ffffff);
            transition: opacity 0.2s;
        }
        .main-btn:active { opacity: 0.8; }

        /* Игровая зона */
        #game-container { display: none; flex-direction: column; align-items: center; justify-content: center; flex: 1; padding: 10px; }
        canvas { border: 1px solid #ccc; border-radius: 8px; background: #f7f7f7; max-width: 100%; }
        #game-score { margin-top: 10px; font-size: 20px; font-weight: bold; }

        /* Нижняя навигация */
        #bottom-nav {
            display: flex; height: 60px; border-top: 1px solid var(--tg-theme-hint-color, #e0e0e0);
            background: var(--tg-theme-secondary-bg-color, #f1f1f1);
        }
        .nav-btn {
            flex: 1; display: flex; align-items: center; justify-content: center;
            font-size: 16px; font-weight: 500; border: none; background: transparent;
            color: var(--tg-theme-hint-color, #999999); cursor: pointer;
        }
        .nav-btn.active { color: var(--tg-theme-button-color, #2481cc); }
    </style>
</head>
<body>

    <!-- Главная страница -->
    <div id="main-content">
        <a href="https://t.me/mifexc_rubot" class="main-btn">Связаться</a>
        <a href="https://t.me/mifexc" class="main-btn">Профиль</a>
    </div>

    <!-- Игровая страница -->
    <div id="game-container">
        <canvas id="gameCanvas" width="300" height="150"></canvas>
        <div id="game-score">Счёт: 0</div>
    </div>

    <!-- Нижняя навигация -->
    <div id="bottom-nav">
        <button class="nav-btn active" onclick="switchTab('main')">Главная</button>
        <button class="nav-btn" onclick="switchTab('game')">Мини-игра</button>
    </div>

    <script>
        // Инициализация Telegram WebApp
        const tg = window.Telegram.WebApp;
        tg.expand(); // Развернуть на весь экран

        // Переключение вкладок
        function switchTab(tab) {
            const main = document.getElementById('main-content');
            const game = document.getElementById('game-container');
            const navBtns = document.querySelectorAll('.nav-btn');
            
            navBtns.forEach(btn => btn.classList.remove('active'));

            if (tab === 'main') {
                main.style.display = 'flex';
                game.style.display = 'none';
                navBtns[0].classList.add('active');
                stopGame();
            } else {
                main.style.display = 'none';
                game.style.display = 'flex';
                navBtns[1].classList.add('active');
                startGame();
            }
        }

        // ================= ЛОГИКА ИГРЫ С ДИНОЗАВРОМ =================
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('game-score');
        
        let gameLoop, gameOver = false, score = 0, speed = 3;
        let dino = { x: 30, y: 110, w: 20, h: 30, vy: 0, jumping: false };
        let obstacles = [];
        let gravity = 0.6, jumpForce = -12;
        let obstacleTimer = 0;

        function startGame() {
            // Сброс состояния
            gameOver = false;
            score = 0;
            speed = 3;
            dino.y = 110;
            dino.vy = 0;
            dino.jumping = false;
            obstacles = [];
            obstacleTimer = 0;
            scoreElement.innerText = `Счёт: ${score}`;
            
            if (gameLoop) cancelAnimationFrame(gameLoop);
            gameLoop = requestAnimationFrame(updateGame);
        }

        function stopGame() {
            if (gameLoop) cancelAnimationFrame(gameLoop);
        }

        function jump() {
            if (!dino.jumping && !gameOver && document.getElementById('game-container').style.display === 'flex') {
                dino.vy = jumpForce;
                dino.jumping = true;
            } else if (gameOver) {
                startGame(); // Рестарт при клике после проигрыша
            }
        }

        // Обработка нажатий (тап по экрану или пробел)
        canvas.addEventListener('touchstart', (e) => { e.preventDefault(); jump(); });
        canvas.addEventListener('mousedown', jump);
        window.addEventListener('keydown', (e) => { if (e.code === 'Space') { e.preventDefault(); jump(); }});

        function updateGame() {
            if (gameOver) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#333';
                ctx.font = '20px sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText('Игра окончена', canvas.width/2, canvas.height/2 - 10);
                ctx.font = '14px sans-serif';
                ctx.fillText('Нажмите, чтобы начать заново', canvas.width/2, canvas.height/2 + 20);
                return;
            }

            // Физика динозавра
            dino.vy += gravity;
            dino.y += dino.vy;
            if (dino.y > 110) {
                dino.y = 110;
                dino.vy = 0;
                dino.jumping = false;
            }

            // Генерация препятствий
            obstacleTimer++;
            if (obstacleTimer > 100 - speed * 3) { // Ускорение появления
                obstacles.push({ x: canvas.width, y: 120, w: 15, h: 20 });
                obstacleTimer = 0;
            }

            // Движение и проверка столкновений
            for (let i = obstacles.length - 1; i >= 0; i--) {
                let obs = obstacles[i];
                obs.x -= speed;

                // Проверка столкновения (AABB)
                if (dino.x < obs.x + obs.w && dino.x + dino.w > obs.x &&
                    dino.y < obs.y + obs.h && dino.y + dino.h > obs.y) {
                    gameOver = true;
                }

                // Удаление за экраном и начисление очков
                if (obs.x + obs.w < 0) {
                    obstacles.splice(i, 1);
                    score += 10;
                    scoreElement.innerText = `Счёт: ${score}`;
                    if (score % 100 === 0) speed += 0.5; // Постепенное ускорение
                }
            }

            // Отрисовка
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Земля
            ctx.fillStyle = '#ddd';
            ctx.fillRect(0, 140, canvas.width, 10);

            // Динозавр (упрощённый прямоугольник)
            ctx.fillStyle = '#333';
            ctx.fillRect(dino.x, dino.y, dino.w, dino.h);
            
            // Глаз
            ctx.fillStyle = '#fff';
            ctx.fillRect(dino.x + 12, dino.y + 5, 4, 4);

            // Препятствия
            ctx.fillStyle = '#555';
            obstacles.forEach(obs => ctx.fillRect(obs.x, obs.y, obs.w, obs.h));

            gameLoop = requestAnimationFrame(updateGame);
        }
    </script>
</body>
</html>
