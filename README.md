<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Superpool Game</title>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
            background: #2e2e2e;
            color: white;
            user-select: none;
        }
        #status {
            font-size: 20px;
            margin-bottom: 10px;
        }
        #table-container {
            position: relative;
            width: 1200px;
            height: 600px;
            margin: auto;
            background: #0a5d23;
            border: 10px solid #5d3a00;
            border-radius: 10px;
            overflow: hidden;
        }
        #table-container::before {
            content: "";
            position: absolute;
            width: 100%;
            height: 100%;
            background: url('superseed.jpeg') no-repeat center;
            background-size: cover;
            opacity: 0.5;
            filter: blur(1px) brightness(0.9);
            pointer-events: none;
        }
        .ball {
            position: absolute;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            text-align: center;
            line-height: 30px;
            font-weight: bold;
            box-shadow: 0 0 5px rgba(0,0,0,0.5);
        }
        .solid { background: red; color: white; text-shadow: 1px 1px 2px black; }
        .striped { background: white; border: 3px solid red; color: black; text-shadow: 1px 1px 2px white; }
        .cue-ball { background: white; }
        .eight-ball { background: black; color: white; }
        .pocket {
            position: absolute;
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background: rgba(0, 0, 0, 0.7);
        }
        .cue-stick {
            position: absolute;
            width: 150px;
            height: 5px;
            background: tan;
            transform-origin: left center;
            display: none;
        }
    </style>
</head>
<body>
    <div id="status">Loading...</div>
    <div id="table-container">
        <div id="cue-ball" class="ball cue-ball"></div>
        <div id="cue-stick" class="cue-stick"></div>
    </div>

    <script>
        document.addEventListener("DOMContentLoaded", function () {
            const table = document.getElementById("table-container");
            const status = document.getElementById("status");
            const cueBall = document.getElementById("cue-ball");
            const cueStick = document.getElementById("cue-stick");

            let currentPlayer = 1;
            let aiming = false;
            let velocity = { x: 0, y: 0 };

            function updateStatus(text) {
                status.textContent = text;
            }

            function createBall(number, x, y, type) {
                const ball = document.createElement("div");
                ball.classList.add("ball", type);
                ball.textContent = number;
                ball.style.left = `${x}px`;
                ball.style.top = `${y}px`;
                ball.dataset.number = number;
                table.appendChild(ball);
                return ball;
            }

            function createPocket(x, y) {
                const pocket = document.createElement("div");
                pocket.classList.add("pocket");
                pocket.style.left = `${x}px`;
                pocket.style.top = `${y}px`;
                table.appendChild(pocket);
            }

            function setupTable() {
                updateStatus(`Player ${currentPlayer}'s turn`);
                
                const startX = 500, startY = 250;
                cueBall.style.left = "100px";
                cueBall.style.top = "285px";

                let rows = 5, index = 1;
                for (let row = 0; row < rows; row++) {
                    for (let i = 0; i <= row; i++) {
                        let x = startX + row * 35;
                        let y = startY - (row * 17) + (i * 34);
                        let type = index === 8 ? "eight-ball" : (index <= 7 ? "solid" : "striped");
                        createBall(index, x, y, type);
                        index++;
                    }
                }

                let pocketPositions = [
                    [0, 0], [575, 0], [1150, 0],
                    [0, 550], [575, 550], [1150, 550]
                ];
                pocketPositions.forEach(pos => createPocket(pos[0], pos[1]));
            }

            table.addEventListener("mousedown", function (e) {
                aiming = true;
                cueStick.style.display = "block";
                cueStick.style.left = `${cueBall.offsetLeft + 15}px`;
                cueStick.style.top = `${cueBall.offsetTop + 15}px`;
            });

            table.addEventListener("mousemove", function (e) {
                if (!aiming) return;
                let dx = e.clientX - cueBall.offsetLeft - 15;
                let dy = e.clientY - cueBall.offsetTop - 15;
                let angle = Math.atan2(dy, dx);
                cueStick.style.transform = `rotate(${angle}rad)`;
            });

            table.addEventListener("mouseup", function (e) {
                if (!aiming) return;
                aiming = false;
                cueStick.style.display = "none";

                let dx = e.clientX - cueBall.offsetLeft - 15;
                let dy = e.clientY - cueBall.offsetTop - 15;
                let magnitude = Math.min(Math.sqrt(dx * dx + dy * dy) / 10, 10);
                let angle = Math.atan2(dy, dx);
                velocity.x = Math.cos(angle) * magnitude;
                velocity.y = Math.sin(angle) * magnitude;

                updateStatus("Ball in motion...");
                requestAnimationFrame(moveBall);
            });

            function moveBall() {
                if (Math.abs(velocity.x) < 0.1 && Math.abs(velocity.y) < 0.1) {
                    velocity.x = 0;
                    velocity.y = 0;
                    updateStatus(`Player ${currentPlayer}'s turn`);
                    return;
                }

                cueBall.style.left = `${cueBall.offsetLeft + velocity.x}px`;
                cueBall.style.top = `${cueBall.offsetTop + velocity.y}px`;

                velocity.x *= 0.98;
                velocity.y *= 0.98;

                requestAnimationFrame(moveBall);
            }

            setupTable();
        });
    </script>
</body>
</html>
