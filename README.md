<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>3D VR Tetris - El Takipli</title>
    <!-- MediaPipe unpkg CDN (daha kararlı) -->
    <script src="https://unpkg.com/@mediapipe/hands@0.4.1646424915/hands.js" crossorigin="anonymous"></script>
    <script src="https://unpkg.com/@mediapipe/camera_utils@0.3.1640029078/camera_utils.js" crossorigin="anonymous"></script>
    <style>
        /* --- Aynı stil, sadece status mesajı ve buton eklendi --- */
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap');
        * { margin:0; padding:0; box-sizing:border-box; font-family:'Orbitron',sans-serif; user-select:none; -webkit-tap-highlight-color:transparent; }
        body {
            background:#0a0a12;
            background-image:radial-gradient(circle at 10% 20%, rgba(100,0,255,0.15) 0%, transparent 30%),
                             radial-gradient(circle at 90% 80%, rgba(0,255,200,0.12) 0%, transparent 30%);
            color:#fff; height:100dvh; display:flex; flex-direction:column; justify-content:center; align-items:center;
            overflow:hidden; touch-action:manipulation; padding:8px;
        }
        .game-wrapper {
            display:flex; flex-direction:column; align-items:center; gap:10px; padding:12px;
            background:rgba(255,255,255,0.03); border-radius:28px; border:1px solid rgba(255,255,255,0.08);
            backdrop-filter:blur(12px); box-shadow:0 20px 60px rgba(0,0,0,0.7), inset 0 0 30px rgba(255,255,255,0.03);
            max-width:100vw; width:fit-content;
        }
        .board-wrapper { position:relative; padding:8px; border-radius:18px; background:rgba(0,0,0,0.7);
            border:2px solid rgba(0,255,200,0.25); box-shadow:0 0 40px rgba(0,255,200,0.15), inset 0 0 40px rgba(0,0,0,0.9); }
        canvas { display:block; border-radius:10px; background:#050508;
            background-image:linear-gradient(rgba(255,255,255,0.02) 1px, transparent 1px),
                             linear-gradient(90deg, rgba(255,255,255,0.02) 1px, transparent 1px);
            background-size:32px 32px; width:100%; height:auto; aspect-ratio:640/640; touch-action:none; image-rendering:pixelated; }
        .game-over-overlay { position:absolute; top:0; left:0; right:0; bottom:0; background:rgba(0,0,0,0.8);
            display:flex; flex-direction:column; justify-content:center; align-items:center; border-radius:18px;
            opacity:0; pointer-events:none; transition:opacity 0.4s; backdrop-filter:blur(6px); }
        .game-over-overlay.active { opacity:1; pointer-events:all; }
        .game-over-text { font-size:clamp(24px,6vw,44px); color:#ff0055; text-shadow:0 0 30px #ff0055; margin-bottom:16px; text-align:center; letter-spacing:2px; }
        .restart-btn { padding:14px 32px; background:linear-gradient(135deg,#00ffc8,#00a8ff); border:none; border-radius:12px;
            color:#000; font-weight:900; font-size:clamp(14px,3vw,20px); cursor:pointer; font-family:'Orbitron',sans-serif;
            transition:transform 0.15s, box-shadow 0.3s; box-shadow:0 0 30px rgba(0,255,200,0.3); }
        .restart-btn:hover { transform:scale(1.05); box-shadow:0 0 50px rgba(0,255,200,0.5); }
        .info-bar { display:flex; justify-content:space-around; width:100%; max-width:640px; padding:6px 12px;
            background:rgba(0,0,0,0.4); border-radius:14px; border:1px solid rgba(255,255,255,0.06);
            font-size:clamp(10px,2.2vw,14px); color:rgba(255,255,255,0.75); gap:6px; flex-wrap:wrap; }
        .info-bar span { color:#00ffc8; font-weight:900; font-size:clamp(14px,3vw,20px); text-shadow:0 0 12px rgba(0,255,200,0.4); }
        .info-item { display:flex; align-items:center; gap:5px; }
        .controls { display:flex; gap:8px; flex-wrap:wrap; justify-content:center; padding:6px 4px;
            background:rgba(0,0,0,0.3); border-radius:18px; border:1px solid rgba(255,255,255,0.06); }
        .ctrl-btn { background:rgba(255,255,255,0.06); border:1px solid rgba(255,255,255,0.12); color:#fff;
            font-size:clamp(18px,4vw,28px); font-weight:bold; width:clamp(48px,11vw,66px); height:clamp(48px,11vw,66px);
            border-radius:16px; display:flex; align-items:center; justify-content:center; cursor:pointer;
            touch-action:manipulation; transition:background 0.1s, transform 0.08s; box-shadow:0 4px 0 rgba(0,0,0,0.5);
            font-family:'Orbitron',sans-serif; }
        .ctrl-btn:active { transform:scale(0.90); background:rgba(0,255,200,0.2); }
        .ctrl-btn.wide { width:clamp(60px,14vw,80px); }

        .status-msg {
            font-size:clamp(12px,2.5vw,18px);
            color:#ffcc00;
            text-shadow:0 0 15px rgba(255,204,0,0.5);
            padding:6px 16px;
            background:rgba(0,0,0,0.6);
            border-radius:30px;
            border:1px solid rgba(255,204,0,0.2);
            text-align:center;
            margin-top:4px;
            min-height:40px;
            display:flex;
            align-items:center;
            justify-content:center;
            gap:10px;
        }
        .cam-btn {
            background:linear-gradient(135deg,#00ffc8,#00a8ff);
            border:none;
            border-radius:20px;
            color:#000;
            font-weight:bold;
            font-size:clamp(12px,2vw,16px);
            padding:6px 18px;
            cursor:pointer;
            font-family:'Orbitron',sans-serif;
            transition:transform 0.15s;
        }
        .cam-btn:active { transform:scale(0.95); }
        .cam-btn:disabled { opacity:0.5; pointer-events:none; }

        @media (max-width:480px) { .game-wrapper { padding:6px; gap:6px; border-radius:18px; }
            .board-wrapper { padding:4px; } .controls { gap:4px; padding:4px; } .info-bar { padding:4px 8px; } }
    </style>
</head>
<body>
<div class="game-wrapper">
    <div class="board-wrapper">
        <canvas id="gameCanvas" width="640" height="640"></canvas>
        <div class="game-over-overlay" id="gameOverOverlay">
            <div class="game-over-text">OYUN BİTTİ</div>
            <button class="restart-btn" id="restartBtn">YENİDEN BAŞLAT</button>
        </div>
    </div>
    <div class="info-bar">
        <div class="info-item">🏆 <span id="score">0</span></div>
        <div class="info-item">📈 <span id="level">1</span></div>
        <div class="info-item">📊 <span id="lines">0</span></div>
    </div>
    <div class="controls">
        <button class="ctrl-btn" id="btnLeft">◀</button>
        <button class="ctrl-btn" id="btnRight">▶</button>
        <button class="ctrl-btn" id="btnRotate">↻</button>
        <button class="ctrl-btn" id="btnDown">▼</button>
        <button class="ctrl-btn wide" id="btnDrop">⤓</button>
    </div>
    <div class="status-msg" id="statusMsg">
        <span>⏳ Oyun hazır</span>
        <button class="cam-btn" id="camStartBtn">📷 Kamerayı Başlat</button>
    </div>
</div>

<!-- Kamera önizlemesi (isteğe bağlı) -->
<div id="camera-preview" style="display:none;">
    <video id="webcam" playsinline></video>
</div>

<script>
    (function() {
        'use strict';

        // ---------- OYUN DEĞİŞKENLERİ -----------
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const statusMsg = document.getElementById('statusMsg');
        const camBtn = document.getElementById('camStartBtn');

        const COLS = 10,
            ROWS = 20,
            BLOCK_SIZE = 32,
            PADDING = 1,
            CANVAS_W = 640,
            CANVAS_H = 640,
            HALF_W = 320,
            PARALLAX = 3;

        const COLORS = { I: '#00ffc8', O: '#ffcc00', T: '#b300ff', S: '#00ff66', Z: '#ff0055', J: '#0066ff', L: '#ff8800' };
        const SHAPES = {
            I: [
                [1, 1, 1, 1]
            ],
            O: [
                [1, 1],
                [1, 1]
            ],
            T: [
                [0, 1, 0],
                [1, 1, 1]
            ],
            S: [
                [0, 1, 1],
                [1, 1, 0]
            ],
            Z: [
                [1, 1, 0],
                [0, 1, 1]
            ],
            J: [
                [1, 0, 0],
                [1, 1, 1]
            ],
            L: [
                [0, 0, 1],
                [1, 1, 1]
            ]
        };

        let board = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
        let currentPiece = null,
            nextPiece = null;
        let score = 0,
            lines = 0,
            level = 1,
            gameOver = false;
        let dropCounter = 0,
            dropInterval = 1000,
            lastTime = 0;
        let loopRunning = false;

        // ---------- SES ----------
        let audioCtx = null;

        function initAudio() {
            if (!audioCtx) {
                try { audioCtx = new(window.AudioContext || window.webkitAudioContext)(); } catch (_) {}
            }
        }

        function playTone(f, d, t = 'sine', v = 0.25) {
            if (!audioCtx) return;
            try {
                const o = audioCtx.createOscillator(),
                    g = audioCtx.createGain();
                o.type = t;
                o.frequency.value = f;
                g.gain.setValueAtTime(v, audioCtx.currentTime);
                g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + d);
                o.connect(g);
                g.connect(audioCtx.destination);
                o.start();
                o.stop(audioCtx.currentTime + d);
            } catch (_) {}
        }

        function playMove() { initAudio();
            playTone(600, 0.05, 'sine', 0.08); }

        function playRotate() { initAudio();
            playTone(800, 0.07, 'sine', 0.12);
            setTimeout(() => playTone(1000, 0.07, 'sine', 0.10), 80); }

        function playDrop() { initAudio();
            playTone(180, 0.15, 'sawtooth', 0.18); }

        function playClearLines() { initAudio();
            playTone(500, 0.08, 'square', 0.12);
            setTimeout(() => playTone(700, 0.08, 'square', 0.12), 120);
            setTimeout(() => playTone(900, 0.15, 'square', 0.12), 240); }

        function playGameOver() { initAudio();
            playTone(300, 0.25, 'sawtooth', 0.2);
            setTimeout(() => playTone(200, 0.25, 'sawtooth', 0.2), 280);
            setTimeout(() => playTone(100, 0.40, 'sawtooth', 0.2), 560); }

        // ---------- 3D BLOK ÇİZİM ----------
        function draw3DBlock(context, x, y, color, size = BLOCK_SIZE, isGhost = false) {
            const bs = size - PADDING * 2,
                xo = x + PADDING,
                yo = y + PADDING;
            if (isGhost) {
                context.save();
                context.globalAlpha = 0.25;
                context.strokeStyle = color;
                context.lineWidth = 2;
                context.setLineDash([3, 5]);
                context.strokeRect(xo + 1, yo + 1, bs - 2, bs - 2);
                context.restore();
                return;
            }
            context.fillStyle = color;
            context.fillRect(xo, yo, bs, bs);
            const g = context.createLinearGradient(xo, yo, xo, yo + bs);
            g.addColorStop(0, 'rgba(255,255,255,0.35)');
            g.addColorStop(0.4, 'rgba(255,255,255,0.05)');
            g.addColorStop(0.7, 'rgba(0,0,0,0.05)');
            g.addColorStop(1, 'rgba(0,0,0,0.50)');
            context.fillStyle = g;
            context.fillRect(xo, yo, bs, bs);
            context.fillStyle = 'rgba(255,255,255,0.60)';
            context.fillRect(xo, yo, bs, 2.5);
            context.fillRect(xo, yo, 2.5, bs);
            context.fillStyle = 'rgba(0,0,0,0.70)';
            context.fillRect(xo, yo + bs - 2.5, bs, 2.5);
            context.fillRect(xo + bs - 2.5, yo, 2.5, bs);
            context.strokeStyle = 'rgba(255,255,255,0.15)';
            context.lineWidth = 1;
            context.strokeRect(xo + 2, yo + 2, bs - 4, bs - 4);
            context.strokeStyle = 'rgba(0,0,0,0.6)';
            context.lineWidth = 1;
            context.strokeRect(xo, yo, bs, bs);
        }

        // ---------- OYUN MEKANİKLERİ ----------
        function createPiece(type) { return { type, shape: SHAPES[type].map(r => [...r]), color: COLORS[type], x: Math.floor(COLS / 2) - Math.ceil(SHAPES[type][0].length / 2), y: 0 }; }

        function randomPiece() { const types = Object.keys(SHAPES); return createPiece(types[Math.floor(Math.random() * types.length)]); }

        function collide(piece, board) {
            for (let y = 0; y < piece.shape.length; y++)
                for (let x = 0; x < piece.shape[y].length; x++)
                    if (piece.shape[y][x]) {
                        const bx = piece.x + x,
                            by = piece.y + y;
                        if (bx < 0 || bx >= COLS || by >= ROWS || (by >= 0 && board[by][bx])) return true;
                    }
            return false;
        }

        function merge(piece, board) { piece.shape.forEach((row, y) => row.forEach((v, x) => { if (v) board[piece.y + y][piece.x + x] = piece.color; })); }

        function rotatePiece(piece) {
            const ns = piece.shape[0].map((_, i) => piece.shape.map(row => row[i]).reverse());
            const os = piece.shape;
            piece.shape = ns;
            if (collide(piece, board)) { piece.shape = os; } else { playRotate(); }
        }

        function clearLines() {
            let cleared = 0;
            outer: for (let y = ROWS - 1; y >= 0; y--) {
                for (let x = 0; x < COLS; x++) { if (!board[y][x]) continue outer; }
                board.splice(y, 1);
                board.unshift(Array(COLS).fill(0));
                cleared++;
                y++;
            }
            if (cleared > 0) {
                lines += cleared;
                score += cleared * 100 * level;
                level = Math.floor(lines / 10) + 1;
                dropInterval = Math.max(80, 1000 - (level - 1) * 90);
                updateUI();
                playClearLines();
            }
        }

        function getGhostY() {
            let gy = currentPiece.y;
            while (!collide({ ...currentPiece, y: gy + 1 }, board)) gy++;
            return gy;
        }

        // ---------- ÇİZİM ----------
        function drawBoard(offsetX, shiftX) {
            board.forEach((row, y) => row.forEach((color, x) => { if (color) { const px = offsetX + x * BLOCK_SIZE + shiftX,
                            py = y * BLOCK_SIZE;
                        draw3DBlock(ctx, px, py, color); } }));
            if (currentPiece) {
                const ghostY = getGhostY();
                currentPiece.shape.forEach((row, y) => row.forEach((v, x) => {
                    if (v) {
                        const px = offsetX + (currentPiece.x + x) * BLOCK_SIZE + shiftX,
                            py = (ghostY + y) * BLOCK_SIZE;
                        draw3DBlock(ctx, px, py, currentPiece.color, BLOCK_SIZE, true);
                    }
                }));
                ctx.shadowBlur = 20;
                ctx.shadowColor = currentPiece.color;
                currentPiece.shape.forEach((row, y) => row.forEach((v, x) => {
                    if (v) {
                        const px = offsetX + (currentPiece.x + x) * BLOCK_SIZE + shiftX,
                            py = (currentPiece.y + y) * BLOCK_SIZE;
                        draw3DBlock(ctx, px, py, currentPiece.color);
                    }
                }));
                ctx.shadowBlur = 0;
            }
            ctx.strokeStyle = 'rgba(255,255,255,0.06)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(offsetX + HALF_W, 0);
            ctx.lineTo(offsetX + HALF_W, CANVAS_H);
            ctx.stroke();
        }

        function drawUI(offsetX) {
            ctx.fillStyle = 'rgba(255,255,255,0.75)';
            ctx.font = 'bold 14px Orbitron, sans-serif';
            ctx.textAlign = 'left';
            ctx.textBaseline = 'top';
            ctx.shadowBlur = 10;
            ctx.shadowColor = 'rgba(0,0,0,0.8)';
            ctx.fillText(`SKOR:${score}  LV:${level}  L:${lines}`, offsetX + 12, 12);
            ctx.shadowBlur = 0;
            if (nextPiece) {
                const nx = offsetX + HALF_W - 64,
                    ny = 10;
                ctx.fillStyle = 'rgba(0,0,0,0.55)';
                ctx.shadowBlur = 20;
                ctx.shadowColor = 'rgba(0,0,0,0.9)';
                ctx.beginPath();
                ctx.rect(nx, ny, 128, 84);
                ctx.fill();
                ctx.shadowBlur = 0;
                ctx.strokeStyle = 'rgba(255,255,255,0.08)';
                ctx.lineWidth = 1;
                ctx.strokeRect(nx, ny, 128, 84);
                ctx.fillStyle = 'rgba(255,255,255,0.5)';
                ctx.font = '9px Orbitron, sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText('SIRADAKİ', nx + 64, ny + 8);
                const bs = 20,
                    shape = nextPiece.shape,
                    cols = shape[0].length,
                    rows = shape.length,
                    sx = nx + (128 - cols * bs) / 2,
                    sy = ny + 24 + (60 - rows * bs) / 2;
                shape.forEach((row, y) => row.forEach((v, x) => { if (v) { const px = sx + x * bs,
                                py = sy + y * bs;
                            draw3DBlock(ctx, px, py, nextPiece.color, bs); } }));
            }
        }

        function render() {
            ctx.clearRect(0, 0, CANVAS_W, CANVAS_H);
            drawBoard(0, 0);
            drawUI(0);
            drawBoard(HALF_W, -PARALLAX);
            drawUI(HALF_W);
            ctx.strokeStyle = 'rgba(255,255,255,0.04)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(HALF_W, 0);
            ctx.lineTo(HALF_W, CANVAS_H);
            ctx.stroke();
        }

        // ---------- OYUN KONTROLLERİ ----------
        function drop() {
            if (!currentPiece || gameOver) return;
            currentPiece.y++;
            if (collide(currentPiece, board)) {
                currentPiece.y--;
                merge(currentPiece, board);
                playDrop();
                clearLines();
                currentPiece = nextPiece;
                nextPiece = randomPiece();
                if (collide(currentPiece, board)) {
                    gameOver = true;
                    document.getElementById('gameOverOverlay').classList.add('active');
                    playGameOver();
                    statusMsg.innerHTML = '💀 Oyun Bitti!';
                }
            }
            dropCounter = 0;
        }

        function hardDrop() {
            if (!currentPiece || gameOver) return;
            while (!collide({ ...currentPiece, y: currentPiece.y + 1 }, board)) { currentPiece.y++;
                score += 2; }
            drop();
            updateUI();
        }

        function move(dir) {
            if (!currentPiece || gameOver) return;
            currentPiece.x += dir;
            if (collide(currentPiece, board)) { currentPiece.x -= dir; } else { playMove(); }
        }

        function doRotate() { if (!currentPiece || gameOver) return;
            rotatePiece(currentPiece); }

        function updateUI() {
            document.getElementById('score').innerText = score;
            document.getElementById('level').innerText = level;
            document.getElementById('lines').innerText = lines;
        }

        function update(time = 0) {
            if (gameOver) { loopRunning = false; return; }
            const delta = time - lastTime;
            lastTime = time;
            dropCounter += delta;
            if (dropCounter > dropInterval) drop();
            render();
            requestAnimationFrame(update);
        }

        function resetGame() {
            board = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
            score = 0;
            lines = 0;
            level = 1;
            dropInterval = 1000;
            gameOver = false;
            currentPiece = randomPiece();
            nextPiece = randomPiece();
            document.getElementById('gameOverOverlay').classList.remove('active');
            updateUI();
            initAudio();
            lastTime = 0;
            dropCounter = 0;
            if (!loopRunning) {
                loopRunning = true;
                requestAnimationFrame(update);
            }
            updateStatus('🎮 Oyun başladı!', cameraActive);
        }

        // ---------- DURUM MESAJI ----------
        function updateStatus(msg, camOk) {
            const statusText = camOk ? '✅ El takibi aktif' : '⚠️ El takibi pasif (butonları kullan)';
            statusMsg.innerHTML = `<span>${msg}</span> <span style="font-size:0.8em;color:#aaa;">${statusText}</span>`;
            if (!camOk) {
                // Kamera başlat butonunu göster
                camBtn.style.display = 'inline-block';
            } else {
                camBtn.style.display = 'none';
            }
        }

        // ---------- KLAVYE & BUTONLAR ----------
        document.addEventListener('keydown', e => {
            initAudio();
            if (gameOver) return;
            switch (e.key) {
                case 'ArrowLeft':
                    move(-1);
                    break;
                case 'ArrowRight':
                    move(1);
                    break;
                case 'ArrowDown':
                    drop();
                    score += 1;
                    updateUI();
                    break;
                case 'ArrowUp':
                    doRotate();
                    break;
                case ' ':
                    hardDrop();
                    break;
                default:
                    return;
            }
            e.preventDefault();
        });

        function setupButton(id, action) {
            const btn = document.getElementById(id);
            if (!btn) return;
            const h = (e) => { e.preventDefault();
                initAudio(); if (!gameOver) action(); };
            btn.addEventListener('touchstart', h, { passive: false });
            btn.addEventListener('mousedown', h);
        }
        setupButton('btnLeft', () => move(-1));
        setupButton('btnRight', () => move(1));
        setupButton('btnRotate', doRotate);
        setupButton('btnDown', () => { drop();
            score += 1;
            updateUI(); });
        setupButton('btnDrop', hardDrop);
        document.getElementById('restartBtn').addEventListener('click', resetGame);
        document.getElementById('restartBtn').addEventListener('touchstart', e => { e.preventDefault();
            resetGame(); }, { passive: false });

        // ---------- KAMERA VE EL TAKİBİ (BUTONLA BAŞLAT) ----------
        let cameraActive = false;
        let hands = null,
            camera = null;
        let leftHandX = 0.5,
            leftHandY = 0.5,
            rightHandX = 0.5,
            rightHandY = 0.5;
        let lastRotateTime = 0,
            lastHardDropTime = 0,
            lastMoveTime = 0;

        function onResults(results) {
            if (!results.multiHandLandmarks || results.multiHandLandmarks.length === 0) return;
            const landmarks = results.multiHandLandmarks;
            const handedness = results.multiHandedness || [];
            let lx = 0.5,
                ly = 0.5,
                rx = 0.5,
                ry = 0.5;
            let hasLeft = false,
                hasRight = false;
            for (let i = 0; i < landmarks.length; i++) {
                const wrist = landmarks[i][0];
                const x = wrist.x,
                    y = wrist.y;
                const label = handedness[i]?.label || (i === 0 ? 'Left' : 'Right');
                if (label === 'Left') { lx = x;
                    ly = y;
                    hasLeft = true; } else { rx = x;
                    ry = y;
                    hasRight = true; }
            }
            if (hasLeft) { leftHandX = lx;
                leftHandY = ly; }
            if (hasRight) { rightHandX = rx;
                rightHandY = ry; }
            if (gameOver) return;
            const now = performance.now();
            if (hasRight) {
                if (rightHandX < 0.25 && now - lastMoveTime > 120) { move(-1);
                    lastMoveTime = now; } else if (rightHandX > 0.75 && now - lastMoveTime > 120) { move(1);
                    lastMoveTime = now; }
            }
            if (hasLeft && leftHandY < 0.25 && now - lastRotateTime > 300) { doRotate();
                lastRotateTime = now; }
            if (hasLeft && hasRight && leftHandY > 0.80 && rightHandY > 0.80 && now - lastHardDropTime > 400) { hardDrop();
                lastHardDropTime = now; }
        }

        // Kamera başlatma fonksiyonu (butonla tetiklenir)
        async function startCamera() {
            if (cameraActive) return;
            try {
                // MediaPipe sınıflarının yüklü olduğundan emin ol
                if (typeof Hands === 'undefined' || typeof Camera === 'undefined') {
                    throw new Error('MediaPipe kütüphaneleri yüklenemedi.');
                }

                hands = new Hands({
                    locateFile: (file) => `https://unpkg.com/@mediapipe/hands@0.4.1646424915/${file}`
                });

                hands.setOptions({
                    maxNumHands: 2,
                    modelComplexity: 1,
                    minDetectionConfidence: 0.7,
                    minTrackingConfidence: 0.5
                });

                hands.onResults(onResults);

                const videoElement = document.getElementById('webcam');
                camera = new Camera(videoElement, {
                    onFrame: async () => {
                        try {
                            await hands.send({ image: videoElement });
                        } catch (_) { /* sessiz */ }
                    },
                    width: 640,
                    height: 480
                });

                await camera.start();
                cameraActive = true;
                console.log('✅ Kamera ve el takibi aktif.');
                updateStatus('🎮 El takibi aktif!', true);
                // Kamera önizlemesini göstermek isterseniz:
                // document.getElementById('camera-preview').style.display = 'block';
                camBtn.style.display = 'none';
            } catch (err) {
                console.warn('⚠️ Kamera başlatılamadı:', err);
                cameraActive = false;
                updateStatus('❌ Kamera başlatılamadı, butonları kullanın.', false);
                camBtn.style.display = 'inline-block';
                camBtn.textContent = '🔄 Tekrar Dene';
            }
        }

        // Butona tıklandığında kamera başlat
        camBtn.addEventListener('click', startCamera);
        camBtn.addEventListener('touchstart', (e) => { e.preventDefault();
            startCamera(); }, { passive: false });

        // Sayfa yüklendiğinde oyunu başlat, kamera başlatma butonu görünsün
        window.addEventListener('load', () => {
            resetGame();
            updateStatus('📷 Kamerayı başlatmak için butona tıklayın', false);
            camBtn.style.display = 'inline-block';
        });

        // Eğer MediaPipe zaten yüklenmişse butonu aktif et
        setTimeout(() => {
            if (typeof Hands !== 'undefined' && !cameraActive) {
                camBtn.disabled = false;
            }
        }, 1000);

    })();
</script>
</body>
</html>
