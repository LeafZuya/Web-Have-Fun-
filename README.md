# Web-Have-Fun
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Catch The Emoji - LeafZuya Have Fun</title>
  <style>
    body {
      margin: 0;
      height: 100vh;
      font-family: Arial, sans-serif;
      overflow: hidden;
      text-align: center;
      background: linear-gradient(135deg, #00ffcc, #3399ff);
      background-size: 300% 300%;
      animation: gradientMove 10s ease infinite;
    }

    @keyframes gradientMove {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }

    h1 {
      margin: 10px;
      color: #fff;
      text-shadow: 2px 2px 4px #000;
    }

    #gameCanvas {
      background: rgba(255,255,255,0.85);
      border: 4px solid #333;
      border-radius: 15px;
      display: block;
      margin: 0 auto;
      z-index: 1;
      position: relative;
    }

    #info {
      margin: 10px;
      font-size: 20px;
      font-weight: bold;
      color: #fff;
      text-shadow: 1px 1px 3px #000;
    }

    #gameOver {
      position: absolute;
      top: 40%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(0,0,0,0.8);
      color: #fff;
      padding: 30px;
      border-radius: 15px;
      font-size: 24px;
      display: none;
      z-index: 10;
    }

    button {
      margin-top: 15px;
      padding: 10px 20px;
      font-size: 18px;
      background: linear-gradient(135deg, #00a859, #1abc9c);
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      transition: 0.3s;
    }

    button:hover {
      transform: scale(1.1);
      background: linear-gradient(135deg, #1abc9c, #00a859);
    }

    /* Emoji background animation */
    .emojiRain {
      position: absolute;
      font-size: 24px;
      opacity: 0.7;
      animation: fall linear forwards;
    }

    @keyframes fall {
      0% { transform: translateY(-50px) rotate(0deg); opacity: 1; }
      100% { transform: translateY(100vh) rotate(360deg); opacity: 0; }
    }
  </style>
</head>
<body>
  <h1>🎮 Catch The Emoji 🎮</h1>
  <div id="info">
    Score: <span id="score">0</span> | 
    Lives: <span id="lives">3</span> | 
    High Score: <span id="highScore">0</span>
  </div>
  <canvas id="gameCanvas" width="400" height="600"></canvas>
  <div id="gameOver">
    <p id="finalScore"></p>
    <button onclick="restartGame()">Play Again</button>
  </div>

  <!-- Sound efek -->
  <audio id="catchSound" src="https://www.fesliyanstudios.com/play-mp3/387" preload="auto"></audio>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreEl = document.getElementById("score");
    const livesEl = document.getElementById("lives");
    const highScoreEl = document.getElementById("highScore");
    const gameOverScreen = document.getElementById("gameOver");
    const finalScore = document.getElementById("finalScore");
    const catchSound = document.getElementById("catchSound");

    const emojis = ["🍎", "🍌", "🍓", "🍒", "🍕", "🍔", "🍩", "⭐", "💎", "🎁", "😂", "😎", "🤩", "🔥", "🥳"];
    let basket = { x: canvas.width/2 - 30, y: canvas.height - 50, width: 60, height: 30 };
    let falling = [];
    let score = 0;
    let lives = 3;
    let speed = 2;
    let gameRunning = true;

    // Load high score dari localStorage
    let highScore = localStorage.getItem("emojiHighScore") || 0;
    highScoreEl.textContent = highScore;

    function drawBasket() {
      ctx.fillStyle = "#00a859";
      ctx.fillRect(basket.x, basket.y, basket.width, basket.height);
      ctx.fillStyle = "#fff";
      ctx.font = "20px Arial";
      ctx.fillText("🧺", basket.x + 15, basket.y + 22);
    }

    function spawnEmoji() {
      let emoji = {
        x: Math.random() * (canvas.width - 30),
        y: -30,
        symbol: emojis[Math.floor(Math.random() * emojis.length)]
      };
      falling.push(emoji);
    }

    function drawEmoji(emoji) {
      ctx.font = "30px Arial";
      ctx.fillText(emoji.symbol, emoji.x, emoji.y);
    }

    function update() {
      if (!gameRunning) return;
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawBasket();

      for (let i = 0; i < falling.length; i++) {
        let emoji = falling[i];
        emoji.y += speed;
        drawEmoji(emoji);

        // check catch
        if (
          emoji.y > basket.y - 20 &&
          emoji.x > basket.x - 10 &&
          emoji.x < basket.x + basket.width
        ) {
          score++;
          scoreEl.textContent = score;
          falling.splice(i, 1);
          i--;

          // efek sound + basket bounce
          catchSound.currentTime = 0;
          catchSound.play();
          basketBounce();
          continue;
        }

        // missed
        if (emoji.y > canvas.height) {
          lives--;
          livesEl.textContent = lives;
          falling.splice(i, 1);
          i--;
          if (lives <= 0) endGame();
        }
      }

      requestAnimationFrame(update);
    }

    function endGame() {
      gameRunning = false;

      // Update High Score
      if (score > highScore) {
        highScore = score;
        localStorage.setItem("emojiHighScore", highScore);
      }

      finalScore.innerHTML = `
        Game Over! <br>
        Your Score: ${score} <br>
        High Score: ${highScore}
      `;
      highScoreEl.textContent = highScore;
      gameOverScreen.style.display = "block";
    }

    function restartGame() {
      score = 0;
      lives = 3;
      speed = 2;
      falling = [];
      scoreEl.textContent = score;
      livesEl.textContent = lives;
      gameOverScreen.style.display = "none";
      gameRunning = true;
      update();
    }

    // Basket bounce animasi
    function basketBounce() {
      let originalY = basket.y;
      basket.y -= 5;
      setTimeout(() => basket.y = originalY, 100);
    }

    // controls
    document.addEventListener("keydown", (e) => {
      if (e.key === "ArrowLeft" && basket.x > 0) basket.x -= 30;
      if (e.key === "ArrowRight" && basket.x < canvas.width - basket.width) basket.x += 30;
    });

    // mobile touch controls
    canvas.addEventListener("touchstart", (e) => {
      let touchX = e.touches[0].clientX - canvas.getBoundingClientRect().left;
      basket.x = touchX - basket.width / 2;
    });

    // spawn loop
    setInterval(() => {
      if (gameRunning) {
        spawnEmoji();
        if (speed < 10) speed += 0.05;
      }
    }, 1000);

    update();

    // background emoji rain
    function createEmojiRain() {
      const emoji = document.createElement("div");
      emoji.classList.add("emojiRain");
      emoji.textContent = emojis[Math.floor(Math.random() * emojis.length)];
      emoji.style.left = Math.random() * window.innerWidth + "px";
      emoji.style.animationDuration = (3 + Math.random() * 3) + "s";
      document.body.appendChild(emoji);

      setTimeout(() => emoji.remove(), 5000);
    }
    setInterval(createEmojiRain, 700);
  </script>
  <!-- BACKSOUND -->
  <audio id="Maneh.mp3" autoplay loop>
    <source src="Maneh.mp3" type="audio/mpeg">

  </audio>

  <script>
    // Kadang autoplay diblokir, jadi musik jalan setelah klik pertama
    document.addEventListener("click", () => {
      const music = document.getElementById("bgMusic");
      if (music.paused) {
        music.play();
      }
    });
  </script>
</body>
</html>
