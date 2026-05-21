<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Brick Breaker</title>
    <style>
        :root {
            --primary: #0ff;
            --bg-color: #050510;
        }
        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: #fff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
            touch-action: none; /* Prevent scroll on touch */
        }
        #game-container {
            position: relative;
            box-shadow: 0 0 30px rgba(0, 255, 255, 0.15);
            background: #020208;
            border-radius: 8px;
            overflow: hidden;
        }
        canvas {
            display: block;
            /* Logical size is 800x600, CSS scales it down to fit screen while maintaining aspect ratio */
            max-width: 100vw;
            max-height: 100vh;
            width: 800px;
            height: 600px;
            object-fit: contain;
        }
        #hud {
            position: absolute;
            top: 15px;
            left: 25px;
            right: 25px;
            display: flex;
            justify-content: space-between;
            pointer-events: none;
            font-size: 1.5rem;
            font-weight: bold;
            text-shadow: 0 0 10px var(--primary);
            z-index: 10;
            user-select: none;
        }
        #hud > div {
            pointer-events: auto; /* Allow clicking the music toggle */
        }
        #music-toggle {
            cursor: pointer;
            transition: color 0.3s;
        }
        #music-toggle:hover {
            color: #fff;
        }
        .overlay {
            position: absolute;
            inset: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(5, 5, 16, 0.85);
            z-index: 20;
            text-align: center;
            backdrop-filter: blur(4px);
        }
        .hidden {
            display: none !important;
        }
        h1 {
            font-size: 3rem;
            margin-bottom: 10px;
            text-shadow: 0 0 20px var(--primary);
            text-transform: uppercase;
            letter-spacing: 2px;
        }
        p.subtitle {
            font-size: 1.2rem;
            color: #aaa;
            margin-bottom: 30px;
        }
        button {
            background: transparent;
            color: var(--primary);
            border: 2px solid var(--primary);
            padding: 15px 40px;
            font-size: 1.5rem;
            cursor: pointer;
            border-radius: 30px;
            font-weight: bold;
            text-transform: uppercase;
            transition: all 0.3s ease;
            box-shadow: 0 0 15px rgba(0, 255, 255, 0.3), inset 0 0 15px rgba(0, 255, 255, 0.1);
        }
        button:hover {
            background: var(--primary);
            color: #000;
            box-shadow: 0 0 30px rgba(0, 255, 255, 0.8);
            transform: scale(1.05);
        }
        .button-group {
            display: flex;
            gap: 20px;
            margin-top: 20px;
        }
        .score-list {
            list-style: none;
            padding: 0;
            margin: 0 0 30px 0;
            width: 300px;
        }
        .score-item {
            display: flex;
            justify-content: space-between;
            font-size: 1.5rem;
            margin-bottom: 10px;
            border-bottom: 1px solid rgba(0,255,255,0.2);
            padding-bottom: 5px;
            color: #fff;
        }
        .score-item span:first-child { color: var(--primary); }
        
        /* Customization Styles */
        .customization-panel {
            margin-top: 10px;
            background: rgba(0, 0, 0, 0.5);
            padding: 10px 20px;
            border-radius: 10px;
            border: 1px solid rgba(0, 255, 255, 0.2);
        }
        .color-picker {
            display: flex;
            gap: 10px;
            justify-content: center;
            margin-bottom: 10px;
        }
        .color-swatch {
            width: 25px;
            height: 25px;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid transparent;
            transition: transform 0.2s;
        }
        .color-swatch:hover { transform: scale(1.1); }
        .color-swatch.selected { border-color: #fff; box-shadow: 0 0 10px currentColor; }
        .player-label {
            font-size: 0.9rem;
            color: #ccc;
            margin-bottom: 5px;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <div id="hud">
            <div>Score: <span id="score">0</span></div>
            <div>Level: <span id="level">1</span></div>
            <div>Lives: <span id="lives">3</span></div>
            <div id="music-toggle" onclick="audioSys.toggleMusic()">🎵 ON</div>
        </div>
        <canvas id="gameCanvas" width="800" height="600"></canvas>
        
        <!-- Screens -->
        <div id="startScreen" class="overlay">
            <h1>Neon Breaker</h1>
            <p class="subtitle">Mouse, WASD, or Arrows to move</p>
            
            <div class="customization-panel">
                <div class="player-label">P1 Paddle Color</div>
                <div class="color-picker" id="p1-colors">
                    <div class="color-swatch selected" style="background-color: #0ff; color: #0ff;" onclick="selectColor('p1', '#0ff', this)"></div>
                    <div class="color-swatch" style="background-color: #f0f; color: #f0f;" onclick="selectColor('p1', '#f0f', this)"></div>
                    <div class="color-swatch" style="background-color: #0f0; color: #0f0;" onclick="selectColor('p1', '#0f0', this)"></div>
                    <div class="color-swatch" style="background-color: #ff0; color: #ff0;" onclick="selectColor('p1', '#ff0', this)"></div>
                    <div class="color-swatch" style="background-color: #fa0; color: #fa0;" onclick="selectColor('p1', '#fa0', this)"></div>
                </div>
                <div class="player-label">P2 Paddle Color</div>
                <div class="color-picker" id="p2-colors">
                    <div class="color-swatch" style="background-color: #0ff; color: #0ff;" onclick="selectColor('p2', '#0ff', this)"></div>
                    <div class="color-swatch selected" style="background-color: #f0f; color: #f0f;" onclick="selectColor('p2', '#f0f', this)"></div>
                    <div class="color-swatch" style="background-color: #0f0; color: #0f0;" onclick="selectColor('p2', '#0f0', this)"></div>
                    <div class="color-swatch" style="background-color: #ff0; color: #ff0;" onclick="selectColor('p2', '#ff0', this)"></div>
                    <div class="color-swatch" style="background-color: #fa0; color: #fa0;" onclick="selectColor('p2', '#fa0', this)"></div>
                </div>
                <div class="player-label">Ball Color</div>
                <div class="color-picker" id="ball-colors">
                    <div class="color-swatch selected" style="background-color: #0ff; color: #0ff;" onclick="selectColor('ball', '#0ff', this)"></div>
                    <div class="color-swatch" style="background-color: #f0f; color: #f0f;" onclick="selectColor('ball', '#f0f', this)"></div>
                    <div class="color-swatch" style="background-color: #0f0; color: #0f0;" onclick="selectColor('ball', '#0f0', this)"></div>
                    <div class="color-swatch" style="background-color: #ff0; color: #ff0;" onclick="selectColor('ball', '#ff0', this)"></div>
                    <div class="color-swatch" style="background-color: #fa0; color: #fa0;" onclick="selectColor('ball', '#fa0', this)"></div>
                    <div class="color-swatch" style="background-color: #fff; color: #fff;" onclick="selectColor('ball', '#fff', this)"></div>
                </div>
            </div>

            <div class="button-group">
                <button onclick="game.start(1)">1 Player</button>
                <button style="border-color: #f0f; color: #f0f; box-shadow: 0 0 15px rgba(255,0,255,0.3)" onclick="game.start(2)">2 Players</button>
            </div>
            <button style="margin-top: 20px; font-size: 1.2rem; padding: 10px 30px; border-color: #0f0; color: #0f0; box-shadow: 0 0 15px rgba(0,255,0,0.3)" onclick="game.showHighScores()">Top Scores</button>
        </div>
        
        <div id="gameOverScreen" class="overlay hidden">
            <h1 style="color: #f0f; text-shadow: 0 0 20px #f0f;">System Failure</h1>
            <p class="subtitle">Final Score: <span id="finalScore">0</span></p>
            <div class="button-group">
                <button style="border-color: #f0f; color: #f0f; box-shadow: 0 0 15px rgba(255,0,255,0.3)" onclick="game.reset()">Reboot</button>
                <button style="font-size: 1.2rem; padding: 10px 30px;" onclick="game.showMenu()">Menu</button>
            </div>
        </div>

        <div id="levelClearScreen" class="overlay hidden">
            <h1 style="color: #0f0; text-shadow: 0 0 20px #0f0;">Level Cleared</h1>
            <p class="subtitle">Get ready for the next wave...</p>
            <button style="border-color: #0f0; color: #0f0; box-shadow: 0 0 15px rgba(0,255,0,0.3)" onclick="game.nextLevel()">Continue</button>
        </div>
        
        <div id="victoryScreen" class="overlay hidden">
            <h1 style="color: #ff0; text-shadow: 0 0 20px #ff0;">You Win!</h1>
            <p class="subtitle">You have conquered the Neon Grid. Score: <span id="victoryScore">0</span></p>
            <div class="button-group">
                <button style="border-color: #ff0; color: #ff0; box-shadow: 0 0 15px rgba(255,255,0,0.3)" onclick="game.reset()">Play Again</button>
                <button style="font-size: 1.2rem; padding: 10px 30px;" onclick="game.showMenu()">Menu</button>
            </div>
        </div>

        <div id="highScoreScreen" class="overlay hidden">
            <h1 style="color: #0ff; text-shadow: 0 0 20px #0ff;">Global Top Scores</h1>
            <ol id="scoreList" class="score-list"></ol>
            <button onclick="game.showMenu()">Back to Menu</button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        let currentUser = null;

        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');

        // Utility: Random range
        const rand = (min, max) => Math.random() * (max - min) + min;

        // --- Game Config & Constants ---
        const GAME_WIDTH = 800;
        const GAME_HEIGHT = 600;
        
        // Colors mapping for brick toughness
        const BRICK_COLORS = [
            '#ff0055', // Level 1 hit (Pink-red)
            '#00ffcc', // Level 2 hits (Cyan)
            '#cc00ff', // Level 3 hits (Purple)
            '#ffff00'  // Level 4 hits (Yellow)
        ];

        // Global input tracking
        const keys = {};
        window.addEventListener('keydown', e => keys[e.code] = true);
        window.addEventListener('keyup', e => keys[e.code] = false);

        // Paddle Customization State
        let selectedP1Color = '#0ff';
        let selectedP2Color = '#f0f';
        let selectedBallColor = '#0ff';

        function selectColor(target, color, element) {
            if (target === 'p1') {
                selectedP1Color = color;
                document.querySelectorAll('#p1-colors .color-swatch').forEach(el => el.classList.remove('selected'));
            } else if (target === 'p2') {
                selectedP2Color = color;
                document.querySelectorAll('#p2-colors .color-swatch').forEach(el => el.classList.remove('selected'));
            } else if (target === 'ball') {
                selectedBallColor = color;
                document.querySelectorAll('#ball-colors .color-swatch').forEach(el => el.classList.remove('selected'));
            }
            element.classList.add('selected');
        }
        window.selectColor = selectColor;

        // --- Audio System ---
        class AudioSystem {
            constructor() {
                this.ctx = null;
                this.musicPlaying = false;
                this.muted = false;
                this.nextNoteTime = 0;
                this.currentNote = 0;
                this.tempo = 130; // 130 BPM for synthwave feel
                this.sequence = [
                    36, 36, 48, 36, 36, 36, 46, 36,
                    36, 36, 48, 36, 36, 43, 46, 43
                ]; // MIDI note numbers for a driving bassline
                this.timerID = null;
            }

            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            toggleMusic() {
                this.muted = !this.muted;
                const btn = document.getElementById('music-toggle');
                if (this.muted) {
                    this.stopMusic();
                    btn.innerText = '🎵 OFF';
                    btn.style.color = '#888';
                    btn.style.textShadow = 'none';
                } else {
                    this.startMusic();
                    btn.innerText = '🎵 ON';
                    btn.style.color = 'var(--primary)';
                    btn.style.textShadow = '0 0 10px var(--primary)';
                }
            }

            playTone(freq, type, duration, vol=0.1) {
                if (this.muted || !this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = type;
                osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
                
                gain.gain.setValueAtTime(vol, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + duration);
                
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                
                osc.start();
                osc.stop(this.ctx.currentTime + duration);
            }

            playHit() { this.playTone(600, 'square', 0.1, 0.05); }
            playPaddle() { this.playTone(400, 'sine', 0.1, 0.05); }
            playPowerup() { 
                this.playTone(800, 'triangle', 0.1, 0.1); 
                setTimeout(() => this.playTone(1200, 'triangle', 0.2, 0.1), 100); 
            }
            playDeath() { this.playTone(150, 'sawtooth', 0.5, 0.1); }
            
            playExplosion() {
                if (this.muted || !this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(100, this.ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(10, this.ctx.currentTime + 0.3);
                
                gain.gain.setValueAtTime(0.3, this.ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.3);
                
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                
                osc.start();
                osc.stop(this.ctx.currentTime + 0.3);
            }

            midiToFreq(midi) { return 440 * Math.pow(2, (midi - 69) / 12); }

            scheduleNote(noteNumber, time) {
                if (this.muted || !this.ctx) return;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                
                osc.type = 'sawtooth';
                osc.frequency.value = this.midiToFreq(this.sequence[noteNumber]);
                
                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(100, time);
                filter.frequency.exponentialRampToValueAtTime(1200, time + 0.05);
                filter.frequency.exponentialRampToValueAtTime(100, time + 0.2);

                gain.gain.setValueAtTime(0.0, time);
                gain.gain.linearRampToValueAtTime(0.05, time + 0.02);
                gain.gain.exponentialRampToValueAtTime(0.001, time + 0.2);

                osc.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);

                osc.start(time);
                osc.stop(time + 0.2);
            }

            scheduler() {
                if (!this.ctx) return;
                while (this.nextNoteTime < this.ctx.currentTime + 0.1) {
                    this.scheduleNote(this.currentNote, this.nextNoteTime);
                    const secondsPerBeat = 60.0 / this.tempo;
                    this.nextNoteTime += 0.25 * secondsPerBeat; // 16th notes
                    this.currentNote = (this.currentNote + 1) % this.sequence.length;
                }
                this.timerID = setTimeout(() => this.scheduler(), 25);
            }

            startMusic() {
                if (this.musicPlaying || this.muted) return;
                this.init();
                this.musicPlaying = true;
                this.nextNoteTime = this.ctx.currentTime + 0.05;
                this.scheduler();
            }

            stopMusic() {
                this.musicPlaying = false;
                clearTimeout(this.timerID);
            }
        }
        const audioSys = new AudioSystem();
        window.audioSys = audioSys;

        // --- Classes ---

        class Paddle {
            constructor(playerNum = 1, numPlayers = 1) {
                this.width = 100;
                this.height = 15;
                this.playerNum = playerNum;
                this.numPlayers = numPlayers;
                
                // Position based on player number
                if (numPlayers === 1) {
                    this.x = GAME_WIDTH / 2 - this.width / 2;
                } else {
                    this.x = playerNum === 1 ? GAME_WIDTH / 3 - this.width / 2 : (GAME_WIDTH * 2) / 3 - this.width / 2;
                }
                
                this.y = GAME_HEIGHT - 40;
                
                // Colors: Use selected customization colors
                this.baseColor = playerNum === 1 ? selectedP1Color : selectedP2Color;
                this.expandColor = '#fff'; // White when expanded to show power-up state clearly
                this.color = this.baseColor;
                
                this.targetX = this.x;
                this.speed = 600; // pixels per second for keyboard movement
                
                // Power-up effects
                this.expandedTimer = 0;
            }

            update(dt) {
                // Handle power-up timers
                if (this.expandedTimer > 0) {
                    this.expandedTimer -= dt;
                    this.width = 160;
                    this.color = this.expandColor;
                } else {
                    this.width = 100;
                    this.color = this.baseColor;
                }

                let movedWithKey = false;
                const moveAmount = this.speed * (dt / 1000);

                // Keyboard Controls
                if (this.numPlayers === 1) {
                    // 1 Player: WASD or Arrows
                    if (keys['KeyA'] || keys['ArrowLeft']) { this.x -= moveAmount; movedWithKey = true; }
                    if (keys['KeyD'] || keys['ArrowRight']) { this.x += moveAmount; movedWithKey = true; }
                } else {
                    // 2 Players: P1 on WASD, P2 on Arrows
                    if (this.playerNum === 1) {
                        if (keys['KeyA']) { this.x -= moveAmount; movedWithKey = true; }
                        if (keys['KeyD']) { this.x += moveAmount; movedWithKey = true; }
                    } else if (this.playerNum === 2) {
                        if (keys['ArrowLeft']) { this.x -= moveAmount; movedWithKey = true; }
                        if (keys['ArrowRight']) { this.x += moveAmount; movedWithKey = true; }
                    }
                }

                // Smooth movement towards mouse/touch target ONLY if no keyboard input and it's player 1 (or solo)
                if (!movedWithKey && this.playerNum === 1) {
                    this.x += (this.targetX - this.width / 2 - this.x) * 0.3;
                }
                if (movedWithKey && this.playerNum === 1) {
                    this.targetX = this.x + this.width / 2; // Sync mouse target with keyboard position
                }
                
                // Clamp to screen
                if (this.x < 0) this.x = 0;
                if (this.x + this.width > GAME_WIDTH) this.x = GAME_WIDTH - this.width;
            }

            draw(ctx) {
                ctx.shadowBlur = 15;
                ctx.shadowColor = this.color;
                ctx.fillStyle = this.color;
                // Rounded rectangle paddle
                ctx.beginPath();
                ctx.roundRect(this.x, this.y, this.width, this.height, 8);
                ctx.fill();
                
                // Inner highlight for depth
                ctx.shadowBlur = 0;
                ctx.fillStyle = 'rgba(255,255,255,0.4)';
                ctx.beginPath();
                ctx.roundRect(this.x + 2, this.y + 2, this.width - 4, this.height / 2, 4);
                ctx.fill();
            }
        }

        class Ball {
            constructor(x, y, isPowerUp = false) {
                this.radius = 8;
                this.x = x;
                this.y = y;
                this.speed = isPowerUp ? 6 : 5;
                this.dx = isPowerUp ? rand(-this.speed, this.speed) : this.speed;
                this.dy = -this.speed;
                this.color = isPowerUp ? '#fff' : selectedBallColor;
                this.active = true;
            }

            update() {
                this.x += this.dx;
                this.y += this.dy;

                // Wall Collisions
                if (this.x - this.radius < 0) {
                    this.x = this.radius;
                    this.dx = -this.dx;
                }
                if (this.x + this.radius > GAME_WIDTH) {
                    this.x = GAME_WIDTH - this.radius;
                    this.dx = -this.dx;
                }
                if (this.y - this.radius < 0) {
                    this.y = this.radius;
                    this.dy = -this.dy;
                }

                // Bottom check
                if (this.y + this.radius > GAME_HEIGHT) {
                    this.active = false;
                }
            }

            draw(ctx) {
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fill();
                
                // Inner white highlight for extra neon depth
                ctx.shadowBlur = 0;
                ctx.fillStyle = 'rgba(255,255,255,0.6)';
                ctx.beginPath();
                ctx.arc(this.x - 2, this.y - 2, this.radius / 2.5, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        class Brick {
            constructor(x, y, width, height, strength, isExplosive = false) {
                this.x = x;
                this.y = y;
                this.width = width;
                this.height = height;
                this.strength = strength;
                this.active = true;
                this.isExplosive = isExplosive;
                this.pulsePhase = Math.random() * Math.PI * 2;
            }

            hit() {
                this.strength--;
                if (this.strength <= 0) {
                    this.active = false;
                    return true; // Destroyed
                }
                return false; // Still alive
            }

            draw(ctx) {
                if (!this.active) return;
                
                if (this.isExplosive) {
                    this.pulsePhase += 0.1;
                    const glow = 10 + Math.sin(this.pulsePhase) * 5;
                    ctx.shadowBlur = glow;
                    ctx.shadowColor = '#ff2200';
                    ctx.fillStyle = '#ff2200';
                    ctx.fillRect(this.x, this.y, this.width, this.height);
                    
                    ctx.shadowBlur = 0;
                    ctx.fillStyle = '#fff';
                    ctx.font = 'bold 16px Arial';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText('!', this.x + this.width / 2, this.y + this.height / 2);
                    
                    ctx.strokeStyle = 'rgba(255,255,255,0.4)';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(this.x, this.y, this.width, this.height);
                } else {
                    const color = BRICK_COLORS[(this.strength - 1) % BRICK_COLORS.length];
                    ctx.shadowBlur = 10;
                    ctx.shadowColor = color;
                    ctx.fillStyle = color;
                    ctx.fillRect(this.x, this.y, this.width, this.height);
                    
                    // Border/Highlight
                    ctx.shadowBlur = 0;
                    ctx.strokeStyle = 'rgba(255,255,255,0.3)';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(this.x, this.y, this.width, this.height);
                }
            }
        }

        class Particle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                const angle = rand(0, Math.PI * 2);
                const speed = rand(1, 6);
                this.dx = Math.cos(angle) * speed;
                this.dy = Math.sin(angle) * speed;
                this.size = rand(2, 6);
                this.life = 1.0; // Alpha
                this.decay = rand(0.02, 0.05);
                this.color = color;
            }
            update() {
                this.x += this.dx;
                this.y += this.dy;
                this.life -= this.decay;
            }
            draw(ctx) {
                ctx.globalAlpha = Math.max(0, this.life);
                ctx.fillStyle = this.color;
                ctx.shadowBlur = 5;
                ctx.shadowColor = this.color;
                ctx.fillRect(this.x, this.y, this.size, this.size);
                ctx.globalAlpha = 1.0;
                ctx.shadowBlur = 0;
            }
        }

        class Shockwave {
            constructor(x, y, maxRadius, color) {
                this.x = x;
                this.y = y;
                this.radius = 5;
                this.maxRadius = maxRadius;
                this.color = color;
                this.alpha = 1.0;
            }
            update() {
                // Expand outward rapidly
                this.radius += 10; 
                // Fade out smoothly
                this.alpha -= 0.04; 
            }
            draw(ctx) {
                if (this.alpha <= 0) return;
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.alpha);
                
                // Draw expanding ring
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.strokeStyle = this.color;
                ctx.lineWidth = 5 * this.alpha; // Thins out as it fades
                ctx.shadowBlur = 20;
                ctx.shadowColor = this.color;
                ctx.stroke();

                // Inner intense white flash that fades twice as fast
                if (this.alpha > 0.5) {
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius * 0.7, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(255, 255, 255, ${(this.alpha - 0.5) * 2})`;
                    ctx.fill();
                }
                
                ctx.restore();
            }
        }

        class PowerUp {
            constructor(x, y, type) {
                this.x = x;
                this.y = y;
                this.type = type; // 'M' = Multiball, 'E' = Expand, 'L' = Life
                this.radius = 10;
                this.dy = 2;
                this.active = true;
                
                switch(type) {
                    case 'M': this.color = '#0088ff'; this.text = 'M'; break;
                    case 'E': this.color = '#00ff00'; this.text = 'E'; break;
                    case 'L': this.color = '#ff00ff'; this.text = '♥'; break;
                }
            }
            update() {
                this.y += this.dy;
                if (this.y > GAME_HEIGHT) this.active = false;
            }
            draw(ctx) {
                ctx.shadowBlur = 10;
                ctx.shadowColor = this.color;
                ctx.fillStyle = this.color;
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fill();
                
                ctx.shadowBlur = 0;
                ctx.fillStyle = '#fff';
                ctx.font = '12px Arial';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(this.text, this.x, this.y);
            }
        }

        // --- Main Game Class ---
        class Game {
            constructor() {
                this.state = 'MENU'; // MENU, PLAYING, GAMEOVER, LEVEL_CLEAR, VICTORY
                this.score = 0;
                this.lives = 3;
                this.levelNum = 1;
                this.maxLevels = 5;
                this.numPlayers = 1;
                this.highScores = [];
                
                this.paddles = [];
                this.balls = [];
                this.bricks = [];
                this.particles = [];
                this.shockwaves = [];
                this.powerUps = [];
                
                this.lastTime = 0;
                this.shakeAmount = 0;
                
                // Background elements
                this.gridOffset = 0;
                this.bgStars = Array.from({length: 60}, () => ({
                    x: rand(0, GAME_WIDTH),
                    y: rand(0, GAME_HEIGHT),
                    size: rand(1, 2.5),
                    speed: rand(0.2, 1.0),
                    color: ['#0ff', '#f0f', '#0f0', '#ff0', '#fff'][Math.floor(rand(0, 5))],
                    alpha: rand(0.1, 0.4)
                }));
                
                // UI Elements
                this.uiScore = document.getElementById('score');
                this.uiLives = document.getElementById('lives');
                this.uiLevel = document.getElementById('level');
                
                this.bindEvents();
            }

            bindEvents() {
                // Mouse and touch tracking (primarily for Player 1)
                const updateTarget = (clientX) => {
                    // Get position relative to canvas logical size
                    const rect = canvas.getBoundingClientRect();
                    const scaleX = canvas.width / rect.width;
                    if (this.paddles.length > 0) {
                        this.paddles[0].targetX = (clientX - rect.left) * scaleX;
                    }
                };

                window.addEventListener('mousemove', (e) => {
                    if (this.state === 'PLAYING') updateTarget(e.clientX);
                });

                window.addEventListener('touchmove', (e) => {
                    if (this.state === 'PLAYING') {
                        e.preventDefault();
                        updateTarget(e.touches[0].clientX);
                    }
                }, { passive: false });
                
                // Launch ball on click or touch
                canvas.addEventListener('mousedown', () => this.launchInitialBall());
                canvas.addEventListener('touchstart', (e) => {
                    if (this.state === 'PLAYING') {
                        e.preventDefault();
                        this.launchInitialBall();
                        updateTarget(e.touches[0].clientX);
                    }
                }, { passive: false });

                // Launch ball on keyboard up/w/space
                window.addEventListener('keydown', (e) => {
                    if (this.state === 'PLAYING' && (e.code === 'Space' || e.code === 'KeyW' || e.code === 'ArrowUp')) {
                        this.launchInitialBall();
                    }
                });
            }

            showMenu() {
                this.state = 'MENU';
                document.querySelectorAll('.overlay').forEach(el => el.classList.add('hidden'));
                document.getElementById('startScreen').classList.remove('hidden');
            }

            showHighScores() {
                document.querySelectorAll('.overlay').forEach(el => el.classList.add('hidden'));
                const list = document.getElementById('scoreList');
                list.innerHTML = '';
                if (this.highScores.length === 0) {
                    list.innerHTML = '<li class="score-item" style="justify-content:center">No scores yet!</li>';
                } else {
                    this.highScores.forEach((score, index) => {
                        list.innerHTML += `<li class="score-item"><span>Rank ${index + 1}</span><span>${score}</span></li>`;
                    });
                }
                document.getElementById('highScoreScreen').classList.remove('hidden');
            }

            recordScore() {
                if (this.score > 0) {
                    if (currentUser) {
                        // Push score to the cloud database
                        const scoresRef = collection(db, 'artifacts', appId, 'public', 'data', 'highscores');
                        addDoc(scoresRef, {
                            score: this.score,
                            userId: currentUser.uid,
                            timestamp: Date.now()
                        }).catch(e => console.error("Error saving score", e));
                    } else {
                        // Fallback to local
                        this.highScores.push(this.score);
                        this.highScores.sort((a, b) => b - a);
                        this.highScores = this.highScores.slice(0, 5); 
                    }
                }
            }

            start(players = 1) {
                this.numPlayers = players;
                audioSys.init();
                audioSys.startMusic();
                document.getElementById('startScreen').classList.add('hidden');
                this.reset();
            }

            reset() {
                this.score = 0;
                this.lives = 3;
                this.levelNum = 1;
                this.updateUI();
                this.initLevel();
                audioSys.startMusic();
                this.state = 'PLAYING';
                document.getElementById('gameOverScreen').classList.add('hidden');
                document.getElementById('victoryScreen').classList.add('hidden');
            }

            nextLevel() {
                this.levelNum++;
                if (this.levelNum > this.maxLevels) {
                    this.state = 'VICTORY';
                    this.recordScore();
                    audioSys.stopMusic();
                    document.getElementById('victoryScore').innerText = this.score;
                    document.getElementById('victoryScreen').classList.remove('hidden');
                } else {
                    this.initLevel();
                    audioSys.startMusic();
                    this.state = 'PLAYING';
                    document.getElementById('levelClearScreen').classList.add('hidden');
                }
                this.updateUI();
            }

            initLevel() {
                this.paddles = [];
                this.paddles.push(new Paddle(1, this.numPlayers));
                if (this.numPlayers === 2) {
                    this.paddles.push(new Paddle(2, this.numPlayers));
                }
                
                this.balls = [];
                this.particles = [];
                this.shockwaves = [];
                this.powerUps = [];
                this.spawnBall(true); // Spawn initial ball attached to paddle
                this.createBricks();
            }

            spawnBall(stuckToPaddle = false) {
                // Default to first paddle
                const p = this.paddles[0];
                const b = new Ball(p.x + p.width / 2, p.y - 10);
                if (stuckToPaddle) {
                    b.dx = 0;
                    b.dy = 0;
                    b.stuck = true;
                    b.stuckTo = p; // Reference which paddle it's stuck to
                }
                this.balls.push(b);
            }

            launchInitialBall() {
                this.balls.forEach(b => {
                    if (b.stuck) {
                        b.stuck = false;
                        b.dx = rand(-4, 4);
                        b.dy = -b.speed;
                    }
                });
            }

            createBricks() {
                this.bricks = [];
                // Dimensions and padding
                const rowCount = 4 + this.levelNum; // Increases per level
                const colCount = 10;
                const padding = 10;
                const offsetTop = 60;
                const offsetLeft = 35;
                const w = (GAME_WIDTH - offsetLeft * 2 - padding * (colCount - 1)) / colCount;
                const h = 20;

                for (let r = 0; r < rowCount; r++) {
                    for (let c = 0; c < colCount; c++) {
                        // Pattern logic: Skip some bricks for cool shapes
                        if (r % 2 === 0 && c % 2 !== 0 && this.levelNum > 2) continue;
                        
                        let brickX = offsetLeft + c * (w + padding);
                        let brickY = offsetTop + r * (h + padding);
                        
                        // Strength based on row and level
                        let strength = Math.ceil((rowCount - r) / 2) + Math.floor(this.levelNum / 2);
                        if (strength > 4) strength = 4; // Cap strength at 4
                        
                        let isExplosive = Math.random() < 0.12; // 12% chance for a brick to be explosive
                        this.bricks.push(new Brick(brickX, brickY, w, h, strength, isExplosive));
                    }
                }
            }

            spawnParticles(x, y, color, amount = 15) {
                for (let i = 0; i < amount; i++) {
                    this.particles.push(new Particle(x, y, color));
                }
            }

            addScreenShake(amount) {
                this.shakeAmount = amount;
            }

            checkCollisions() {
                // Ball vs Paddles
                this.balls.forEach(ball => {
                    if (ball.stuck) {
                        ball.x = ball.stuckTo.x + ball.stuckTo.width / 2;
                        return;
                    }

                    // Check collision with all paddles
                    this.paddles.forEach(paddle => {
                        // AABB Collision Paddle
                        if (ball.x + ball.radius > paddle.x &&
                            ball.x - ball.radius < paddle.x + paddle.width &&
                            ball.y + ball.radius > paddle.y &&
                            ball.y - ball.radius < paddle.y + paddle.height) {
                            
                            // Prevent multi-bounce getting stuck
                            if (ball.dy > 0) {
                                ball.dy = -ball.dy;
                                audioSys.playPaddle();
                                
                                // Adjust angle based on hit location
                                const hitPoint = (ball.x - (paddle.x + paddle.width / 2));
                                // Normalize hitPoint from -1 to 1
                                const normalized = hitPoint / (paddle.width / 2);
                                
                                ball.dx = normalized * ball.speed;
                                
                                // Prevent perfectly vertical or horizontal bouncing
                                const speedMod = Math.sqrt(ball.dx*ball.dx + ball.dy*ball.dy);
                                ball.dx = (ball.dx / speedMod) * ball.speed;
                                ball.dy = (ball.dy / speedMod) * ball.speed;
                                // Ensure minimum upward velocity
                                if(ball.dy > -2) ball.dy = -2;
                                
                                // Move ball above paddle to prevent getting stuck
                                ball.y = paddle.y - ball.radius - 1;
                            }
                        }
                    });

                    // Ball vs Bricks
                    this.bricks.forEach(brick => {
                        if (!brick.active) return;
                        
                        // Quick AABB check
                        if (ball.x + ball.radius > brick.x && ball.x - ball.radius < brick.x + brick.width &&
                            ball.y + ball.radius > brick.y && ball.y - ball.radius < brick.y + brick.height) {
                            
                            // Determine collision side
                            let overlapLeft = (ball.x + ball.radius) - brick.x;
                            let overlapRight = (brick.x + brick.width) - (ball.x - ball.radius);
                            let overlapTop = (ball.y + ball.radius) - brick.y;
                            let overlapBottom = (brick.y + brick.height) - (ball.y - ball.radius);
                            
                            let minOverlap = Math.min(overlapLeft, overlapRight, overlapTop, overlapBottom);

                            if (minOverlap === overlapLeft || minOverlap === overlapRight) {
                                ball.dx = -ball.dx;
                            } else {
                                ball.dy = -ball.dy;
                            }

                            // Handle Hit
                            const destroyed = brick.hit();
                            audioSys.playHit();
                            this.score += 10;
                            this.updateUI();
                            
                            const color = brick.isExplosive ? '#ff2200' : BRICK_COLORS[(brick.strength) % BRICK_COLORS.length]; // Color before hit
                            this.spawnParticles(ball.x, ball.y, color);
                            
                            // Slight shake for normal hit, big shake for destroy
                            if (destroyed) {
                                this.addScreenShake(3);
                                this.score += 40;
                                
                                if (brick.isExplosive) {
                                    this.triggerExplosion(brick.x + brick.width / 2, brick.y + brick.height / 2);
                                }
                                
                                // Power-up Drop Chance (15%)
                                if (Math.random() < 0.15) {
                                    const types = ['M', 'M', 'E', 'E', 'L']; // Weighted array
                                    const type = types[Math.floor(Math.random() * types.length)];
                                    this.powerUps.push(new PowerUp(brick.x + brick.width/2, brick.y, type));
                                }
                            } else {
                                this.addScreenShake(1);
                            }
                        }
                    });
                });

                // PowerUp vs Paddles
                this.powerUps.forEach(pu => {
                    if (!pu.active) return;
                    
                    this.paddles.forEach(paddle => {
                        if (!pu.active) return; // Skip if already caught by another paddle
                        
                        if (pu.y + pu.radius > paddle.y && pu.y - pu.radius < paddle.y + paddle.height &&
                            pu.x + pu.radius > paddle.x && pu.x - pu.radius < paddle.x + paddle.width) {
                            
                            pu.active = false;
                            audioSys.playPowerup();
                            this.score += 50;
                            this.updateUI();
                            
                            // Apply effect
                            if (pu.type === 'M') {
                                // Multiball: Spawn 2 new balls from the paddle that caught it
                                this.balls.push(new Ball(paddle.x + paddle.width/2, paddle.y - 10, true));
                                this.balls.push(new Ball(paddle.x + paddle.width/2, paddle.y - 10, true));
                            } else if (pu.type === 'E') {
                                paddle.expandedTimer = 10000; // 10 seconds for the catching paddle
                            } else if (pu.type === 'L') {
                                this.lives++;
                                this.updateUI();
                            }
                        }
                    });
                });
            }

            triggerExplosion(cx, cy) {
                audioSys.playExplosion();
                this.shakeAmount = Math.max(this.shakeAmount, 15);
                this.spawnParticles(cx, cy, '#ff2200', 30);
                
                const radius = 100; // Blast radius
                
                // Add shockwave animation
                this.shockwaves.push(new Shockwave(cx, cy, radius, '#ff2200'));

                const queue = [];
                
                this.bricks.forEach(otherBrick => {
                    if (!otherBrick.active) return;
                    
                    const bx = otherBrick.x + otherBrick.width / 2;
                    const by = otherBrick.y + otherBrick.height / 2;
                    const dist = Math.hypot(bx - cx, by - cy);
                    
                    if (dist <= radius) {
                        otherBrick.active = false;
                        this.score += 50;
                        const pColor = otherBrick.isExplosive ? '#ff2200' : BRICK_COLORS[(otherBrick.strength - 1) % BRICK_COLORS.length];
                        this.spawnParticles(bx, by, pColor, 10);
                        
                        if (otherBrick.isExplosive) {
                            queue.push({x: bx, y: by});
                        }
                    }
                });
                
                this.updateUI();
                
                // Process chain reactions
                queue.forEach(exp => this.triggerExplosion(exp.x, exp.y));
            }

            update(dt) {
                // Update Background animations (runs in all states)
                this.gridOffset = (this.gridOffset + 30 * (dt / 1000)) % 40;
                this.bgStars.forEach(star => {
                    star.y += star.speed * (dt / 16);
                    if (star.y > GAME_HEIGHT) {
                        star.y = -star.size;
                        star.x = rand(0, GAME_WIDTH);
                    }
                });

                if (this.state !== 'PLAYING') return;

                this.paddles.forEach(p => p.update(dt));
                
                // Update balls
                this.balls.forEach(b => b.update());
                this.balls = this.balls.filter(b => b.active);

                // Check death condition
                if (this.balls.length === 0) {
                    this.lives--;
                    this.updateUI();
                    this.addScreenShake(10);
                    audioSys.playDeath();
                    if (this.lives <= 0) {
                        this.state = 'GAMEOVER';
                        this.recordScore();
                        audioSys.stopMusic();
                        document.getElementById('finalScore').innerText = this.score;
                        document.getElementById('gameOverScreen').classList.remove('hidden');
                    } else {
                        // Reset powerups and paddles
                        this.powerUps = [];
                        this.paddles.forEach(p => p.expandedTimer = 0);
                        this.spawnBall(true);
                    }
                }

                // Check win condition
                let activeBricks = this.bricks.filter(b => b.active).length;
                if (activeBricks === 0 && this.bricks.length > 0) { // Check length > 0 to ensure level was initialized
                    this.state = 'LEVEL_CLEAR';
                    audioSys.stopMusic();
                    document.getElementById('levelClearScreen').classList.remove('hidden');
                }

                this.checkCollisions();

                // Update particles, shockwaves, and powerups
                this.particles.forEach(p => p.update());
                this.particles = this.particles.filter(p => p.life > 0);
                
                this.shockwaves.forEach(sw => sw.update());
                this.shockwaves = this.shockwaves.filter(sw => sw.alpha > 0);
                
                this.powerUps.forEach(pu => pu.update());
                this.powerUps = this.powerUps.filter(pu => pu.active);
            }

            draw() {
                // Clear background with slight fade for motion blur
                ctx.fillStyle = 'rgba(5, 5, 16, 0.3)';
                ctx.fillRect(0, 0, GAME_WIDTH, GAME_HEIGHT);

                // Draw Background Stars
                ctx.save();
                this.bgStars.forEach(star => {
                    ctx.globalAlpha = star.alpha;
                    ctx.fillStyle = star.color;
                    ctx.beginPath();
                    ctx.arc(star.x, star.y, star.size, 0, Math.PI * 2);
                    ctx.fill();
                });
                ctx.restore();

                // Handle Screen Shake
                ctx.save();
                if (this.shakeAmount > 0) {
                    const dx = rand(-this.shakeAmount, this.shakeAmount);
                    const dy = rand(-this.shakeAmount, this.shakeAmount);
                    ctx.translate(dx, dy);
                    this.shakeAmount *= 0.9; // Decay
                    if (this.shakeAmount < 0.5) this.shakeAmount = 0;
                }

                // Draw Grid Background (subtle & scrolling)
                ctx.strokeStyle = 'rgba(0, 255, 255, 0.03)';
                ctx.lineWidth = 1;
                for (let i = 0; i < GAME_WIDTH; i += 40) {
                    ctx.beginPath(); ctx.moveTo(i, 0); ctx.lineTo(i, GAME_HEIGHT); ctx.stroke();
                }
                for (let i = -40; i < GAME_HEIGHT; i += 40) {
                    let y = i + this.gridOffset;
                    ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(GAME_WIDTH, y); ctx.stroke();
                }

                // Draw Entities
                this.bricks.forEach(b => b.draw(ctx));
                this.shockwaves.forEach(sw => sw.draw(ctx));
                this.powerUps.forEach(pu => pu.draw(ctx));
                this.paddles.forEach(p => p.draw(ctx));
                this.particles.forEach(p => p.draw(ctx));
                this.balls.forEach(b => b.draw(ctx));

                ctx.restore();
            }

            updateUI() {
                this.uiScore.innerText = this.score;
                this.uiLives.innerText = this.lives;
                this.uiLevel.innerText = this.levelNum;
            }

            loop(timestamp) {
                let dt = timestamp - this.lastTime;
                if (dt > 100) dt = 16; // Cap delta time if tab gets backgrounded
                this.lastTime = timestamp;

                this.update(dt);
                this.draw();
                
                requestAnimationFrame((ts) => this.loop(ts));
            }
        }

        // --- Initialization ---
        const game = new Game();
        window.game = game;

        // Firebase Auth & Realtime Database Listener
        const initAuth = async () => {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (e) {
                console.error("Auth error", e);
            }
        };
        initAuth();

        onAuthStateChanged(auth, (user) => {
            currentUser = user;
            if(user) {
                const scoresRef = collection(db, 'artifacts', appId, 'public', 'data', 'highscores');
                onSnapshot(scoresRef, (snapshot) => {
                    let allScores = [];
                    snapshot.forEach(doc => {
                        allScores.push(doc.data().score);
                    });
                    
                    // Sort descending
                    allScores.sort((a, b) => b - a);
                    
                    if (window.game) {
                        // Keep top 5
                        window.game.highScores = allScores.slice(0, 5);
                        // Refresh UI dynamically if the highscore screen is active
                        if (!document.getElementById('highScoreScreen').classList.contains('hidden')) {
                            window.game.showHighScores();
                        }
                    }
                }, (error) => {
                    console.error("Firestore snapshot error:", error);
                });
            }
        });
        
        // Ensure fonts are loaded before first draw
        window.onload = () => {
            requestAnimationFrame((ts) => game.loop(ts));
        };

    </script>
</body>
</html>
