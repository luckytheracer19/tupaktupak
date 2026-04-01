# tupaktupak
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Offline Runner Game</title>
    <style>
        body {
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f0f0f0;
            font-family: 'Courier New', Courier, monospace;
            overflow: hidden;
            touch-action: manipulation;
        }
        #gameCanvas {
            background-color: #fff;
            border-bottom: 2px solid #555;
            max-width: 100%;
        }
        .ui {
            position: absolute;
            top: 20px;
            text-align: center;
            pointer-events: none;
        }
    </style>
</head>
<body>

    <div class="ui">
        <h1 id="scoreBoard">Score: 0</h1>
        <p id="msg">Press SPACE or TAP to Jump</p>
    </div>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreBoard = document.getElementById('scoreBoard');
        const msg = document.getElementById('msg');

        // Game Settings
        canvas.width = 800;
        canvas.height = 200;
        let score = 0;
        let gameActive = true;
        let gameSpeed = 5;

        // Player Object
        const player = {
            x: 50,
            y: canvas.height - 50,
            width: 40,
            height: 40,
            color: '#ff4757',
            dy: 0,
            jumpForce: 12,
            gravity: 0.6,
            grounded: false,
            
            draw() {
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, this.width, this.height);
            },
            
            jump() {
                if (this.grounded) {
                    this.dy = -this.jumpForce;
                    this.grounded = false;
                }
            },
            
            update() {
                // Gravity logic
                if (!this.grounded) {
                    this.dy += this.gravity;
                    this.y += this.dy;
                }

                // Floor collision
                if (this.y + this.height > canvas.height) {
                    this.y = canvas.height - this.height;
                    this.dy = 0;
                    this.grounded = true;
                }
            }
        };

        // Obstacles
        const obstacles = [];
        function spawnObstacle() {
            const size = Math.random() * (50 - 20) + 20;
            obstacles.push({
                x: canvas.width,
                y: canvas.height - size,
                width: 20,
                height: size,
                color: '#2f3542'
            });
        }

        let frame = 0;
        function animate() {
            if (!gameActive) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            frame++;

            // Draw and Update Player
            player.update();
            player.draw();

            // Handle Obstacles
            if (frame % Math.floor(100 / (gameSpeed / 5)) === 0) {
                spawnObstacle();
            }

            obstacles.forEach((obs, index) => {
                obs.x -= gameSpeed;
                ctx.fillStyle = obs.color;
                ctx.fillRect(obs.x, obs.y, obs.width, obs.height);

                // Collision Detection
                if (
                    player.x < obs.x + obs.width &&
                    player.x + player.width > obs.x &&
                    player.y < obs.y + obs.height &&
                    player.y + player.height > obs.y
                ) {
                    gameOver();
                }

                // Remove off-screen obstacles & increase score
                if (obs.x + obs.width < 0) {
                    obstacles.splice(index, 1);
                    score++;
                    scoreBoard.innerText = `Score: ${score}`;
                    if (score % 10 === 0) gameSpeed += 0.5; // Difficulty increase
                }
            });

            requestAnimationFrame(animate);
        }

        function gameOver() {
            gameActive = false;
            msg.innerText = "GAME OVER! Press SPACE to Restart";
            msg.style.color = "red";
        }

        function resetGame() {
            score = 0;
            gameSpeed = 5;
            obstacles.length = 0;
            gameActive = true;
            scoreBoard.innerText = `Score: 0`;
            msg.innerText = "Press SPACE or TAP to Jump";
            msg.style.color = "black";
            animate();
        }

        // Controls
        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                if (gameActive) player.jump();
                else resetGame();
            }
        });

        canvas.addEventListener('touchstart', (e) => {
            if (gameActive) player.jump();
            else resetGame();
        });

        // Start
        animate();
    </script>
</body>
</html>
