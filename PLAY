<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Neon Cat</title>
  <style>
    html, body {
      margin: 0;
      height: 100%;
      overflow: hidden;
      background: #02010a;
      font-family: Arial, sans-serif;
      color: #fff;
    }

    body {
      display: grid;
      place-items: center;
    }

    canvas {
      width: min(100vw, 980px);
      height: min(100vh, 620px);
      border: 1px solid rgba(255,255,255,0.12);
      box-shadow: 0 0 40px rgba(0, 255, 255, 0.2);
      background: radial-gradient(circle at top, #081224 0%, #02010a 60%, #000 100%);
    }

    .ui {
      position: fixed;
      top: 16px;
      left: 16px;
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
      pointer-events: auto;
      z-index: 5;
    }

    .panel {
      padding: 8px 12px;
      border: 1px solid rgba(255,255,255,0.18);
      background: rgba(0,0,0,0.35);
      border-radius: 999px;
      font-size: 14px;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      backdrop-filter: blur(6px);
      transition: transform 0.2s ease, background 0.2s ease;
    }

    .panel.button {
      cursor: pointer;
      background: rgba(0,255,255,0.14);
    }

    .panel.button:hover {
      transform: translateY(-1px);
      background: rgba(0,255,255,0.22);
    }

    .hint {
      position: fixed;
      bottom: 16px;
      left: 50%;
      transform: translateX(-50%);
      font-size: 12px;
      color: rgba(255,255,255,0.7);
      letter-spacing: 0.16em;
      text-transform: uppercase;
      pointer-events: none;
      text-align: center;
      max-width: min(90vw, 680px);
    }
  </style>
</head>
<body>
  <canvas id="game"></canvas>
  <div class="ui">
    <div class="panel" id="score">Score: 0</div>
    <div class="panel" id="best">Best: 0</div>
    <div class="panel" id="timer">Time: 0</div>
    <div class="panel" id="boost">Boost: OFF</div>
    <div class="panel button" id="sound">Sound: ON</div>
  </div>
  <div class="hint">Нажми Enter или Space, чтобы начать • WASD / Стрелки — двигайся • собирай неон-лампочки, избегай врагов</div>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const bestEl = document.getElementById('best');
    const timerEl = document.getElementById('timer');
    const boostEl = document.getElementById('boost');
    const soundEl = document.getElementById('sound');

    let audioContext = null;

    function initAudio() {
      if (audioContext) return;
      audioContext = new (window.AudioContext || window.webkitAudioContext)();
      if (audioContext.state === 'suspended') {
        audioContext.resume();
      }
    }

    function playTone(frequency, type = 'sine', duration = 0.1, volume = 0.12, delay = 0) {
      if (!state.soundEnabled || !audioContext) return;
      const now = audioContext.currentTime + delay;
      const osc = audioContext.createOscillator();
      const gain = audioContext.createGain();
      osc.type = type;
      osc.frequency.setValueAtTime(frequency, now);
      gain.gain.setValueAtTime(0.001, now);
      gain.gain.linearRampToValueAtTime(volume, now + 0.01);
      gain.gain.exponentialRampToValueAtTime(0.001, now + duration);
      osc.connect(gain);
      gain.connect(audioContext.destination);
      osc.start(now);
      osc.stop(now + duration + 0.02);
    }

    function playSound(type) {
      if (!state.soundEnabled) return;
      initAudio();
      if (!audioContext) return;
      switch (type) {
        case 'start':
          playTone(240, 'sine', 0.16, 0.12);
          playTone(320, 'triangle', 0.14, 0.08, 0.05);
          break;
        case 'collect':
          playTone(520, 'triangle', 0.08, 0.14);
          playTone(760, 'sine', 0.06, 0.08, 0.04);
          break;
        case 'gameover':
          playTone(180, 'sawtooth', 0.22, 0.12);
          playTone(120, 'sine', 0.14, 0.09, 0.06);
          break;
        case 'boost':
          playTone(420, 'square', 0.1, 0.14);
          break;
      }
    }

    function toggleSound() {
      state.soundEnabled = !state.soundEnabled;
      soundEl.textContent = `Sound: ${state.soundEnabled ? 'ON' : 'OFF'}`;
      if (state.soundEnabled) {
        initAudio();
        playSound('start');
      }
    }

    function resize() {
      const ratio = window.devicePixelRatio || 1;
      const width = Math.min(window.innerWidth * 0.96, 980);
      const height = Math.min(window.innerHeight * 0.92, 620);
      canvas.width = width * ratio;
      canvas.height = height * ratio;
      ctx.setTransform(ratio, 0, 0, ratio, 0, 0);
      state.width = width;
      state.height = height;
      state.bgGradient = ctx.createLinearGradient(0, 0, 0, height);
      state.bgGradient.addColorStop(0, '#061127');
      state.bgGradient.addColorStop(1, '#010107');
    }

    window.addEventListener('resize', resize);

    const keys = {};
    window.addEventListener('keydown', (e) => {
      keys[e.key.toLowerCase()] = true;
      if (e.key === ' ' || e.key === 'Spacebar' || e.key === 'Enter') {
        e.preventDefault();
        initAudio();
        if (!state.started || state.gameOver) {
          startGame();
        }
      }
    });

    canvas.addEventListener('click', () => {
      initAudio();
      if (!state.started || state.gameOver) {
        startGame();
      }
    });
    window.addEventListener('keyup', (e) => {
      keys[e.key.toLowerCase()] = false;
    });

    const state = {
      width: 900,
      height: 600,
      started: false,
      gameOver: false,
      score: 0,
      best: 0,
      player: { x: 220, y: 220, r: 24, speed: 230 },
      collectibles: [],
      enemies: [],
      stars: [],
      particles: [],
      flash: 0,
      time: 0,
      timer: 0,
      boost: false,
      soundEnabled: true
    };

    function resetStars() {
      state.stars = Array.from({ length: 55 }, () => ({
        x: Math.random() * state.width,
        y: Math.random() * state.height,
        r: Math.random() * 1.6 + 0.4,
        twinkle: Math.random() * Math.PI * 2
      }));
    }

    function spawnCollectibles() {
      state.collectibles = Array.from({ length: 5 }, () => ({
        x: Math.random() * (state.width - 60) + 30,
        y: Math.random() * (state.height - 60) + 30,
        r: 10,
        pulse: Math.random() * Math.PI * 2
      }));
    }

    function spawnEnemies() {
      state.enemies = Array.from({ length: 3 }, () => ({
        x: Math.random() * state.width,
        y: Math.random() * state.height,
        r: 16,
        vx: (Math.random() - 0.5) * 180,
        vy: (Math.random() - 0.5) * 180,
        hue: Math.random() * 40 + 320
      }));
    }

    function startGame() {
      initAudio();
      state.started = true;
      state.gameOver = false;
      state.score = 0;
      state.timer = 0;
      state.boost = false;
      state.player.x = state.width * 0.5;
      state.player.y = state.height * 0.5;
      state.particles = [];
      spawnCollectibles();
      spawnEnemies();
      playSound('start');
      updateHud();
    }

    function restart() {
      startGame();
    }

    function updateHud() {
      scoreEl.textContent = `Score: ${state.score}`;
      bestEl.textContent = `Best: ${state.best}`;
      timerEl.textContent = `Time: ${Math.floor(state.timer)}`;
      boostEl.textContent = state.boost ? 'Boost: ON' : 'Boost: OFF';
      soundEl.textContent = `Sound: ${state.soundEnabled ? 'ON' : 'OFF'}`;
    }

    function distance(ax, ay, bx, by) {
      return Math.hypot(ax - bx, ay - by);
    }

    function drawBackground(t) {
      ctx.fillStyle = state.bgGradient;
      ctx.fillRect(0, 0, state.width, state.height);

      ctx.save();
      ctx.strokeStyle = 'rgba(0,255,255,0.06)';
      ctx.lineWidth = 1;
      for (let i = 0; i < state.width; i += 60) {
        ctx.beginPath();
        ctx.moveTo(i, 0);
        ctx.lineTo(i, state.height);
        ctx.stroke();
      }
      for (let i = 0; i < state.height; i += 60) {
        ctx.beginPath();
        ctx.moveTo(0, i);
        ctx.lineTo(state.width, i);
        ctx.stroke();
      }
      ctx.restore();

      state.stars.forEach((star) => {
        const pulse = 0.7 + 0.3 * Math.sin(t * 0.001 + star.twinkle);
        ctx.beginPath();
        ctx.arc(star.x, star.y, star.r * pulse, 0, Math.PI * 2);
        ctx.fillStyle = `rgba(255,255,255,${0.4 + pulse * 0.3})`;
        ctx.fill();
      });
    }

    function drawPlayer() {
      const p = state.player;
      ctx.save();
      ctx.translate(p.x, p.y);
      ctx.shadowBlur = 26;
      ctx.shadowColor = '#00f7ff';

      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.ellipse(0, 2, 18, 16, 0, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.moveTo(-12, -10);
      ctx.lineTo(-24, -24);
      ctx.lineTo(-4, -14);
      ctx.closePath();
      ctx.fill();

      ctx.beginPath();
      ctx.moveTo(12, -10);
      ctx.lineTo(24, -24);
      ctx.lineTo(4, -14);
      ctx.closePath();
      ctx.fill();

      ctx.fillStyle = '#00f7ff';
      ctx.beginPath();
      ctx.arc(-6, -2, 4, 0, Math.PI * 2);
      ctx.arc(6, -2, 4, 0, Math.PI * 2);
      ctx.fill();

      ctx.fillStyle = '#000';
      ctx.beginPath();
      ctx.arc(-6, -2, 2, 0, Math.PI * 2);
      ctx.arc(6, -2, 2, 0, Math.PI * 2);
      ctx.fill();

      ctx.strokeStyle = '#ffffff';
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.moveTo(-8, 10);
      ctx.quadraticCurveTo(0, 18, 8, 10);
      ctx.stroke();

      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.moveTo(16, 8);
      ctx.quadraticCurveTo(24, 12, 30, 6);
      ctx.quadraticCurveTo(24, 2, 16, 8);
      ctx.closePath();
      ctx.fill();

      ctx.restore();
    }

    function drawCollectibles() {
      state.collectibles.forEach((item) => {
        const pulse = 0.7 + 0.3 * Math.sin(state.time * 0.004 + item.pulse);
        ctx.save();
        ctx.translate(item.x, item.y);
        ctx.scale(pulse, pulse);
        ctx.shadowBlur = 20;
        ctx.shadowColor = '#00f7ff';
        ctx.fillStyle = '#00f7ff';
        ctx.beginPath();
        ctx.moveTo(0, -12);
        ctx.lineTo(12, 0);
        ctx.lineTo(0, 12);
        ctx.lineTo(-12, 0);
        ctx.closePath();
        ctx.fill();
        ctx.fillStyle = 'rgba(255,255,255,0.8)';
        ctx.beginPath();
        ctx.arc(0, 0, 4, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
      });
    }

    function drawEnemies() {
      state.enemies.forEach((enemy) => {
        ctx.save();
        ctx.translate(enemy.x, enemy.y);
        ctx.shadowBlur = 18;
        ctx.shadowColor = `hsl(${enemy.hue}, 100%, 60%)`;
        ctx.fillStyle = `hsl(${enemy.hue}, 100%, 60%)`;
        ctx.beginPath();
        ctx.arc(0, 0, enemy.r, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = '#fff';
        ctx.beginPath();
        ctx.arc(-4, -2, 2, 0, Math.PI * 2);
        ctx.arc(4, -2, 2, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
      });
    }

    function update(dt) {
      if (state.gameOver) return;

      const right = !!(keys['d'] || keys['arrowright']);
      const left = !!(keys['a'] || keys['arrowleft']);
      const down = !!(keys['s'] || keys['arrowdown']);
      const up = !!(keys['w'] || keys['arrowup']);

      const boostFactor = state.boost ? 1.25 : 1;
      const moveX = (Number(right) - Number(left)) * state.player.speed * boostFactor * dt;
      const moveY = (Number(down) - Number(up)) * state.player.speed * boostFactor * dt;
      state.player.x += moveX;
      state.player.y += moveY;

      state.player.x = Math.max(state.player.r, Math.min(state.width - state.player.r, state.player.x));
      state.player.y = Math.max(state.player.r, Math.min(state.height - state.player.r, state.player.y));

      state.enemies.forEach((enemy) => {
        const dx = state.player.x - enemy.x;
        const dy = state.player.y - enemy.y;
        const dist = Math.hypot(dx, dy) || 1;
        enemy.x += (dx / dist) * 95 * dt;
        enemy.y += (dy / dist) * 95 * dt;
        enemy.x += enemy.vx * dt;
        enemy.y += enemy.vy * dt;
        if (enemy.x < enemy.r || enemy.x > state.width - enemy.r) enemy.vx *= -1;
        if (enemy.y < enemy.r || enemy.y > state.height - enemy.r) enemy.vy *= -1;
      });

      state.timer += dt;
      const boostActive = state.score > 0 && state.timer % 6 < 3;
      if (boostActive && !state.boost) {
        playSound('boost');
      }
      state.boost = boostActive;

      state.collectibles = state.collectibles.filter((item) => {
        if (distance(state.player.x, state.player.y, item.x, item.y) < 28) {
          state.score += 1;
          state.best = Math.max(state.best, state.score);
          updateHud();
          state.flash = 0.25;
          playSound('collect');
          spawnParticles(item.x, item.y, 180);
          return false;
        }
        return true;
      });

      if (state.collectibles.length < 4) {
        for (let i = 0; i < 3; i += 1) {
          state.collectibles.push({
            x: Math.random() * (state.width - 60) + 30,
            y: Math.random() * (state.height - 60) + 30,
            r: 10,
            pulse: Math.random() * Math.PI * 2
          });
        }
      }

      state.enemies.forEach((enemy) => {
        if (!state.gameOver && distance(state.player.x, state.player.y, enemy.x, enemy.y) < state.player.r + enemy.r) {
          state.gameOver = true;
          playSound('gameover');
        }
      });
      updateParticles(dt);
    }

    function drawOverlay() {
      ctx.save();
      if (!state.started) {
        ctx.fillStyle = 'rgba(0,0,0,0.72)';
        ctx.fillRect(0, 0, state.width, state.height);
        ctx.fillStyle = '#ffffff';
        ctx.font = 'bold 42px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('NEON CAT', state.width / 2, state.height / 2 - 40);
        ctx.font = '20px Arial';
        ctx.fillStyle = '#00f7ff';
        ctx.fillText('Нажми Enter или Space, чтобы стартовать', state.width / 2, state.height / 2 + 5);
        ctx.font = '16px Arial';
        ctx.fillStyle = '#dcefff';
        ctx.fillText('Управление: WASD или стрелки', state.width / 2, state.height / 2 + 40);
        ctx.fillText('Собирай неон-лампочки и не сталкивайся с врагами', state.width / 2, state.height / 2 + 66);
        ctx.restore();
        return;
      }

      if (!state.gameOver) return;
      ctx.fillStyle = 'rgba(0,0,0,0.72)';
      ctx.fillRect(0, 0, state.width, state.height);
      ctx.fillStyle = '#fff';
      ctx.font = 'bold 36px Arial';
      ctx.textAlign = 'center';
      ctx.fillText('GAME OVER', state.width / 2, state.height / 2 - 12);
      ctx.font = '18px Arial';
      ctx.fillStyle = '#00f7ff';
      ctx.fillText('Нажми Enter или Space, чтобы играть снова', state.width / 2, state.height / 2 + 22);
      ctx.restore();
    }

    function drawFlash() {
      if (state.flash > 0) {
        state.flash = Math.max(0, state.flash - 0.02);
        ctx.save();
        ctx.globalAlpha = state.flash * 0.5;
        ctx.fillStyle = '#00f7ff';
        ctx.fillRect(0, 0, state.width, state.height);
        ctx.restore();
      }
    }

    function drawParticles() {
      state.particles.forEach((particle) => {
        ctx.save();
        ctx.globalAlpha = particle.life;
        ctx.fillStyle = `hsla(${particle.hue}, 100%, 82%, ${particle.life})`;
        ctx.beginPath();
        ctx.arc(particle.x, particle.y, particle.r, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
      });
    }

    function updateParticles(dt) {
      state.particles = state.particles.filter((particle) => {
        particle.x += particle.vx * dt;
        particle.y += particle.vy * dt;
        particle.life -= dt * 1.4;
        return particle.life > 0;
      });
    }

    function spawnParticles(x, y, hue) {
      for (let i = 0; i < 10; i += 1) {
        state.particles.push({
          x,
          y,
          vx: (Math.random() - 0.5) * 140,
          vy: (Math.random() - 0.5) * 140,
          r: Math.random() * 2.4 + 1.2,
          life: 1,
          hue
        });
      }
    }

    function loop(timestamp) {
      const dt = Math.min(0.032, (timestamp - (loop.lastTime || timestamp)) / 1000);
      loop.lastTime = timestamp;
      state.time = timestamp;

      if (state.started && !state.gameOver) {
        update(dt);
      }

      drawBackground(timestamp);
      if (state.started) {
        drawCollectibles();
        drawEnemies();
        drawPlayer();
        drawParticles();
        drawFlash();
      }
      if (state.started && !state.gameOver) {
        updateHud();
      }
      drawOverlay();
      requestAnimationFrame(loop);
    }

    soundEl.addEventListener('click', toggleSound);
    resize();
    resetStars();
    updateHud();
    requestAnimationFrame(loop);
  </script>
</body>
</html>
