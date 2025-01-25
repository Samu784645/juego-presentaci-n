<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Street Fighter - Vida con Porcentaje</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #87ceeb; /* Fondo cielo */
        }
        canvas {
            display: block;
            margin: 0 auto;
            background: #333; /* Fondo del juego */
        }
        #menu, #store {
            display: none;
            text-align: center;
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-size: cover;
            background-position: center;
        }
        .button {
            padding: 15px 30px;
            margin: 10px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            font-size: 20px;
        }
        #backToMenu {
            display: none;
            position: absolute;
            top: 10px;
            left: 10px;
            padding: 10px 20px;
            background-color: #f44336;
            color: white;
            border: none;
            cursor: pointer;
        }
        .menu-content {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
        }
        .game-title {
            margin-bottom: 20px;
        }
        .game-title img {
            max-width: 100%;
            height: auto;
        }
    </style>
</head>
<body>
    <div id="menu">
        <div class="menu-content">
            <div class="game-title">
                <img src="imagen para el nombre.jpg" alt="Nombre del Juego">
            </div>
            <button class="button" onclick="startGame('single')">Jugar Solo</button>
            <button class="button" onclick="startGame('multi')">Multijugador</button>
            <button class="button" onclick="openStore()">Tienda</button>
        </div>
    </div>
    <div id="store">
        <h2>Tienda</h2>
        <p>Monedas: <span id="coins">0</span></p>
        <button class="button" onclick="buyColor('red')">Comprar Color Rojo</button>
        <button class="button" onclick="buyColor('blue')">Comprar Color Azul</button>
        <button class="button" onclick="closeStore()">Cerrar Tienda</button>
    </div>
    <button id="backToMenu" onclick="returnToMenu()">Volver al Menú</button>
    <canvas id="gameCanvas"></canvas>
    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        canvas.width = 800;
        canvas.height = 400;

        let coins = 0;
        let singlePlayer = true;

        const player1 = {
            x: 50,
            y: canvas.height - 100,
            width: 50,
            height: 100,
            color: "red",
            speed: 5,
            health: 100,
            powerCooldown: false
        };

        const player2 = {
            x: canvas.width - 100,
            y: canvas.height - 100,
            width: 50,
            height: 100,
            color: "blue",
            speed: 5,
            health: 100,
            powerCooldown: false
        };

        const powers = [];
        let keys = {};
        let gameOver = false;

        const backgroundImage = new Image();
        backgroundImage.src = 'imagen para juego.jpg';

        const menuBackgroundImage = new Image();
        menuBackgroundImage.src = 'imagen para menu.jpg';
        menuBackgroundImage.onload = () => {
            document.getElementById('menu').style.backgroundImage = `url('imagen para menu.jpg')`;
            document.getElementById('menu').style.display = 'block';
        };

        window.addEventListener("keydown", (e) => keys[e.key] = true);
        window.addEventListener("keyup", (e) => keys[e.key] = false);

        function startGame(mode) {
            singlePlayer = (mode === 'single');
            document.getElementById('menu').style.display = 'none';
            document.getElementById('backToMenu').style.display = 'block';
            canvas.style.display = 'block';
            gameOver = false;
            player1.health = 100;
            player2.health = 100;
            player1.x = 50;
            player2.x = canvas.width - 100;
            powers.length = 0;
            gameLoop();
        }

        function openStore() {
            document.getElementById('menu').style.display = 'none';
            document.getElementById('store').style.display = 'block';
        }

        function closeStore() {
            document.getElementById('store').style.display = 'none';
            document.getElementById('menu').style.display = 'block';
        }

        function returnToMenu() {
            gameOver = true;
            document.getElementById('backToMenu').style.display = 'none';
            document.getElementById('menu').style.display = 'block';
            canvas.style.display = 'none';
        }

        function buyColor(color) {
            if (coins >= 100) {
                coins -= 100;
                player1.color = color;
                document.getElementById('coins').innerText = coins;
            } else {
                alert('No tienes suficientes monedas');
            }
        }

        function drawPlayer(player) {
            ctx.fillStyle = player.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);
        }

        function drawHealth(player, x, color) {
            ctx.fillStyle = color;
            ctx.fillRect(x, 10, player.health * 2, 20);

            // Dibujar porcentaje de vida encima de la barra
            ctx.fillStyle = "white";
            ctx.font = "16px Arial";
            ctx.fillText(`${player.health}%`, x + 80, 25);
        }

        function drawPowers() {
            powers.forEach((power, index) => {
                ctx.fillStyle = power.color;
                ctx.fillRect(power.x, power.y, power.width, power.height);
                power.x += power.speed;

                const target = power.owner === player1 ? player2 : player1;
                if (power.x < target.x + target.width &&
                    power.x + power.width > target.x &&
                    power.y < target.y + target.height &&
                    power.y + power.height > target.y) {
                    target.health = Math.max(target.health - 5, 0);
                    powers.splice(index, 1);
                }

                if (power.x > canvas.width || power.x + power.width < 0) {
                    powers.splice(index, 1);
                }
            });
        }

        function update() {
            if (gameOver) return;

            if (keys["w"] && player1.y > 0) player1.y -= player1.speed;
            if (keys["s"] && player1.y + player1.height < canvas.height) player1.y += player1.speed;
            if (keys["a"] && player1.x > 0) player1.x -= player1.speed;
            if (keys["d"] && player1.x + player1.width < canvas.width) player1.x += player1.speed;
            if (keys["f"] && !player1.powerCooldown) {
                powers.push({
                    x: player1.x + player1.width,
                    y: player1.y + player1.height / 2 - 10,
                    width: 20,
                    height: 10,
                    speed: 7,
                    color: "orange",
                    owner: player1
                });
                player1.powerCooldown = true;
                setTimeout(() => player1.powerCooldown = false, 500);
            }

            if (!singlePlayer) {
                if (keys["ArrowUp"] && player2.y > 0) player2.y -= player2.speed;
                if (keys["ArrowDown"] && player2.y + player2.height < canvas.height) player2.y += player2.speed;
                if (keys["ArrowLeft"] && player2.x > 0) player2.x -= player2.speed;
                if (keys["ArrowRight"] && player2.x + player2.width < canvas.width) player2.x += player2.speed;
                if (keys["m"] && !player2.powerCooldown) {
                    powers.push({
                        x: player2.x - 20,
                        y: player2.y + player2.height / 2 - 10,
                        width: 20,
                        height: 10,
                        speed: -7,
                        color: "cyan",
                        owner: player2
                    });
                    player2.powerCooldown = true;
                    setTimeout(() => player2.powerCooldown = false, 500);
                }
            } else {
                // Lógica de IA para el jugador 2
                const distanceX = Math.abs(player1.x - player2.x);
                const distanceY = Math.abs(player1.y - player2.y);

                if (distanceX < 200 && distanceY < 100) {
                    if (player2.y > player1.y) player2.y -= player2.speed;
                    if (player2.y + player2.height < player1.y + player1.height) player2.y += player2.speed;
                    if (player2.x > player1.x) player2.x -= player2.speed;
                    if (player2.x + player2.width < player1.x + player1.width) player2.x += player2.speed;
                }

                if (!player2.powerCooldown && distanceX < 300) {
                    powers.push({
                        x: player2.x - 20,
                        y: player2.y + player2.height / 2 - 10,
                        width: 20,
                        height: 10,
                        speed: -7,
                        color: "cyan",
                        owner: player2
                    });
                    player2.powerCooldown = true;
                    setTimeout(() => player2.powerCooldown = false, 500);
                }
            }

            // Lógica de colisión para evitar que los jugadores se fusionen
            if (player1.x < player2.x + player2.width &&
                player1.x + player1.width > player2.x &&
                player1.y < player2.y + player2.height &&
                player1.y + player1.height > player2.y) {
                // Colisión detectada, ajustar posiciones
                const overlapX = (player1.x + player1.width / 2) - (player2.x + player2.width / 2);
                const overlapY = (player1.y + player1.height / 2) - (player2.y + player2.height / 2);

                if (Math.abs(overlapX) > Math.abs(overlapY)) {
                    if (overlapX > 0) {
                        player1.x = player2.x + player2.width;
                    } else {
                        player1.x = player2.x - player1.width;
                    }
                } else {
                    if (overlapY > 0) {
                        player1.y = player2.y + player2.height;
                    } else {
                        player1.y = player2.y - player1.height;
                    }
                }
            }

            // Limitar los movimientos de los jugadores dentro del canvas
            player1.x = Math.max(0, Math.min(player1.x, canvas.width - player1.width));
            player1.y = Math.max(0, Math.min(player1.y, canvas.height - player1.height));
            player2.x = Math.max(0, Math.min(player2.x, canvas.width - player2.width));
            player2.y = Math.max(0, Math.min(player2.y, canvas.height - player2.height));
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Dibujar la imagen de fondo
            ctx.drawImage(backgroundImage, 0, 0, canvas.width, canvas.height);

            drawPlayer(player1);
            drawPlayer(player2);

            drawHealth(player1, 10, "red");
            drawHealth(player2, canvas.width - 210, "blue");

            drawPowers();
        }

        function gameLoop() {
            if (gameOver) return;

            update();
            draw();

            if (player1.health > 0 && player2.health > 0) {
                requestAnimationFrame(gameLoop);
            } else {
                gameOver = true;
                ctx.fillStyle = "white";
                ctx.font = "30px Arial";
                const winner = player1.health > 0 ? "Player 1" : "Player 2";
                ctx.fillText(`${winner} Wins!`, canvas.width / 2 - 100, canvas.height / 2);
                ctx.fillText("Press Enter to Restart", canvas.width / 2 - 150, canvas.height / 2 + 40);

                if (winner === "Player 1") {
                    coins += 100;
                    document.getElementById('coins').innerText = coins;
                }
            }
        }

        function restartGame() {
            player1.health = 100;
            player2.health = 100;
            player1.x = 50;
            player2.x = canvas.width - 100;
            powers.length = 0;
            gameOver = false;
            gameLoop();
        }

        window.addEventListener("keydown", (e) => {
            if (e.key === "Enter" && gameOver) restartGame();
        });

        // Esperar a que la imagen de fondo se cargue antes de mostrar el menú
        backgroundImage.onload = () => {
            document.getElementById('menu').style.display = 'block';
        };

        document.getElementById('menu').style.display = 'block';
    </script>
</body>
</html>
