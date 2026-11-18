<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Guardião das Sombras</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
        }

        body {
            background-color: #0f172a;
            color: #f8fafc;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
        }

        .game-container {
            position: relative;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.8);
            border-radius: 12px;
            overflow: hidden;
            border: 3px solid #334155;
            background: #1e293b;
        }

        canvas {
            display: block;
            background: #0f172a;
        }

        /* Overlays (Screens) */
        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(15, 23, 42, 0.85);
            backdrop-filter: blur(4px);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            opacity: 1;
            transition: opacity 0.3s ease;
        }

        .hidden {
            display: none !important;
        }

        h1 {
            font-size: 2.5rem;
            margin-bottom: 0.5rem;
            color: #38bdf8;
            text-shadow: 0 0 10px rgba(56, 189, 248, 0.5);
            text-transform: uppercase;
            letter-spacing: 2px;
        }

        p {
            font-size: 1.1rem;
            margin-bottom: 1.5rem;
            color: #cbd5e1;
            text-align: center;
            max-width: 80%;
            line-height: 1.5;
        }

        .btn {
            background: linear-gradient(135deg, #0284c7, #0369a1);
            color: #ffffff;
            border: none;
            padding: 12px 28px;
            font-size: 1.1rem;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 4px 12px rgba(2, 132, 199, 0.4);
            transition: all 0.2s ease;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .btn:hover {
            background: linear-gradient(135deg, #0369a1, #075985);
            transform: translateY(-2px);
            box-shadow: 0 6px 16px rgba(2, 132, 199, 0.6);
        }

        .btn:active {
            transform: translateY(0);
        }

        /* Controls guide */
        .controls-hint {
            margin-top: 15px;
            display: flex;
            gap: 12px;
            flex-wrap: wrap;
            justify-content: center;
            font-size: 0.85rem;
            color: #94a3b8;
        }

        .key {
            background: #334155;
            padding: 2px 8px;
            border-radius: 4px;
            color: #f1f5f9;
            border: 1px solid #475569;
        }

        /* Mobile controls touch buttons (responsive layout) */
        .touch-controls {
            display: none;
            width: 100%;
            max-width: 800px;
            margin-top: 10px;
            justify-content: space-between;
            padding: 0 10px;
        }

        @media (max-width: 768px) {
            .touch-controls {
                display: flex;
            }
        }

        .touch-group {
            display: flex;
            gap: 8px;
        }

        .t-btn {
            background: #334155;
            color: #fff;
            border: 2px solid #475569;
            width: 50px;
            height: 50px;
            border-radius: 10px;
            font-weight: bold;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.2rem;
            touch-action: manipulation;
        }
    </style>
</head>
<body>

    <div class="game-container">
        <!-- Canvas Game Screen -->
        <canvas id="gameCanvas" width="800" height="450"></canvas>

        <!-- Start Overlay -->
        <div id="startScreen" class="overlay">
            <h1>Guardião das Sombras</h1>
            <p>Colete todas as moedas, derrote inimigos com sua espada, use as barreiras para se esconder e alcance o <strong>Pote de Ouro</strong> no final de cada fase!</p>
            <button class="btn" onclick="startGame()">Iniciar Jogo</button>
            <div class="controls-hint">
                <span><span class="key">A</span> / <span class="key">D</span> ou <span class="key">←</span> <span class="key">→</span> Mover</span>
                <span><span class="key">W</span> / <span class="key">Espaço</span> / <span class="key">↑</span> Pular</span>
                <span><span class="key">S</span> / <span class="key">↓</span> Agachar (Esconder)</span>
                <span><span class="key">J</span> / <span class="key">K</span> Atacar</span>
            </div>
        </div>

        <!-- Game Over Overlay -->
        <div id="gameOverScreen" class="overlay hidden">
            <h1 style="color: #ef4444; text-shadow: 0 0 10px rgba(239, 68, 68, 0.5);">Derrota</h1>
            <p>As sombras dominaram você. Tente novamente!</p>
            <button class="btn" style="background: linear-gradient(135deg, #dc2626, #991b1b);" onclick="restartGame()">Tentar Novamente</button>
        </div>

        <!-- Victory Overlay -->
        <div id="victoryScreen" class="overlay hidden">
            <h1 style="color: #10b981; text-shadow: 0 0 10px rgba(16, 185, 129, 0.5);">Vitória!</h1>
            <p id="victoryMessage">Você coletou todas as moedas e libertou o reino!</p>
            <button class="btn" style="background: linear-gradient(135deg, #059669, #047857);" onclick="restartGame()">Jogar Novamente</button>
        </div>

        <!-- Level Transition Overlay -->
        <div id="levelTransitionScreen" class="overlay hidden" style="background: rgba(15, 23, 42, 0.75);">
            <h1 id="transitionTitle" style="color: #fbbf24; text-shadow: 0 0 15px rgba(251, 191, 36, 0.6);">Fase Concluída!</h1>
            <p id="transitionSubtitle">Avançando para a próxima fase...</p>
        </div>
    </div>

    <!-- Touch controls for mobile users -->
    <div class="touch-controls">
        <div class="touch-group">
            <button class="t-btn" id="btnLeft">←</button>
            <button class="t-btn" id="btnDown">↓</button>
            <button class="t-btn" id="btnRight">→</button>
        </div>
        <div class="touch-group">
            <button class="t-btn" id="btnAttack" style="background: #e11d48;">⚔️</button>
            <button class="t-btn" id="btnJump" style="background: #0284c7;">↑</button>
        </div>
    </div>

    <script>
        // Set Canvas and Context
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // State & Settings
        const GRAVITY = 0.55;
        let gameState = 'START'; // START, PLAYING, TRANSITION, GAMEOVER, VICTORY
        let animationFrameId;
        let currentLevel = 1;
        const maxLevels = 3;
        let globalScore = 0;

        // Background & Environment FX State
        let bgElements = {
            stars: [],
            clouds: [],
            lightningTimer: 0,
            lightningAlpha: 0
        };

        // Input Keyboard state
        const keys = {
            left: false,
            right: false,
            up: false,
            down: false,
            attack: false
        };

        // Player Class
        class Player {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = 28;
                this.normalHeight = 40;
                this.crouchHeight = 24;
                this.height = this.normalHeight;
                this.vx = 0;
                this.vy = 0;
                this.speed = 4.2;
                this.jumpPower = -11.5;
                this.isGrounded = false;
                this.isCrouching = false;
                this.isHidden = false;
                this.facing = 'right';
                this.lives = 3;
                this.score = 0;
                this.invulnerableTimer = 0;

                // Sword Attack Properties
                this.isAttacking = false;
                this.attackTimer = 0;
                this.attackCooldown = 0;
                this.attackBox = { x: 0, y: 0, width: 34, height: 28 };
            }

            update(platforms, barriers) {
                if (this.invulnerableTimer > 0) this.invulnerableTimer--;
                if (this.attackCooldown > 0) this.attackCooldown--;

                // Crouch mechanic
                if (keys.down && this.isGrounded) {
                    if (!this.isCrouching) {
                        this.isCrouching = true;
                        this.y += (this.normalHeight - this.crouchHeight);
                        this.height = this.crouchHeight;
                    }
                } else if (this.isCrouching) {
                    this.isCrouching = false;
                    this.y -= (this.normalHeight - this.crouchHeight);
                    this.height = this.normalHeight;
                }

                // Horizontal movement
                const currentSpeed = this.isCrouching ? this.speed * 0.45 : this.speed;
                if (keys.left) {
                    this.vx = -currentSpeed;
                    this.facing = 'left';
                } else if (keys.right) {
                    this.vx = currentSpeed;
                    this.facing = 'right';
                } else {
                    this.vx = 0;
                }

                // Jump mechanic
                if (keys.up && this.isGrounded && !this.isCrouching) {
                    this.vy = this.jumpPower;
                    this.isGrounded = false;
                }

                // Sword attack trigger
                if (keys.attack && !this.isAttacking && this.attackCooldown === 0 && !this.isCrouching) {
                    this.isAttacking = true;
                    this.attackTimer = 12;
                    this.attackCooldown = 25;
                }

                if (this.isAttacking) {
                    this.attackTimer--;
                    if (this.attackTimer <= 0) {
                        this.isAttacking = false;
                    }
                }

                // Apply Gravity and Velocity
                this.vy += GRAVITY;
                this.x += this.vx;
                this.y += this.vy;

                // Keep inside horizontal screen bounds
                if (this.x < 0) this.x = 0;
                if (this.x + this.width > canvas.width) this.x = canvas.width - this.width;

                // Check collision with platforms
                this.isGrounded = false;
                for (let plat of platforms) {
                    // Moving platform displacement tracking
                    const prevPlatX = plat.prevX !== undefined ? plat.prevX : plat.x;
                    const deltaX = plat.x - prevPlatX;

                    if (
                        this.x + this.width > plat.x &&
                        this.x < plat.x + plat.width &&
                        this.y + this.height >= plat.y &&
                        this.y + this.height <= plat.y + plat.height + Math.abs(this.vy) + 4 &&
                        this.vy >= 0
                    ) {
                        this.y = plat.y - this.height;
                        this.vy = 0;
                        this.isGrounded = true;
                        // Carry player along with moving platform
                        this.x += deltaX;
                    }
                }

                // Calculate attack box position based on direction
                if (this.facing === 'right') {
                    this.attackBox.x = this.x + this.width;
                } else {
                    this.attackBox.x = this.x - this.attackBox.width;
                }
                this.attackBox.y = this.y + (this.height / 2) - (this.attackBox.height / 2);

                // Check hiding mechanic in barriers
                this.isHidden = false;
                if (this.isCrouching) {
                    for (let barrier of barriers) {
                        if (
                            this.x + this.width > barrier.x &&
                            this.x < barrier.x + barrier.width &&
                            this.y + this.height > barrier.y &&
                            this.y < barrier.y + barrier.height
                        ) {
                            this.isHidden = true;
                            break;
                        }
                    }
                }

                // Fall off screen reset/damage check
                if (this.y > canvas.height + 50) {
                    this.takeDamage(1);
                    this.respawn();
                }
            }

            respawn() {
                this.x = 50;
                this.y = 300;
                this.vx = 0;
                this.vy = 0;
            }

            takeDamage(amount) {
                if (this.invulnerableTimer > 0) return;
                this.lives -= amount;
                this.invulnerableTimer = 60; // 1 second of invincibility at 60fps
                if (this.lives <= 0) {
                    triggerGameOver();
                }
            }

            draw() {
                ctx.save();

                // Blink effect when invulnerable
                if (this.invulnerableTimer % 6 > 3) {
                    ctx.globalAlpha = 0.4;
                }

                // Shadow under player
                ctx.fillStyle = 'rgba(0, 0, 0, 0.3)';
                ctx.beginPath();
                ctx.ellipse(this.x + this.width/2, this.y + this.height, this.width/2, 4, 0, 0, Math.PI*2);
                ctx.fill();

                // Player Body (Knight/Guardião)
                // Cloak/Back
                ctx.fillStyle = this.isHidden ? '#475569' : '#0284c7';
                ctx.fillRect(this.x, this.y, this.width, this.height);

                // Helmet/Head
                ctx.fillStyle = '#94a3b8';
                ctx.fillRect(this.x + 4, this.y + 4, this.width - 8, 12);

                // Visor Eye Glow
                ctx.fillStyle = this.isHidden ? '#94a3b8' : '#38bdf8';
                if (this.facing === 'right') {
                    ctx.fillRect(this.x + 16, this.y + 8, 8, 4);
                } else {
                    ctx.fillRect(this.x + 4, this.y + 8, 8, 4);
                }

                // Stealth Shield aura when hidden
                if (this.isHidden) {
                    ctx.strokeStyle = '#38bdf8';
                    ctx.setLineDash([2, 4]);
                    ctx.strokeRect(this.x - 2, this.y - 2, this.width + 4, this.height + 4);
                }

                // Draw Sword Slash Attack animation
                if (this.isAttacking) {
                    ctx.fillStyle = 'rgba(245, 158, 11, 0.8)';
                    ctx.strokeStyle = '#ef4444';
                    ctx.lineWidth = 3;

                    ctx.beginPath();
                    if (this.facing === 'right') {
                        ctx.arc(this.x + this.width, this.y + this.height/2, 28, -Math.PI/3, Math.PI/3, false);
                    } else {
                        ctx.arc(this.x, this.y + this.height/2, 28, Math.PI * 2/3, Math.PI * 4/3, false);
                    }
                    ctx.stroke();
                }

                ctx.restore();
            }
        }

        // Enemy Class
        class Enemy {
            constructor(x, y, rangeLeft, rangeRight, speed = 1.8, maxHp = 1) {
                this.x = x;
                this.y = y;
                this.width = maxHp > 1 ? 34 : 30;
                this.height = maxHp > 1 ? 42 : 36;
                this.rangeLeft = rangeLeft;
                this.rangeRight = rangeRight;
                this.speed = speed;
                this.vx = speed;
                this.maxHp = maxHp;
                this.hp = maxHp;
                this.hitTimer = 0;
                this.isAlive = true;
            }

            takeDamage(amount) {
                if (this.hitTimer > 0) return;
                this.hp -= amount;
                this.hitTimer = 20; // Brief invulnerability frames after being slashed
                if (this.hp <= 0) {
                    this.isAlive = false;
                    return true; // Defeated
                }
                return false;
            }

            update() {
                if (!this.isAlive) return;
                if (this.hitTimer > 0) this.hitTimer--;

                // Patrol movement
                this.x += this.vx;
                if (this.x <= this.rangeLeft) {
                    this.x = this.rangeLeft;
                    this.vx = this.speed;
                } else if (this.x + this.width >= this.rangeRight) {
                    this.x = this.rangeRight - this.width;
                    this.vx = -this.speed;
                }
            }

            draw() {
                if (!this.isAlive) return;

                ctx.save();
                if (this.hitTimer % 4 > 1) {
                    ctx.globalAlpha = 0.5;
                }

                // Shadow enemy body (Armored purple/crimson for 2 HP enemies)
                ctx.fillStyle = this.maxHp > 1 ? '#581c87' : '#881337';
                ctx.fillRect(this.x, this.y, this.width, this.height);

                // Armor trim for tough enemies
                if (this.maxHp > 1) {
                    ctx.strokeStyle = '#c084fc';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(this.x + 2, this.y + 2, this.width - 4, this.height - 4);
                }

                // Glowing evil eyes
                ctx.fillStyle = this.maxHp > 1 ? '#facc15' : '#f43f5e';
                if (this.vx > 0) {
                    ctx.fillRect(this.x + this.width - 12, this.y + 10, 6, 6);
                    ctx.fillRect(this.x + this.width - 22, this.y + 10, 6, 6);
                } else {
                    ctx.fillRect(this.x + 6, this.y + 10, 6, 6);
                    ctx.fillRect(this.x + 16, this.y + 10, 6, 6);
                }

                // Horns/Spikes
                ctx.fillStyle = '#4c0519';
                ctx.beginPath();
                ctx.moveTo(this.x + 4, this.y);
                ctx.lineTo(this.x + 8, this.y - 8);
                ctx.lineTo(this.x + 12, this.y);
                ctx.fill();

                ctx.beginPath();
                ctx.moveTo(this.x + this.width - 12, this.y);
                ctx.lineTo(this.x + this.width - 8, this.y - 8);
                ctx.lineTo(this.x + this.width - 4, this.y);
                ctx.fill();

                // Enemy Health Bar (visible if damaged or multi-HP)
                if (this.maxHp > 1) {
                    const barW = this.width;
                    const barH = 5;
                    const barY = this.y - 16;
                    ctx.fillStyle = '#1e293b';
                    ctx.fillRect(this.x, barY, barW, barH);
                    ctx.fillStyle = '#ef4444';
                    ctx.fillRect(this.x, barY, barW * (this.hp / this.maxHp), barH);
                }

                ctx.restore();
            }
        }

        // Collectible Coin Class
        class Coin {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.radius = 8;
                this.isCollected = false;
                this.bounceAngle = Math.random() * Math.PI;
            }

            update() {
                // Floating bounce animation
                this.bounceAngle += 0.08;
            }

            draw() {
                if (this.isCollected) return;

                const offsetY = Math.sin(this.bounceAngle) * 4;

                ctx.save();
                ctx.fillStyle = '#fbbf24';
                ctx.strokeStyle = '#d97706';
                ctx.lineWidth = 2;

                ctx.beginPath();
                ctx.arc(this.x, this.y + offsetY, this.radius, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                // Inner detail
                ctx.fillStyle = '#fef08a';
                ctx.beginPath();
                ctx.arc(this.x - 2, this.y + offsetY - 2, 3, 0, Math.PI * 2);
                ctx.fill();

                ctx.restore();
            }
        }

        // Hiding Barrier / Cover Object
        class Barrier {
            constructor(x, y, width, height) {
                this.x = x;
                this.y = y;
                this.width = width;
                this.height = height;
            }

            draw() {
                ctx.save();
                // Draw stone pillar / bush stealth barrier
                ctx.fillStyle = '#334155';
                ctx.fillRect(this.x, this.y, this.width, this.height);

                ctx.fillStyle = '#475569';
                ctx.fillRect(this.x + 4, this.y + 4, this.width - 8, this.height - 4);

                // Decorative vines / stealth pattern
                ctx.fillStyle = '#1e293b';
                ctx.fillRect(this.x + 8, this.y + 12, 10, 20);
                ctx.fillRect(this.x + 22, this.y + 25, 12, 15);

                ctx.restore();
            }
        }

        // End-of-Level Pot of Gold Class
        class PotOfGold {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = 44;
                this.height = 38;
                this.glowAngle = 0;
            }

            update() {
                this.glowAngle += 0.06;
            }

            draw(isUnlocked) {
                ctx.save();
                const centerX = this.x + this.width / 2;
                const centerY = this.y + this.height / 2;

                // Golden Aura when unlocked
                if (isUnlocked) {
                    const pulse = 28 + Math.sin(this.glowAngle) * 6;
                    const grad = ctx.createRadialGradient(centerX, centerY, 4, centerX, centerY, pulse);
                    grad.addColorStop(0, 'rgba(250, 204, 21, 0.75)');
                    grad.addColorStop(1, 'rgba(250, 204, 21, 0)');
                    ctx.fillStyle = grad;
                    ctx.beginPath();
                    ctx.arc(centerX, centerY, pulse, 0, Math.PI * 2);
                    ctx.fill();
                } else {
                    ctx.globalAlpha = 0.45;
                }

                // Mound of Gold Coins overflowing on top
                ctx.fillStyle = '#facc15';
                ctx.beginPath();
                ctx.arc(centerX, this.y + 10, 16, Math.PI, 0, false);
                ctx.fill();

                // Individual sparkling nuggets
                ctx.fillStyle = '#fef08a';
                ctx.beginPath();
                ctx.arc(centerX - 6, this.y + 4, 4, 0, Math.PI * 2);
                ctx.arc(centerX + 5, this.y + 3, 4.5, 0, Math.PI * 2);
                ctx.arc(centerX, this.y - 1, 4, 0, Math.PI * 2);
                ctx.fill();

                // Cauldron / Pot Body
                ctx.fillStyle = '#1e293b';
                ctx.strokeStyle = '#64748b';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.arc(centerX, this.y + 22, 18, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                // Pot Rim
                ctx.fillStyle = '#334155';
                ctx.beginPath();
                ctx.roundRect(this.x + 3, this.y + 8, this.width - 6, 7, 3);
                ctx.fill();
                ctx.stroke();

                // Lock or Ready Label
                ctx.globalAlpha = 1;
                ctx.font = 'bold 11px "Segoe UI", sans-serif';
                ctx.textAlign = 'center';
                if (isUnlocked) {
                    ctx.fillStyle = '#fef08a';
                    ctx.fillText('POTE DE OURO!', centerX, this.y - 10);
                } else {
                    ctx.fillStyle = '#94a3b8';
                    ctx.fillText('🔒 COLETE AS MOEDAS', centerX, this.y - 10);
                }

                ctx.restore();
            }
        }

        // Level Elements Global Holders
        let player;
        let platforms = [];
        let enemies = [];
        let coins = [];
        let barriers = [];
        let potOfGold = null;
                    new Enemy(250, 109, 245, 385, 2.4, 1)
                ];

                coins = [
                    new Coin(180, 285),
                    new Coin(230, 285),
                    new Coin(430, 210),
                    new Coin(600, 285),
                    new Coin(660, 285),
                    new Coin(690, 375)
                ];

                // Pote de Ouro no final da Fase 1 (lado direito)
                potOfGold = new PotOfGold(735, 372);
            } else if (levelNum === 2) {
                // FASE 2: Desfiladeiro Carmesim (Abismos + Plataformas elevadas)
                platforms = [
                    { x: 0, y: 410, width: 220, height: 40, color: '#451a1a' },      // Chão Esquerdo
                    { x: 310, y: 410, width: 190, height: 40, color: '#451a1a' },    // Ilha Central
                    { x: 590, y: 410, width: 210, height: 40, color: '#451a1a' },    // Chão Direito
                    { x: 130, y: 305, width: 130, height: 18, color: '#7f1d1d' },    // Plataforma Elevada 1
                    { x: 340, y: 235, width: 140, height: 18, color: '#7f1d1d' },    // Plataforma Elevada 2
                    { x: 540, y: 295, width: 140, height: 18, color: '#7f1d1d' },    // Plataforma Elevada 3
                    { x: 240, y: 145, width: 150, height: 18, color: '#991b1b' }     // Topo
                ];

                barriers = [
                    new Barrier(390, 360, 40, 50),
                    new Barrier(590, 245, 40, 50)
                ];

                enemies = [
                    new Enemy(320, 374, 315, 495, 2.2, 1),
                    new Enemy(550, 259, 545, 675, 2.3, 1),
                    new Enemy(250, 109, 245, 385, 2.4, 1)
                ];

                coins = [
                    new Coin(170, 270),
                    new Coin(220, 270),
                    new Coin(410, 200),
                    new Coin(280, 110),
                    new Coin(340, 110),
                    new Coin(650, 260),
                    new Coin(685, 375)
                ];

                // Pote de Ouro no final da Fase 2
                potOfGold = new PotOfGold(735, 372);
            } else if (levelNum === 3) {
                // FASE 3: Fortaleza Sombria (Plataformas móveis + Inimigos de 2 HP)
                platforms = [
                    { x: 0, y: 410, width: 260, height: 40, color: '#1e1b4b' },      // Chão Esquerdo
                    { x: 520, y: 410, width: 280, height: 40, color: '#1e1b4b' },    // Chão Direito
                    { x: 290, y: 350, width: 120, height: 18, color: '#4338ca', moveX: true, minX: 270, maxX: 400, speedX: 1.4 }, // Plataforma Móvel 1
                    { x: 110, y: 275, width: 140, height: 18, color: '#312e81' },    // Plataforma Fixa Esq
                    { x: 540, y: 275, width: 150, height: 18, color: '#312e81' },    // Plataforma Fixa Dir
                    { x: 310, y: 185, width: 150, height: 18, color: '#4338ca', moveX: true, minX: 250, maxX: 430, speedX: -1.6 }, // Plataforma Móvel 2
                    { x: 80, y: 125, width: 140, height: 18, color: '#3730a3' }      // Torre Superior
                ];

                barriers = [
                    new Barrier(160, 225, 40, 50),
                    new Barrier(610, 360, 40, 50),
                    new Barrier(590, 225, 40, 50)
                ];

                enemies = [
                    new Enemy(540, 368, 530, 770, 2.1, 2), // Guardião Sombrio 2 HP
                    new Enemy(120, 239, 115, 245, 2.2, 1),
                    new Enemy(550, 233, 545, 685, 2.3, 2), // Guardião Sombrio 2 HP
                    new Enemy(90, 89, 85, 215, 2.0, 1)
                ];

                coins = [
                    new Coin(140, 240),
                    new Coin(210, 240),
                    new Coin(115, 90),
                    new Coin(175, 90),
                    new Coin(380, 150),
                    new Coin(570, 240),
                    new Coin(650, 240),
                    new Coin(695, 375)
                ];

                // Pote de Ouro no final da Fase 3
                potOfGold = new PotOfGold(740, 372);
            }
        }

        // Handle Keyboard Events
        window.addEventListener('keydown', (e) => {
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') keys.left = true;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') keys.right = true;
            if (e.code === 'KeyW' || e.code === 'ArrowUp' || e.code === 'Space') keys.up = true;
            if (e.code === 'KeyS' || e.code === 'ArrowDown') keys.down = true;
            if (e.code === 'KeyJ' || e.code === 'KeyK' || e.code === 'KeyZ' || e.code === 'KeyX') keys.attack = true;
        });

        window.addEventListener('keyup', (e) => {
            if (e.code === 'KeyA' || e.code === 'ArrowLeft') keys.left = false;
            if (e.code === 'KeyD' || e.code === 'ArrowRight') keys.right = false;
            if (e.code === 'KeyW' || e.code === 'ArrowUp' || e.code === 'Space') keys.up = false;
            if (e.code === 'KeyS' || e.code === 'ArrowDown') keys.down = false;
            if (e.code === 'KeyJ' || e.code === 'KeyK' || e.code === 'KeyZ' || e.code === 'KeyX') keys.attack = false;
        });

        // Touch Control Bindings
        function setupTouchControls() {
            const bindTouch = (id, keyName) => {
                const el = document.getElementById(id);
                if (!el) return;
                el.addEventListener('touchstart', (e) => { e.preventDefault(); keys[keyName] = true; });
                el.addEventListener('touchend', (e) => { e.preventDefault(); keys[keyName] = false; });
                el.addEventListener('mousedown', () => { keys[keyName] = true; });
                el.addEventListener('mouseup', () => { keys[keyName] = false; });
            };

            bindTouch('btnLeft', 'left');
            bindTouch('btnRight', 'right');
            bindTouch('btnDown', 'down');
            bindTouch('btnJump', 'up');
            bindTouch('btnAttack', 'attack');
        }

        // Collision Utilities
        function checkRectCollision(rect1, rect2) {
            return (
                rect1.x < rect2.x + rect2.width &&
                rect1.x + rect1.width > rect2.x &&
                rect1.y < rect2.y + rect2.height &&
                rect1.y + rect1.height > rect2.y
            );
        }

        function checkCircleRectCollision(circle, rect) {
            let closestX = Math.max(rect.x, Math.min(circle.x, rect.x + rect.width));
            let closestY = Math.max(rect.y, Math.min(circle.y, rect.y + rect.height));

            let distanceX = circle.x - closestX;
            let distanceY = circle.y - closestY;

            return (distanceX * distanceX + distanceY * distanceY) < (circle.radius * circle.radius);
        }

        // Game Loop Update & Render
        function gameLoop() {
            if (gameState !== 'PLAYING') return;

            // Update Moving Platforms
            platforms.forEach(plat => {
                plat.prevX = plat.x;
                if (plat.moveX) {
                    plat.x += plat.speedX;
                    if (plat.x <= plat.minX || plat.x >= plat.maxX) {
                        plat.speedX *= -1;
                    }
                }
            });

            // 1. UPDATE
            player.update(platforms, barriers);

            // Update Coins & Collision
            let remainingCoins = 0;
            coins.forEach(coin => {
                coin.update();
                if (!coin.isCollected) {
                    remainingCoins++;
                    if (checkCircleRectCollision(coin, player)) {
                        coin.isCollected = true;
                        player.score += 100;
                        globalScore = player.score;
                    }
                }
            });

            // Update Enemies & Interactions
            let aliveEnemies = 0;
            enemies.forEach(enemy => {
                if (!enemy.isAlive) return;
                aliveEnemies++;

                enemy.update();

                // Player Attack Hits Enemy
                if (player.isAttacking && checkRectCollision(player.attackBox, enemy)) {
                    const defeated = enemy.takeDamage(1);
                    if (defeated) {
                        player.score += enemy.maxHp > 1 ? 450 : 250;
                        globalScore = player.score;
                    }
                }

                // Enemy Hits Player (Only if player is NOT hidden behind a barrier)
                if (enemy.isAlive && checkRectCollision(player, enemy)) {
                    if (!player.isHidden) {
                        player.takeDamage(1);
                    }
                }
            });

            // Check Level Progression / Victory Condition
            // Pot of Gold unlocks once all coins are collected (and enemies defeated on level 3)
            const potUnlocked = currentLevel === 3
                ? (remainingCoins === 0 && aliveEnemies === 0)
                : (remainingCoins === 0);

            if (potOfGold) {
                potOfGold.update();
                if (potUnlocked && checkRectCollision(player, potOfGold)) {
                    player.score += 500; // Bônus pelo Pote de Ouro!
                    globalScore = player.score;
                    if (currentLevel < maxLevels) {
                        advanceToNextLevel();
                        return;
                    } else {
                        triggerVictory();
                        return;
                    }
                }
            }

            // 2. RENDER
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Render Thematic Parallax Background
            drawBackground();

            // Render Platforms
            platforms.forEach(plat => {
                ctx.fillStyle = plat.color;
                ctx.fillRect(plat.x, plat.y, plat.width, plat.height);

                // Top border line highlight based on theme
                ctx.fillStyle = currentLevel === 1 ? '#475569' : (currentLevel === 2 ? '#ef4444' : '#6366f1');
                ctx.fillRect(plat.x, plat.y, plat.width, 4);
            });

            // Render Hiding Barriers
            barriers.forEach(barrier => barrier.draw());

            // Render Collectibles
            coins.forEach(coin => coin.draw());

            // Render Pot of Gold Goal
            if (potOfGold) {
                potOfGold.draw(potUnlocked);
            }

            // Render Enemies
            enemies.forEach(enemy => enemy.draw());

            // Render Player
            player.draw();

            // Render HUD / Interface
            drawHUD();

            // Next Frame loop call
            animationFrameId = requestAnimationFrame(gameLoop);
        }

        // Dynamic Parallax Background Renderer per Level
        function drawBackground() {
            const parallaxFar = player.x * 0.08;
            const parallaxMid = player.x * 0.2;

            ctx.save();

            if (currentLevel === 1) {
                // FASE 1: Céu Noturno, Lua Cheia, Estrelas e Ruínas da Floresta
                const skyGrad = ctx.createLinearGradient(0, 0, 0, canvas.height);
                skyGrad.addColorStop(0, '#020617');
                skyGrad.addColorStop(0.6, '#0f172a');
                skyGrad.addColorStop(1, '#1e293b');
                ctx.fillStyle = skyGrad;
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                // Twinkling Stars
                bgElements.stars.forEach(star => {
                    star.phase += star.blinkSpeed;
                    const alpha = 0.3 + Math.abs(Math.sin(star.phase)) * 0.7;
                    ctx.fillStyle = `rgba(248, 250, 252, ${alpha})`;
                    ctx.fillRect(star.x, star.y, star.size, star.size);
                });

                // Full Moon with Glow
                ctx.fillStyle = 'rgba(224, 242, 254, 0.12)';
                ctx.beginPath();
                ctx.arc(650 - parallaxFar * 0.3, 85, 52, 0, Math.PI * 2);
                ctx.fill();

                ctx.fillStyle = '#e0f2fe';
                ctx.beginPath();
                ctx.arc(650 - parallaxFar * 0.3, 85, 34, 0, Math.PI * 2);
                ctx.fill();

                // Distant Mountains (Far Parallax)
                ctx.fillStyle = '#162033';
                ctx.beginPath();
                ctx.moveTo(-50, 410);
                ctx.lineTo(120 - parallaxFar, 210);
                ctx.lineTo(310 - parallaxFar, 410);
                ctx.lineTo(480 - parallaxFar, 180);
                ctx.lineTo(680 - parallaxFar, 410);
                ctx.lineTo(860 - parallaxFar, 230);
                ctx.lineTo(950, 410);
                ctx.fill();

                // Ancient Ruins & Silhouette Trees (Mid Parallax)
                ctx.fillStyle = '#0f1a2c';
                for (let i = 0; i < 6; i++) {
                    const rx = i * 170 - parallaxMid;
                    // Castle Pillar Silhouette
                    ctx.fillRect(rx + 20, 250, 26, 160);
                    ctx.fillRect(rx + 14, 242, 38, 10);
                    // Pine Tree Silhouette
                    ctx.beginPath();
                    ctx.moveTo(rx + 95, 270);
                    ctx.lineTo(rx + 65, 410);
                    ctx.lineTo(rx + 125, 410);
                    ctx.fill();
                }

            } else if (currentLevel === 2) {
                // FASE 2: Desfiladeiro Carmesim (Pôr do sol avermelhado, vulcões e colunas antigas)
                const skyGrad = ctx.createLinearGradient(0, 0, 0, canvas.height);
                skyGrad.addColorStop(0, '#2a0808');
                skyGrad.addColorStop(0.55, '#7f1d1d');
                skyGrad.addColorStop(1, '#450a0a');
                ctx.fillStyle = skyGrad;
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                // Crimson Giant Sun
                ctx.fillStyle = 'rgba(251, 146, 60, 0.25)';
                ctx.beginPath();
                ctx.arc(400 - parallaxFar * 0.4, 150, 85, 0, Math.PI * 2);
                ctx.fill();

                ctx.fillStyle = '#f97316';
                ctx.beginPath();
                ctx.arc(400 - parallaxFar * 0.4, 150, 58, 0, Math.PI * 2);
                ctx.fill();

                // Distant Volcanoes (Far Parallax)
                ctx.fillStyle = '#3b0707';
                ctx.beginPath();
                ctx.moveTo(-40, 410);
                ctx.lineTo(150 - parallaxFar, 190);
                ctx.lineTo(200 - parallaxFar, 190);
                ctx.lineTo(380 - parallaxFar, 410);
                ctx.lineTo(560 - parallaxFar, 170);
                ctx.lineTo(620 - parallaxFar, 170);
                ctx.lineTo(850 - parallaxFar, 410);
                ctx.fill();

                // Lava Glow at bottom of abyss
                const lavaGrad = ctx.createLinearGradient(0, 415, 0, 450);
                lavaGrad.addColorStop(0, '#ea580c');
                lavaGrad.addColorStop(1, '#fef08a');
                ctx.fillStyle = lavaGrad;
                ctx.fillRect(0, 422, canvas.width, 28);

                // Broken Canyon Pillars (Mid Parallax)
                ctx.fillStyle = '#270505';
                for (let i = 0; i < 5; i++) {
                    const cx = i * 200 + 30 - parallaxMid;
                    ctx.fillRect(cx, 220 + (i % 2) * 40, 34, 200);
                    ctx.fillRect(cx - 6, 215 + (i % 2) * 40, 46, 12);
                }

            } else {
                // FASE 3: Fortaleza Sombria (Tempestade com relâmpagos e nuvens em movimento)
                const stormGrad = ctx.createLinearGradient(0, 0, 0, canvas.height);
                stormGrad.addColorStop(0, '#090514');
                stormGrad.addColorStop(0.6, '#1e1b4b');
                stormGrad.addColorStop(1, '#0f172a');
                ctx.fillStyle = stormGrad;
                ctx.fillRect(0, 0, canvas.width, canvas.height);

                // Occasional Lightning Flash
                bgElements.lightningTimer++;
                if (bgElements.lightningTimer > 220 && Math.random() < 0.03) {
                    bgElements.lightningAlpha = 0.65;
                    bgElements.lightningTimer = 0;
                }
                if (bgElements.lightningAlpha > 0) {
                    ctx.fillStyle = `rgba(224, 231, 255, ${bgElements.lightningAlpha})`;
                    ctx.fillRect(0, 0, canvas.width, canvas.height);
                    bgElements.lightningAlpha -= 0.05;
                }

                // Moving Dark Storm Clouds
                bgElements.clouds.forEach(cloud => {
                    cloud.x -= cloud.speed;
                    if (cloud.x + cloud.width < 0) cloud.x = canvas.width + 20;

                    ctx.fillStyle = 'rgba(30, 27, 75, 0.75)';
                    ctx.beginPath();
                    ctx.roundRect(cloud.x, cloud.y, cloud.width, cloud.height, 12);
                    ctx.fill();
                });

                // Dark Fortress Spires & Towers (Far/Mid Parallax)
                ctx.fillStyle = '#110e2e';
                for (let i = 0; i < 4; i++) {
                    const fx = i * 240 + 40 - parallaxMid;
                    // Main Tower
                    ctx.fillRect(fx, 150, 65, 260);
                    // Pointed Roof
                    ctx.beginPath();
                    ctx.moveTo(fx - 8, 150);
                    ctx.lineTo(fx + 32, 95);
                    ctx.lineTo(fx + 73, 150);
                    ctx.fill();
                    // Glowing Tower Window
                    ctx.fillStyle = '#818cf8';
                    ctx.fillRect(fx + 26, 175, 12, 22);
                    ctx.fillStyle = '#110e2e';
                }
            }

            ctx.restore();
        }

        // HUD Render function
        function drawHUD() {
            ctx.save();

            // Hearts for Lives
            for (let i = 0; i < 3; i++) {
                ctx.fillStyle = i < player.lives ? '#ef4444' : '#475569';
                ctx.beginPath();
                let hx = 25 + i * 28;
                let hy = 25;
                ctx.arc(hx - 5, hy - 4, 6, Math.PI, 0, false);
                ctx.arc(hx + 5, hy - 4, 6, Math.PI, 0, false);
                ctx.lineTo(hx, hy + 8);
                ctx.closePath();
                ctx.fill();
            }

            // Score Display
            ctx.fillStyle = '#f8fafc';
            ctx.font = 'bold 15px "Segoe UI", sans-serif';
            ctx.fillText(`PONTOS: ${player.score}`, 115, 26);

            // Current Level Indicator
            const levelNames = ['Floresta das Ruínas', 'Desfiladeiro Carmesim', 'Fortaleza Sombria'];
            ctx.fillStyle = '#a5b4fc';
            ctx.fillText(`FASE ${currentLevel}/${maxLevels}: ${levelNames[currentLevel - 1]}`, 245, 26);

            // Stealth Indicator Status
            if (player.isHidden) {
                ctx.fillStyle = '#38bdf8';
                ctx.font = 'bold 14px "Segoe UI", sans-serif';
                ctx.fillText('🛡️ ESCONDIDO', 525, 26);
            }

            // Remaining Coins Counter
            const countCoins = coins.filter(c => !c.isCollected).length;
            ctx.fillStyle = '#fbbf24';
            ctx.font = 'bold 15px "Segoe UI", sans-serif';
            ctx.fillText(`MOEDAS: ${coins.length - countCoins}/${coins.length}`, 665, 26);

            // Extra Objective Hint on Level 3
            if (currentLevel === 3) {
                const countEnemies = enemies.filter(e => e.isAlive).length;
                ctx.fillStyle = '#f43f5e';
                ctx.font = 'bold 13px "Segoe UI", sans-serif';
                ctx.fillText(`INIMIGOS RESTANTES: ${countEnemies}`, 635, 48);
            }

            ctx.restore();
        }

        // State Control Functions
        function startGame() {
            document.getElementById('startScreen').classList.add('hidden');
            document.getElementById('gameOverScreen').classList.add('hidden');
            document.getElementById('victoryScreen').classList.add('hidden');
            document.getElementById('levelTransitionScreen').classList.add('hidden');

            currentLevel = 1;
            globalScore = 0;
            initLevel(currentLevel);
            gameState = 'PLAYING';
            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            gameLoop();
        }

        function advanceToNextLevel() {
            gameState = 'TRANSITION';
            const nextLevel = currentLevel + 1;
            const levelNames = ['Floresta das Ruínas', 'Desfiladeiro Carmesim', 'Fortaleza Sombria'];

            document.getElementById('transitionTitle').innerText = `Fase ${currentLevel} Concluída! (+500 pts)`;
            document.getElementById('transitionSubtitle').innerText = `Você pegou o Pote de Ouro! Avançando para a Fase ${nextLevel}: ${levelNames[nextLevel - 1]}...`;
            document.getElementById('levelTransitionScreen').classList.remove('hidden');

            // Reset pressed keys so player doesn't keep running automatically
            keys.left = false;
            keys.right = false;
            keys.up = false;
            keys.down = false;
            keys.attack = false;

            setTimeout(() => {
                document.getElementById('levelTransitionScreen').classList.add('hidden');
                currentLevel = nextLevel;
                initLevel(currentLevel);
                gameState = 'PLAYING';
                gameLoop();
            }, 2200);
        }

        function restartGame() {
            startGame();
        }

        window.startGame = startGame;
        window.restartGame = restartGame;

        function triggerGameOver() {
            gameState = 'GAMEOVER';
            document.getElementById('gameOverScreen').classList.remove('hidden');
        }

        function triggerVictory() {
            gameState = 'VICTORY';
            document.getElementById('victoryMessage').innerText = `Você completou todas as 3 fases com a pontuação final de ${player.score}!`;
            document.getElementById('victoryScreen').classList.remove('hidden');
        }

        // Initialize Touch Controls on load
        setupTouchControls();
    </script>
</body>
</html>
