<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Arcade Space Shooter</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');
        
        body {
            font-family: 'Press Start 2P', monospace;
            /* Cool Blurry Space Background */
            background-color: #0A0A1F;
            background-image: 
                radial-gradient(circle at 10% 20%, rgba(200, 50, 255, 0.1) 0%, rgba(0, 0, 0, 0) 30%),
                radial-gradient(circle at 90% 80%, rgba(50, 150, 255, 0.15) 0%, rgba(0, 0, 0, 0) 40%),
                radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.02) 0%, rgba(0, 0, 0, 0) 100%);
            color: #ffffff;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            overflow: hidden;
            transition: background-image 2s ease;
        }

        .game-container {
            width: 100%;
            max-width: 600px;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 1rem;
        }

        #gameCanvas {
            border: 4px solid #3b82f6;
            background-color: #000010;
            box-shadow: 0 0 20px rgba(59, 130, 246, 0.5);
            cursor: crosshair;
            touch-action: none;
            width: 100%;
            height: 400px;
        }

        .hud {
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 600px;
            margin-bottom: 1rem;
            font-size: 0.75rem;
            gap: 1rem;
        }

        .hud > div {
            flex-grow: 1;
            text-align: center;
        }

        .control-panel {
            margin-top: 1rem;
            width: 100%;
            display: flex;
            flex-direction: column;
            gap: 0.5rem;
            align-items: center;
        }

        /* Container for Start/Stop buttons */
        .button-row {
            display: flex;
            gap: 1rem;
            justify-content: center;
            width: 100%;
            max-width: 400px;
        }

        /* Glowy Outline Button Style */
        .game-button {
            flex-grow: 1; /* Allow buttons to grow inside button-row */
            background: transparent;
            color: #ffffff;
            font-weight: bold;
            padding: 0.75rem 1.5rem;
            border: 2px solid #3b82f6;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(59, 130, 246, 0.6), inset 0 0 5px rgba(59, 130, 246, 0.3);
            cursor: pointer;
            transition: all 0.2s ease;
            font-size: 0.8rem;
            text-transform: uppercase;
        }

        .game-button:hover {
            border-color: #60a5fa;
            box-shadow: 0 0 20px rgba(96, 165, 250, 0.8), inset 0 0 10px rgba(96, 165, 250, 0.5);
            transform: scale(1.03);
        }
        
        .game-button:active {
            border-color: #3b82f6;
            box-shadow: 0 0 5px rgba(59, 130, 246, 0.5), inset 0 0 2px rgba(59, 130, 246, 0.2);
            transform: scale(0.98);
        }
        
        /* Mobile adjustment: stack buttons vertically */
        @media (max-width: 639px) {
            .button-row {
                flex-direction: column; 
            }
        }

        .message-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: rgba(17, 24, 39, 0.95);
            border: 4px solid #ef4444;
            padding: 2rem;
            border-radius: 12px;
            text-align: center;
            box-shadow: 0 0 30px rgba(239, 68, 68, 0.8);
            z-index: 100;
            display: none;
        }

        @media (min-width: 640px) {
            #gameCanvas {
                height: 500px;
            }
            .hud {
                font-size: 1rem;
            }
        }
    </style>
</head>
<body>

    <div class="game-container">
        <div class="hud">
            <div id="scoreDisplay">SCORE: 0</div>
            <div id="highScoreDisplay" class="text-yellow-400">HIGH SCORE: 0</div>
            <div id="livesDisplay">LIVES: 3</div>
        </div>

        <canvas id="gameCanvas" width="600" height="400"></canvas>
        
        <div class="control-panel">
            <div class="button-row">
                <button id="startButton" class="game-button">Start Game</button>
                <button id="stopButton" class="game-button">Stop Game</button>
            </div>
            <p class="text-xs text-gray-400 mt-2">Controls: Arrow Keys or A/D to move, Space or W/Up to shoot. Touch for movement and auto-fire.</p>
        </div>
    </div>

    <!-- Message Box for Game Over / Pause --><div id="messageBox" class="message-box">
        <h2 id="messageTitle" class="text-2xl mb-4 text-red-400">GAME OVER</h2>
        <p id="messageText" class="mb-6 text-lg">Your final score was: 0</p>
        <button id="messageButton" class="game-button">Play Again</button>
    </div>

    <script>
        // Set up the global variables for the canvas and game state
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const livesDisplay = document.getElementById('livesDisplay');
        const startButton = document.getElementById('startButton');
        const stopButton = document.getElementById('stopButton');
        const messageBox = document.getElementById('messageBox');
        const messageTitle = document.getElementById('messageTitle');
        const messageText = document.getElementById('messageText');
        const messageButton = document.getElementById('messageButton');
        const highScoreDisplay = document.getElementById('highScoreDisplay');

        // --- Asset Configuration ---
        let assets = {
            loaded: true
        };
        // -------------------------


        let game = {
            running: false,
            paused: false,
            score: 0,
            lives: 3,
            lastTime: 0,
            deltaTime: 0,
            maxEnemies: 5,
            enemySpawnTimer: 0,
            enemySpawnInterval: 1.5, // seconds
            levelMultiplier: 1.0,
            difficultyStep: 30, // score points to increase difficulty
            highScore: 0 // New property for local persistence
        };

        let player = {
            x: canvas.width / 2,
            y: canvas.height - 30,
            width: 30, 
            height: 30, 
            speed: 250, // pixels per second
            color: '#38bdf8', // Tailwind sky-400 (Fallback color)
            bullets: [],
            bulletSpeed: 400, // pixels per second
            shootTimer: 0,
            shootInterval: 0.2 // seconds
        };

        let enemies = [];
        let particles = [];
        let keys = {};
        let touch = { active: false, x: 0, y: 0 };

        // --- Persistence Functions (Local Storage) ---

        function loadHighScore() {
            try {
                // Check if a score is saved in the browser's local storage
                const storedScore = localStorage.getItem('spaceShooterHighScore');
                if (storedScore) {
                    game.highScore = parseInt(storedScore, 10);
                }
            } catch (e) {
                console.warn("Could not load score from local storage:", e);
            }
        }

        function saveHighScore() {
            // Only save if the current score beats the high score
            if (game.score > game.highScore) {
                game.highScore = game.score;
                try {
                    localStorage.setItem('spaceShooterHighScore', game.highScore.toString());
                } catch (e) {
                    console.warn("Could not save score to local storage:", e);
                }
                return true; // Indicate a new high score was set
            }
            return false;
        }

        // --- Game Setup Functions ---

        /**
         * Converts base64 PCM audio data to a playable WAV blob.
         */
        function pcmToWav(pcmData, sampleRate) {
            const numChannels = 1;
            const bytesPerSample = 2; // 16-bit
            const blockAlign = numChannels * bytesPerSample;
            const byteRate = sampleRate * blockAlign;
            const dataSize = pcmData.length * bytesPerSample;

            const buffer = new ArrayBuffer(44 + dataSize);
            const view = new DataView(buffer);

            let offset = 0;

            // Write 4-byte string
            function writeString(s) {
                for (let i = 0; i < s.length; i++) {
                    view.setUint8(offset + i, s.charCodeAt(i));
                }
                offset += s.length;
            }

            // RIFF chunk
            writeString('RIFF');
            view.setUint32(offset, 36 + dataSize, true); offset += 4;
            writeString('WAVE');

            // FMT sub-chunk
            writeString('fmt ');
            view.setUint32(offset, 16, true); offset += 4; // Sub-chunk size
            view.setUint16(offset, 1, true); offset += 2; // Audio format (1=PCM)
            view.setUint16(offset, numChannels, true); offset += 2;
            view.setUint32(offset, sampleRate, true); offset += 4;
            view.setUint32(offset, byteRate, true); offset += 4;
            view.setUint16(offset, blockAlign, true); offset += 2;
            view.setUint16(offset, bytesPerSample * 8, true); offset += 2; // Bits per sample

            // DATA sub-chunk
            writeString('data');
            view.setUint32(offset, dataSize, true); offset += 4;

            // Write PCM data
            for (let i = 0; i < pcmData.length; i++) {
                view.setInt16(offset, pcmData[i], true);
                offset += 2;
            }

            return new Blob([buffer], { type: 'audio/wav' });
        }


        /**
         * Plays a sound effect using the Gemini TTS API.
         */
        async function playSound(text) {
             const apiKey = "";
             const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=${apiKey}`;

             const payload = {
                contents: [{ parts: [{ text: text }] }],
                generationConfig: {
                    responseModalities: ["AUDIO"],
                    speechConfig: {
                        voiceConfig: { prebuiltVoiceConfig: { voiceName: "Puck" } }
                    }
                },
                model: "gemini-2.5-flash-preview-tts"
            };

            // Helper to convert base64 to ArrayBuffer
            function base64ToArrayBuffer(base64) {
                const binaryString = atob(base64);
                const len = binaryString.length;
                const bytes = new Uint8Array(len);
                for (let i = 0; i < len; i++) {
                    bytes[i] = binaryString.charCodeAt(i);
                }
                return bytes.buffer;
            }

            // Exponential backoff retry logic
            for (let attempt = 0; attempt < 3; attempt++) {
                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });

                    if (!response.ok) throw new Error(`API call failed with status ${response.status}`);

                    const result = await response.json();
                    const part = result?.candidates?.[0]?.content?.parts?.[0];
                    const audioData = part?.inlineData?.data;
                    const mimeType = part?.inlineData?.mimeType;

                    if (audioData && mimeType && mimeType.startsWith("audio/L16")) {
                        const sampleRateMatch = mimeType.match(/rate=(\d+)/);
                        const sampleRate = sampleRateMatch ? parseInt(sampleRateMatch[1], 10) : 16000; // Defaulting to 16kHz
                        const pcmData = base64ToArrayBuffer(audioData);
                        // API returns signed PCM16 audio data.
                        const pcm16 = new Int16Array(pcmData);
                        const wavBlob = pcmToWav(pcm16, sampleRate);
                        const audioUrl = URL.createObjectURL(wavBlob);

                        const audio = new Audio(audioUrl);
                        audio.play().catch(e => console.error("Error playing sound:", e));
                        return; // Success, exit retry loop
                    } else {
                        throw new Error("Invalid or missing audio data in response.");
                    }
                } catch (error) {
                    console.error(`Attempt ${attempt + 1} failed:`, error);
                    if (attempt < 2) {
                        const delay = Math.pow(2, attempt) * 1000;
                        await new Promise(resolve => setTimeout(resolve, delay));
                    }
                }
            }
            console.error("Failed to play sound after multiple retries.");
        }


        function resetGame() {
            game.score = 0;
            game.lives = 3;
            game.running = true;
            game.paused = false; // Reset paused state
            game.levelMultiplier = 1.0;
            player.bullets = [];
            enemies = [];
            particles = [];
            game.enemySpawnTimer = 0;
            player.x = canvas.width / 2;
            
            updateHUD();
            messageBox.style.display = 'none';
        }

        function spawnEnemy() {
            const size = 15 + Math.random() * 15;
            enemies.push({
                x: Math.random() * (canvas.width - size * 2) + size,
                y: -size,
                size: size,
                speed: (50 + Math.random() * 100) * game.levelMultiplier, // Pixels per second
                color: '#ef4444' // Tailwind red-500
            });
        }

        // --- Drawing Functions ---

        function drawPlayer() {
            ctx.save();
            ctx.shadowColor = '#3b82f6';
            ctx.shadowBlur = 15;

            ctx.fillStyle = '#93c5fd';
            ctx.beginPath();
            ctx.moveTo(player.x, player.y - player.height / 2);
            ctx.lineTo(player.x - player.width / 2, player.y + player.height / 2);
            ctx.lineTo(player.x + player.width / 2, player.y + player.height / 2);
            ctx.closePath();
            ctx.fill();

            ctx.restore();
        }

        function drawBullets() {
            ctx.fillStyle = '#facc15';
            player.bullets.forEach(bullet => {
                ctx.beginPath();
                ctx.arc(bullet.x, bullet.y, 3, 0, Math.PI * 2);
                ctx.fill();
            });
        }

        function drawEnemies() {
            enemies.forEach(enemy => {
                ctx.save();

                ctx.shadowColor = '#ef4444';
                ctx.shadowBlur = 12;

                ctx.fillStyle = '#f87171';
                ctx.beginPath();
                ctx.moveTo(enemy.x, enemy.y + enemy.size / 2);
                ctx.lineTo(enemy.x - enemy.size / 2, enemy.y - enemy.size / 2);
                ctx.lineTo(enemy.x + enemy.size / 2, enemy.y - enemy.size / 2);
                ctx.closePath();
                ctx.fill();

                ctx.restore();
            });
        }

        function drawParticles() {
            particles.forEach(p => {
                ctx.fillStyle = p.color;
                ctx.globalAlpha = p.alpha;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();
                ctx.globalAlpha = 1;
            });
        }

        // --- Update Functions ---

        function updateHUD() {
            scoreDisplay.textContent = `SCORE: ${game.score}`;
            // FIX: Use Math.max(0, game.lives) to prevent negative counts when repeating the heart emoji
            livesDisplay.textContent = `LIVES: ${'❤️'.repeat(Math.max(0, game.lives))}`;
            highScoreDisplay.textContent = `HIGH SCORE: ${game.highScore}`;
            // Control visibility of Stop button
            stopButton.style.display = game.running ? 'inline-block' : 'none';
        }

        function updatePlayer(dt) {
            const movement = player.speed * dt;

            // Handle Keyboard/Touch Movement
            if (keys['ArrowLeft'] || keys['a']) {
                player.x -= movement;
            }
            if (keys['ArrowRight'] || keys['d']) {
                player.x += movement;
            }
            
            // Handle touch movement if active
            if (touch.active) {
                const targetX = touch.x;
                const diffX = targetX - player.x;
                const maxMove = movement * 1.5;
                
                if (Math.abs(diffX) > 5) {
                    player.x += Math.min(Math.abs(diffX), maxMove) * (diffX > 0 ? 1 : -1);
                }
            }

            // Keep player within bounds
            player.x = Math.max(player.width / 2, Math.min(canvas.width - player.width / 2, player.x));

            // Handle Shooting
            player.shootTimer -= dt;
            if (player.shootTimer <= 0 && (keys['Space'] || keys['w'] || keys['ArrowUp'] || touch.active)) {
                player.bullets.push({
                    x: player.x,
                    y: player.y - player.height / 2,
                    radius: 3
                });
                player.shootTimer = player.shootInterval;
            }
        }

        function updateBullets(dt) {
            player.bullets.forEach(bullet => {
                bullet.y -= player.bulletSpeed * dt;
            });
            // Remove bullets that go off screen
            player.bullets = player.bullets.filter(bullet => bullet.y > 0);
        }

        function updateEnemies(dt) {
            // Check for new enemy spawn
            game.enemySpawnTimer += dt;
            if (game.enemySpawnTimer >= game.enemySpawnInterval) {
                if (enemies.length < game.maxEnemies) {
                    spawnEnemy();
                }
                game.enemySpawnTimer = 0;
            }

            // Move enemies
            enemies.forEach(enemy => {
                enemy.y += enemy.speed * dt;
            });

            // Remove enemies that pass the bottom and reduce life
            const enemiesHitBottom = enemies.filter(enemy => enemy.y - enemy.size / 2 > canvas.height);
            enemiesHitBottom.forEach(() => {
                game.lives--;
                createExplosion(player.x, player.y + player.height / 2, '#38bdf8');
            });
            
            enemies = enemies.filter(enemy => enemy.y - enemy.size / 2 <= canvas.height);

            // Check for Game Over
            if (game.lives <= 0) {
                gameOver();
            }
        }

        function createExplosion(x, y, color) {
            for (let i = 0; i < 15; i++) {
                particles.push({
                    x: x,
                    y: y,
                    size: 2 + Math.random() * 3,
                    vx: (Math.random() - 0.5) * 150,
                    vy: (Math.random() - 0.5) * 150,
                    color: color,
                    life: 0.5 + Math.random() * 0.5,
                    alpha: 1
                });
            }
        }
        
        function updateParticles(dt) {
            for (let i = particles.length - 1; i >= 0; i--) {
                const p = particles[i];
                p.x += p.vx * dt;
                p.y += p.vy * dt;
                p.life -= dt;
                p.alpha = p.life / (0.5 + Math.random() * 0.5);

                if (p.life <= 0) {
                    particles.splice(i, 1);
                }
            }
        }

        function checkCollisions() {
            const bulletsToRemove = new Set();
            const enemiesToRemove = new Set();
            
            // Loop through all bullets and enemies
            for (let i = 0; i < player.bullets.length; i++) {
                const bullet = player.bullets[i];
                
                for (let j = 0; j < enemies.length; j++) {
                    const enemy = enemies[j];
                    
                    // Simple AABB (Axis-Aligned Bounding Box) collision check
                    const dx = bullet.x - enemy.x;
                    const dy = bullet.y - enemy.y;
                    const distance = Math.sqrt(dx * dx + dy * dy);
                    
                    if (distance < enemy.size / 2 + bullet.radius) {
                        // Collision detected!
                        bulletsToRemove.add(i);
                        enemiesToRemove.add(j);
                        game.score += 10;
                        updateDifficulty();
                        createExplosion(enemy.x, enemy.y, enemy.color);
                    }
                }
            }

            // Filter out destroyed objects
            player.bullets = player.bullets.filter((_, index) => !bulletsToRemove.has(index));
            enemies = enemies.filter((_, index) => !enemiesToRemove.has(index));
        }

        function updateDifficulty() {
            if (game.score % game.difficultyStep === 0 && game.score > 0) {
                game.levelMultiplier += 0.05; // 5% speed increase
                game.maxEnemies = Math.min(10, game.maxEnemies + 1); // Max enemies up to 10
                console.log(`Difficulty increased! Multiplier: ${game.levelMultiplier.toFixed(2)}`);
            }
        }

        function showPauseMenu() {
            game.running = false;
            game.paused = true;
            messageTitle.textContent = "GAME PAUSED";
            messageText.innerHTML = `Current Score: <span class="text-yellow-400">${game.score}</span>. Hit resume to continue.`;
            messageButton.textContent = "Resume Game";
            messageBox.style.display = 'block';
            updateHUD();
        }
        
        function gameOver() {
            game.running = false;
            game.paused = false; // Ensure paused is false on game over
            
            // Check and save high score locally
            const newHighScore = saveHighScore();

            messageTitle.textContent = "GAME OVER";
            
            let scoreMessage = `Your final score was: <span class="text-yellow-400">${game.score}</span>.`;
            
            if (newHighScore) {
                scoreMessage += `<br><span class="text-green-400">NEW HIGH SCORE! ${game.highScore}</span>`;
            } else {
                scoreMessage += `<br>High Score: <span class="text-yellow-400">${game.highScore}</span>`;
            }
            
            messageText.innerHTML = scoreMessage;
            messageButton.textContent = "Play Again";
            messageBox.style.display = 'block';
            updateHUD();
        }

        // --- Main Game Loop ---

        function gameLoop(currentTime) {
            if (!game.running) return;

            // Calculate delta time (time elapsed since last frame)
            if (game.lastTime === 0) game.lastTime = currentTime;
            game.deltaTime = (currentTime - game.lastTime) / 1000; // Convert to seconds
            game.lastTime = currentTime;

            // 1. Clear the canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 2. Update all objects
            updatePlayer(game.deltaTime);
            updateBullets(game.deltaTime);
            updateEnemies(game.deltaTime);
            updateParticles(game.deltaTime);
            
            // 3. Check for interactions
            checkCollisions();
            updateHUD();

            // 4. Draw all objects
            drawPlayer();
            drawBullets();
            drawEnemies();
            drawParticles();

            // Request the next frame
            requestAnimationFrame(gameLoop);
        }

        // --- Event Listeners ---

        startButton.addEventListener('click', () => {
            if (!game.running && !game.paused) {
                resetGame();
                requestAnimationFrame(gameLoop);
            }
        });
        
        stopButton.addEventListener('click', () => {
            if (game.running) {
                showPauseMenu();
            }
        });

        messageButton.addEventListener('click', () => {
            if (game.paused) {
                // Resume logic
                game.running = true;
                game.paused = false;
                messageBox.style.display = 'none';
                requestAnimationFrame(gameLoop); 
            } else if (!game.running && !game.paused) { 
                // Play Again / Start Game logic (Game Over screen)
                resetGame();
                requestAnimationFrame(gameLoop);
            }
        });

        window.addEventListener('keydown', (e) => {
            keys[e.key] = true;
            if(['Space', 'ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.key)) {
                e.preventDefault(); // Prevent scrolling with arrow keys/space
            }
        });

        window.addEventListener('keyup', (e) => {
            keys[e.key] = false;
        });

        // Touch input handling
        function handleTouch(e) {
            e.preventDefault();
            const rect = canvas.getBoundingClientRect();
            const touchX = e.touches[0].clientX - rect.left;
            const touchY = e.touches[0].clientY - rect.top;

            // Scale touch coordinates to canvas resolution
            touch.x = touchX * (canvas.width / rect.width);
            touch.y = touchY * (canvas.height / rect.height);
        }

        canvas.addEventListener('touchstart', (e) => {
            touch.active = true;
            handleTouch(e);
        });

        canvas.addEventListener('touchmove', handleTouch);

        canvas.addEventListener('touchend', () => {
            touch.active = false;
        });

        // Handle canvas resizing to maintain responsiveness
        function resizeCanvas() {
            const container = canvas.parentElement;
            // Set canvas size based on container dimensions
            canvas.width = container.clientWidth;
            canvas.height = container.clientHeight;
            
            // Adjust player initial position if needed (only on game start/reset)
            if (!game.running) {
                player.x = canvas.width / 2;
                player.y = canvas.height - 30;
            }
        }

        window.addEventListener('resize', resizeCanvas);

        // Initial setup
        window.onload = function() {
            resizeCanvas();
            loadHighScore(); // Load score immediately from local storage
            
            // Initial Start Screen
            messageTitle.textContent = "SPACE SHOOTER";
            messageText.innerHTML = `Destroy the glowing red triangles and survive! <br> Your current high score is: <span class="text-yellow-400">${game.highScore}</span>. Use the **Start Game** button to begin.`;
            messageButton.textContent = "Start Game";
            messageBox.style.display = 'block';
            updateHUD(); // Hides the Stop button initially
        };

    </script>
</body>
</html>
