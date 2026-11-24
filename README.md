<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Neon Runner: Fuga de Dados</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      user-select: none;
    }

    body {
      background-color: #0b0510;
      color: #fff;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      overflow: hidden;
    }

    .game-header {
      text-align: center;
      margin-bottom: 10px;
    }

    h1 {
      font-family: 'Press Start 2P', monospace;
      color: #00e5ff;
      font-size: 22px;
      text-shadow: 0 0 10px #00e5ff, 0 0 20px #00e5ff;
      margin-bottom: 15px;
    }

    .instructions {
      color: #b3c2d1;
      font-size: 13px;
      line-height: 1.6;
      background: #151020;
      padding: 10px;
      border: 2px solid #3a1c54;
      border-radius: 8px;
      max-width: 800px;
    }

    .instructions b {
      color: #ff0055;
    }

    .game-container {
      position: relative;
      width: 800px;
      height: 400px;
      box-shadow: 0 0 30px rgba(0, 229, 255, 0.15);
      border: 4px solid #00e5ff;
      border-radius: 8px;
      background: linear-gradient(180deg, #090412 0%, #1c0b2b 100%);
    }

    canvas {
      display: block;
      border-radius: 4px;
    }

    .ui-overlay {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(5, 2, 10, 0.9);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      border-radius: 4px;
      z-index: 10;
    }

    .hidden {
      display: none !important;
    }

    .ui-title {
      font-family: 'Press Start 2P', monospace;
      font-size: 32px;
      margin-bottom: 15px;
      text-align: center;
      text-transform: uppercase;
    }

    .ui-text {
      font-size: 14px;
      margin-bottom: 30px;
      color: #ccc;
      font-family: 'Press Start 2P', monospace;
      text-align: center;
      line-height: 1.5;
    }

    button {
      background: linear-gradient(180deg, #00e5ff 0%, #0099aa 100%);
      color: #000;
      border: none;
      padding: 15px 30px;
      font-size: 16px;
      font-weight: bold;
      border-radius: 6px;
      cursor: pointer;
      font-family: 'Press Start 2P', monospace;
      transition: 0.2s;
      box-shadow: 0 0 15px #00e5ff;
    }

    button:hover {
      transform: scale(1.05);
      background: linear-gradient(180deg, #ff0055 0%, #cc0044 100%);
      box-shadow: 0 0 20px #ff0055;
      color: #fff;
    }
  </style>
</head>
<body>

  <div class="game-header">
    <h1>NEON RUNNER: FUGA DE DADOS</h1>
    <div class="instructions">
      <b>[W][A][S][D]</b> ou <b>Setas</b>: Mover | <b>[Espaço]</b>: Pular<br>
      <b>História:</b> A Megacorporação roubou seus dados! Atravesse os distritos cibernéticos, desvie dos guardas robóticos e recupere o Núcleo de Dados antes que o sistema reinicie!
    </div>
  </div>

  <div class="game-container">
    <canvas id="gameCanvas" width="800" height="400"></canvas>
   
    <div id="uiOverlay" class="ui-overlay hidden">
      <h2 id="uiTitle" class="ui-title"></h2>
      <p id="uiText" class="ui-text"></p>
      <button onclick="resetGame()">REINICIAR SISTEMA</button>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const uiOverlay = document.getElementById('uiOverlay');
    const uiTitle = document.getElementById('uiTitle');
    const uiText = document.getElementById('uiText');

    let gameState = 'PLAYING';
    let score = 0;
    let lives = 3;
    let currentLevel = 1;
    let frameCount = 0;

    const GRAVITY = 0.55;
    const JUMP_POWER = -11;
    const PLAYER_SPEED = 4;

    const keys = { right: false, left: false, up: false };

    window.addEventListener('keydown', (e) => {
      if (['ArrowRight', 'd', 'D'].includes(e.key)) keys.right = true;
      if (['ArrowLeft', 'a', 'A'].includes(e.key)) keys.left = true;
      if (['ArrowUp', 'w', 'W', ' '].includes(e.key)) keys.up = true;
    });

    window.addEventListener('keyup', (e) => {
      if (['ArrowRight', 'd', 'D'].includes(e.key)) keys.right = false;
      if (['ArrowLeft', 'a', 'A'].includes(e.key)) keys.left = false;
      if (['ArrowUp', 'w', 'W', ' '].includes(e.key)) keys.up = false;
    });

    let player = { x: 30, y: 250, w: 24, h: 36, dx: 0, dy: 0, grounded: false, invul: 0 };
    let platforms = [];
    let enemies = [];
    let coins = [];
    let particles = [];
    let levelGoal = { x: 740, y: 60, w: 30, h: 30 };

    function initLevel() {
      player = { x: 30, y: 250, w: 24, h: 36, dx: 0, dy: 0, grounded: false, invul: 0 };
     
      // Criar partículas cibernéticas (chuva de dados neon)
      particles = [];
      for(let i=0; i<60; i++) {
        particles.push({ 
          x: Math.random() * 800, 
          y: Math.random() * 400, 
          size: Math.random() * 2 + 1,
          color: Math.random() > 0.5 ? '#00e5ff' : '#ff0055',
          speed: Math.random() * 0.5 + 0.2
        });
      }

      if (currentLevel === 1) {
        platforms = [
          { x: 0, y: 360, w: 800, h: 40, type: 'ground' },    
          { x: 200, y: 280, w: 120, h: 16, type: 'plat' },
          { x: 400, y: 200, w: 120, h: 16, type: 'plat' },
          { x: 600, y: 120, w: 150, h: 16, type: 'plat' }
        ];
        enemies = [
          { name: 'Guarda', x: 400, y: 324, w: 26, h: 36, dx: 1.5, minX: 300, maxX: 700,
            cHair: '#555555', cShirt: '#222222', cEye: '#ff0055' }
        ];
        coins = [ { x: 250, y: 240, collected: false }, { x: 450, y: 160, collected: false } ];
        levelGoal = { x: 680, y: 80, w: 30, h: 30 };
      }
      else if (currentLevel === 2) {
        platforms = [
          { x: 0, y: 360, w: 200, h: 40, type: 'ground' },      
          { x: 300, y: 360, w: 200, h: 40, type: 'ground' },
          { x: 600, y: 360, w: 200, h: 40, type: 'ground' },
          { x: 150, y: 260, w: 100, h: 16, type: 'plat' },
          { x: 350, y: 180, w: 100, h: 16, type: 'plat' },
          { x: 550, y: 260, w: 100, h: 16, type: 'plat' }
        ];
        enemies = [
          { name: 'Drone', x: 320, y: 324, w: 24, h: 36, dx: 2, minX: 300, maxX: 480,
            cHair: '#cccccc', cShirt: '#111111', cEye: '#00e5ff' },
          { name: 'Ciborgue', x: 350, y: 144, w: 26, h: 36, dx: 1, minX: 350, maxX: 430,
            cHair: '#ff0055', cShirt: '#330022', cEye: '#ffff00' }
        ];
        coins = [ { x: 180, y: 220, collected: false }, { x: 580, y: 220, collected: false } ];
        levelGoal = { x: 700, y: 320, w: 30, h: 30 };
      }
      else if (currentLevel === 3) {
        platforms = [
          { x: 0, y: 360, w: 150, h: 40, type: 'ground' },    
          { x: 200, y: 280, w: 80, h: 16, type: 'plat' },
          { x: 350, y: 200, w: 80, h: 16, type: 'plat' },
          { x: 500, y: 120, w: 80, h: 16, type: 'plat' },
          { x: 650, y: 120, w: 150, h: 16, type: 'plat' }
        ];
        enemies = [
          { name: 'Sentinela', x: 200, y: 244, w: 26, h: 36, dx: 0.5, minX: 200, maxX: 260,
            cHair: '#555555', cShirt: '#222222', cEye: '#ff0055' },
          { name: 'Chefe', x: 500, y: 84, w: 24, h: 36, dx: 0.5, minX: 500, maxX: 560,
            cHair: '#ffffff', cShirt: '#441111', cEye: '#ff0000' }
        ];
        coins = [ { x: 390, y: 150, collected: false }, { x: 540, y: 70, collected: false } ];
        levelGoal = { x: 720, y: 80, w: 30, h: 30 };
      }
    }

    function drawCharacter(ctx, x, y, w, h, cHair, cShirt, cEye) {
      // Pele
      ctx.fillStyle = '#ffcc99';
      ctx.fillRect(x + 4, y, w - 8, 12);
      // Cabelo (Estilo Cyber)
      ctx.fillStyle = cHair;
      ctx.fillRect(x, y - 4, w, 10);
      // Olho / Visor cibernético
      ctx.fillStyle = cEye;
      ctx.fillRect(x + 14, y + 4, 8, 4);
      ctx.shadowBlur = 5;
      ctx.shadowColor = cEye;
      ctx.fillRect(x + 14, y + 4, 8, 4);
      ctx.shadowBlur = 0;
      // Jaqueta / Roupa
      ctx.fillStyle = cShirt;
      ctx.fillRect(x + 2, y + 12, w - 4, 16);
      // Detalhe da roupa
      ctx.fillStyle = '#111';
      ctx.fillRect(x + 10, y + 12, 4, 16);
      // Calças
      ctx.fillStyle = '#222';
      ctx.fillRect(x + 4, y + 28, w - 8, 8);
    }

    function update() {
      if (gameState !== 'PLAYING') return;

      if (keys.right) player.dx = PLAYER_SPEED;
      else if (keys.left) player.dx = -PLAYER_SPEED;
      else player.dx = 0;

      if (keys.up && player.grounded) {
        player.dy = JUMP_POWER;
        player.grounded = false;
      }

      player.dy += GRAVITY;
      player.x += player.dx;
      player.y += player.dy;

      if (player.invul > 0) player.invul--;

      if (player.x < 0) player.x = 0;
      if (player.x + player.w > canvas.width) player.x = canvas.width - player.w;

      if (player.y > canvas.height) takeDamage();

      player.grounded = false;
      for (let p of platforms) {
        if (player.x < p.x + p.w && player.x + player.w > p.x &&
            player.y + player.h > p.y && player.y + player.h < p.y + player.dy + 2) {
          player.grounded = true;
          player.dy = 0;
          player.y = p.y - player.h;
        }
      }

      for (let c of coins) {
        if (!c.collected && Math.hypot((player.x + player.w/2) - c.x, (player.y + player.h/2) - c.y) < 20) {
          c.collected = true;
          score += 150;
        }
      }

      for (let e of enemies) {
        e.x += e.dx;
        if (e.x < e.minX || e.x + e.w > e.maxX) e.dx *= -1;

        if (player.invul === 0 &&
            player.x < e.x + e.w && player.x + player.w > e.x &&
            player.y < e.y + e.h && player.y + player.h > e.y) {
          takeDamage();
        }
      }

      if (player.x < levelGoal.x + levelGoal.w && player.x + player.w > levelGoal.x &&
          player.y < levelGoal.y + levelGoal.h && player.y + player.h > levelGoal.y) {
        advanceLevel();
      }

      // Atualizar partículas
      for(let p of particles) {
        p.y -= p.speed;
        if(p.y < 0) p.y = canvas.height;
      }
    }

    function advanceLevel() {
      score += 500;
      if (currentLevel < 3) {
        currentLevel++;
        initLevel();
      } else {
        winGame();
      }
    }

    function takeDamage() {
      lives--;
      if (lives <= 0) {
        gameState = 'GAMEOVER';
        uiTitle.innerText = "SISTEMA COMPROMETIDO";
        uiTitle.style.color = "#ff0055";
        uiTitle.style.textShadow = "0 0 15px #ff0055";
        uiText.innerHTML = "Os guardas da corporação te capturaram.<br>Seus dados foram apagados.";
        uiOverlay.classList.remove('hidden');
      } else {
        player.x = 30;
        player.y = 250;
        player.dy = 0;
        player.invul = 60;
      }
    }

    function winGame() {
      gameState = 'WIN';
      score += lives * 1000;
      uiTitle.innerText = "UPLOAD CONCLUÍDO";
      uiTitle.style.color = "#00e5ff";
      uiTitle.style.textShadow = "0 0 15px #00e5ff";
      uiText.innerHTML = `Dados extraídos com sucesso!<br>A Megacorporação caiu.<br><br>Pontuação Final: ${score}`;
      uiOverlay.classList.remove('hidden');
    }

    window.resetGame = function() {
      uiOverlay.classList.add('hidden');
      score = 0;
      lives = 3;
      currentLevel = 1;
      initLevel();
      gameState = 'PLAYING';
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Desenhar partículas de dados
      for(let p of particles) {
        ctx.fillStyle = p.color;
        ctx.globalAlpha = 0.5;
        ctx.fillRect(p.x, p.y, p.size, p.size * 3);
      }
      ctx.globalAlpha = 1.0;
     
      ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
      ctx.font = '12px "Press Start 2P"';
     
      if (currentLevel === 1) {
        ctx.fillText("SETOR 1: Perímetro Externo", 20, 100);
        ctx.fillText("Busque o ponto de acesso...", 20, 130);
      } else if (currentLevel === 2) {
        ctx.fillText("SETOR 2: Dutos Neon", 20, 100);
        ctx.fillText("Sinal instável, cuidado onde pisa.", 20, 130);
      } else if (currentLevel === 3) {
        ctx.fillText("SETOR 3: Servidor Central", 20, 100);
        ctx.fillText("O Núcleo de Dados está no topo!", 20, 130);
      }

      for (let p of platforms) {
        // Base da plataforma
        ctx.fillStyle = '#1c0b2b';
        ctx.fillRect(p.x, p.y, p.w, p.h);
        
        // Linha neon da plataforma
        ctx.fillStyle = p.type === 'ground' ? '#ff0055' : '#00e5ff';
        ctx.shadowBlur = 10;
        ctx.shadowColor = ctx.fillStyle;
        ctx.fillRect(p.x, p.y, p.w, 3);
        ctx.shadowBlur = 0;
        
        // Grade tecnológica na plataforma
        ctx.strokeStyle = 'rgba(0, 229, 255, 0.1)';
        ctx.lineWidth = 1;
        for(let i=0; i < p.w; i+=20) {
          ctx.beginPath();
          ctx.moveTo(p.x + i, p.y + 3);
          ctx.lineTo(p.x + i, p.y + p.h);
          ctx.stroke();
        }
      }

      for (let c of coins) {
        if (!c.collected) {
          ctx.fillStyle = '#ffff00';
          ctx.shadowBlur = 10;
          ctx.shadowColor = '#ffff00';
          ctx.fillRect(c.x - 6, c.y - 6, 12, 12);
          ctx.fillStyle = '#fff';
          ctx.fillRect(c.x - 3, c.y - 3, 6, 6);
          ctx.shadowBlur = 0;
        }
      }

      // Desenhar objetivo final (Núcleo de Dados / Disquete)
      ctx.fillStyle = '#ff0055';
      ctx.shadowBlur = 15;
      ctx.shadowColor = '#ff0055';
      ctx.fillRect(levelGoal.x, levelGoal.y, levelGoal.w, levelGoal.h);
      ctx.fillStyle = '#00e5ff';
      ctx.fillRect(levelGoal.x + 4, levelGoal.y + 4, levelGoal.w - 8, levelGoal.h - 8);
      ctx.fillStyle = '#ffffff';
      ctx.fillRect(levelGoal.x + 8, levelGoal.y + 8, levelGoal.w - 16, levelGoal.h - 16);
      ctx.shadowBlur = 0;

      for (let e of enemies) {
        drawCharacter(ctx, e.x, e.y, e.w, e.h, e.cHair, e.cShirt, e.cEye);
      }

      if (gameState !== 'GAMEOVER') {
        if (player.invul % 10 < 5) {
          // Personagem: Cabelo Neon Magenta, Jaqueta Escura, Visor Ciano
          drawCharacter(ctx, player.x, player.y, player.w, player.h, '#ff00aa', '#111122', '#00e5ff');
        }
      }

      ctx.fillStyle = '#fff';
      ctx.font = '12px "Press Start 2P"';
      ctx.fillText(`PTS: ${score}`, 15, 30);
      ctx.fillText(`SETOR: ${currentLevel}/3`, 320, 30);
     
      ctx.fillStyle = '#00e5ff';
      ctx.fillText(`LIGAÇÕES: ${lives}`, canvas.width - 150, 30);
    }

    function gameLoop() {
      update();
      draw();
      frameCount++;
      requestAnimationFrame(gameLoop);
    }

    initLevel();
    gameLoop();
  </script>
</body>
</html>
