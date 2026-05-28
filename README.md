<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>窓壊しバッティング！ - スケッチ「IMG_1915 2.jpeg」より</title>
    <!-- Tailwind CSS for UI styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Font "Inter" & "Mochiy Pop One" (for playful Japanese text) -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Mochiy+Pop+One&family=Outfit:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Outfit', 'Mochiy Pop One', 'sans-serif';
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
            background-color: #0f172a;
        }
        .retro-btn {
            box-shadow: 0 6px 0 #1e293b;
            transition: all 0.1s ease;
        }
        .retro-btn:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #1e293b;
        }
        .retro-btn-red {
            box-shadow: 0 6px 0 #991b1b;
        }
        .retro-btn-red:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #991b1b;
        }
        .retro-btn-green {
            box-shadow: 0 6px 0 #166534;
        }
        .retro-btn-green:active {
            transform: translateY(4px);
            box-shadow: 0 2px 0 #166534;
        }
        /* Shake effect for warnings */
        @keyframes shake {
            0%, 100% { transform: translate(0, 0) rotate(0deg); }
            20% { transform: translate(-3px, 3px) rotate(-1deg); }
            40% { transform: translate(3px, -2px) rotate(1deg); }
            60% { transform: translate(-3px, -3px) rotate(-1deg); }
            80% { transform: translate(3px, 3px) rotate(1deg); }
        }
        .shake-element {
            animation: shake 0.3s infinite;
        }
    </style>
</head>
<body class="overflow-hidden flex flex-col justify-between min-h-screen text-slate-100 p-2 md:p-4">

<div class="w-full max-w-4xl mx-auto flex-1 flex flex-col bg-slate-900 border-4 border-slate-700 rounded-3xl overflow-hidden shadow-2xl relative">
    
    <!-- Top HUD Bar -->
    <div class="bg-slate-800 p-3 md:p-4 border-b-4 border-slate-700 flex flex-wrap items-center justify-between gap-2 z-10">
        <!-- Score & Combo -->
        <div class="flex items-center space-x-4">
            <div>
                <span class="text-xs text-slate-400 block uppercase font-bold tracking-wider">SCORE</span>
                <span id="score-val" class="text-2xl md:text-3xl font-black text-amber-400">00000</span>
            </div>
            <div id="combo-container" class="opacity-0 scale-75 transition-all duration-200 bg-red-500/20 border border-red-500/40 px-2.5 py-1 rounded-xl text-center">
                <span class="text-[10px] text-red-400 block font-bold leading-none">COMBO</span>
                <span id="combo-val" class="text-lg font-black text-red-400 leading-none">x1</span>
            </div>
        </div>

        <!-- Warn Danger Bar (Resident's Awareness Indicator) -->
        <div class="flex-1 max-w-xs mx-2">
            <div class="flex justify-between text-[11px] font-bold text-slate-400 mb-1">
                <span class="flex items-center gap-1">
                    <span id="warning-icon" class="text-xs">🤫</span> 住民の気配 (Danger)
                </span>
                <span id="warning-status" class="text-emerald-400">安全 (SAFE)</span>
            </div>
            <div class="w-full bg-slate-950 h-3 rounded-full overflow-hidden p-[2px] border border-slate-700">
                <div id="danger-bar" class="w-0 h-full bg-gradient-to-r from-yellow-400 to-red-500 rounded-full transition-all duration-100"></div>
            </div>
        </div>

        <!-- Timer -->
        <div class="text-right">
            <span class="text-xs text-slate-400 block uppercase font-bold tracking-wider">TIME LEFT</span>
            <span id="timer-val" class="text-2xl md:text-3xl font-black text-emerald-400">60.0s</span>
        </div>
    </div>

    <!-- The Playground View -->
    <div class="relative flex-1 bg-gradient-to-b from-slate-950 via-slate-900 to-emerald-950/70 overflow-hidden cursor-crosshair">
        
        <!-- Retro Warning Flash Overlay -->
        <div id="danger-overlay" class="absolute inset-0 bg-red-600/20 pointer-events-none opacity-0 transition-opacity duration-150 z-20"></div>
        <div id="stealth-overlay" class="absolute inset-0 bg-emerald-950/40 pointer-events-none opacity-0 transition-opacity duration-200 z-20 flex items-center justify-center">
            <div class="bg-emerald-900/80 border border-emerald-400 text-emerald-300 font-bold px-4 py-2 rounded-full text-sm tracking-widest uppercase">隠れ中 (Hiding Behind Play Equipment)</div>
        </div>

        <!-- Canvas -->
        <canvas id="game-canvas" class="block w-full h-full"></canvas>

        <!-- Dynamic Warning Message Popups -->
        <div id="center-message" class="absolute top-1/3 left-1/2 transform -translate-x-1/2 -translate-y-1/2 text-center pointer-events-none z-30 opacity-0 transition-all duration-300 scale-75">
            <h1 id="center-message-text" class="text-4xl md:text-6xl font-black text-red-500 drop-shadow-[0_4px_12px_rgba(0,0,0,0.8)] leading-tight uppercase tracking-wider"></h1>
            <p id="center-message-sub" class="text-white text-sm md:text-lg font-bold drop-shadow-[0_2px_4px_rgba(0,0,0,0.8)] mt-2"></p>
        </div>
    </div>

    <!-- Bottom Controller Panel for Touch & Desktop -->
    <div class="bg-slate-800 p-4 border-t-4 border-slate-700 flex flex-col sm:flex-row items-center justify-between gap-4 z-10">
        <!-- Desktop Controls Tip -->
        <div class="hidden sm:flex flex-col text-xs text-slate-400">
            <div><kbd class="px-2 py-0.5 bg-slate-700 rounded text-slate-200 font-mono">SPACE</kbd> or <kbd class="px-2 py-0.5 bg-slate-700 rounded text-slate-200 font-mono">Left Click</kbd> : バットを振る (Swing Bat)</div>
            <div class="mt-1"><kbd class="px-2 py-0.5 bg-slate-700 rounded text-slate-200 font-mono">SHIFT</kbd> or <kbd class="px-2 py-0.5 bg-slate-700 rounded text-slate-200 font-mono">Right Click</kbd> (長押し) : 遊具に隠れる (Hide in Play Equipment)</div>
        </div>

        <!-- Large Mobile Action Buttons -->
        <div class="w-full flex justify-around items-center gap-4">
            <!-- Hide Button (Action: Hide) -->
            <button id="btn-hide" class="flex-1 py-4 px-6 bg-gradient-to-b from-emerald-500 to-emerald-600 text-white rounded-2xl font-black text-lg shadow-lg hover:brightness-110 active:scale-95 transition-all retro-btn retro-btn-green flex flex-col items-center justify-center select-none" style="-webkit-tap-highlight-color: transparent;">
                <span class="text-2xl">🫣</span>
                <span>遊具に隠れる！</span>
                <span class="text-[10px] font-normal opacity-80">(長押し/HOLD)</span>
            </button>
            
            <!-- Swing Button (Action: Hit) -->
            <button id="btn-swing" class="flex-1 py-4 px-6 bg-gradient-to-b from-rose-500 to-rose-600 text-white rounded-2xl font-black text-lg shadow-lg hover:brightness-110 active:scale-95 transition-all retro-btn retro-btn-red flex flex-col items-center justify-center select-none" style="-webkit-tap-highlight-color: transparent;">
                <span class="text-2xl">⚾</span>
                <span>バットを振る！</span>
                <span class="text-[10px] font-normal opacity-80">(タップ/TAP)</span>
            </button>
        </div>
    </div>

    <!-- Screen: Main Menu Overlay -->
    <div id="menu-overlay" class="absolute inset-0 bg-slate-950/95 backdrop-blur-sm z-40 flex flex-col items-center justify-center p-6 text-center overflow-y-auto">
        <div class="max-w-md w-full my-auto space-y-6">
            <!-- Game Title with playful Badge -->
            <div class="space-y-2">
                <span class="bg-amber-500 text-slate-950 text-xs font-black uppercase px-3 py-1 rounded-full tracking-wider animate-bounce inline-block">手描きスケッチ「IMG_1915 2.jpeg」完全再現</span>
                <h1 class="text-4xl md:text-5xl font-extrabold tracking-tight leading-none text-transparent bg-clip-text bg-gradient-to-r from-amber-400 via-orange-400 to-rose-500 drop-shadow">
                    窓壊し<br><span class="text-slate-100">バッティング！</span>
                </h1>
            </div>

            <!-- Sketch Reference Notice -->
            <div class="bg-slate-800/80 border border-slate-700 rounded-2xl p-4 text-left text-xs text-slate-300 space-y-2 leading-relaxed">
                <p class="font-bold text-amber-400">📝 ゲームのあらすじ・ルール</p>
                <p>公園の遊具（すべり台）からトスされるボールをバットで打ち、お向かいのマンションの窓をたくさん割ろう！</p>
                <ul class="list-disc list-inside space-y-1 text-slate-400 ml-1">
                    <li>タイミング良く打つとボールが飛んで窓が割れる（高得点！）</li>
                    <li>住民の気配（不穏な足音や警報ゲージ）がしたら、すぐに<strong class="text-emerald-400">「遊具に隠れる」ボタンを長押し</strong>して息を潜めて！</li>
                    <li>住民が現れたときに見つかると、即<strong class="text-rose-400">ゲームオーバー！</strong></li>
                    <li>制限時間60秒以内にどれだけ多く割れるかのチャレンジ！</li>
                </ul>
            </div>

            <!-- Start Button -->
            <button id="btn-start" class="w-full py-5 bg-gradient-to-r from-amber-400 to-amber-500 text-slate-950 font-black text-2xl rounded-2xl shadow-xl hover:scale-105 active:scale-95 transition-all border-b-8 border-amber-700">
                ゲームスタート！ ⚾
            </button>

            <!-- Sound Disclaimer -->
            <p class="text-[10px] text-slate-500">※ ゲーム中、安全でレトロなWebシンセサイザー効果音が自動生成されます。音量にご注意ください。</p>
        </div>
    </div>

    <!-- Screen: Game Over / Game Finished Screen -->
    <div id="result-overlay" class="absolute inset-0 bg-slate-950/95 backdrop-blur-sm z-40 hidden flex flex-col items-center justify-center p-6 text-center">
        <div class="max-w-md w-full space-y-6">
            <div class="space-y-1">
                <span id="result-badge" class="bg-red-600 text-white text-xs font-black uppercase px-3 py-1 rounded-full tracking-wider inline-block">GAME OVER</span>
                <h2 id="result-title" class="text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-red-500 to-rose-400">見つかっちゃった！</h2>
                <p id="result-desc" class="text-slate-400 text-sm">住民が懐中電灯であなたをばっちり照らしました！</p>
            </div>

            <!-- Stats Box -->
            <div class="bg-slate-800/80 border-2 border-slate-700 rounded-3xl p-6 grid grid-cols-2 gap-4">
                <div class="text-center">
                    <span class="text-xs text-slate-400 block uppercase font-bold tracking-wider">割った窓数 (Smashed)</span>
                    <span id="result-windows" class="text-3xl font-black text-rose-400">0 枚</span>
                </div>
                <div class="text-center border-l border-slate-700">
                    <span class="text-xs text-slate-400 block uppercase font-bold tracking-wider">最終スコア (Score)</span>
                    <span id="result-score" class="text-3xl font-black text-amber-400">00000</span>
                </div>
                <div class="col-span-2 text-center border-t border-slate-700 pt-4">
                    <span class="text-xs text-slate-400 block uppercase font-bold tracking-wider">最大コンボ数 (Max Combo)</span>
                    <span id="result-combo" class="text-xl font-bold text-cyan-400">0 combo</span>
                </div>
            </div>

            <!-- Rank Assessment -->
            <div class="bg-slate-900 border border-slate-800 rounded-2xl py-3 px-4">
                <span class="text-xs text-slate-500 block">わんぱく野球少年の評価ランク</span>
                <span id="result-rank" class="text-2xl font-black text-amber-300">「公園のヒゲ魔人」</span>
            </div>

            <!-- Action Buttons -->
            <div class="flex gap-4">
                <button id="btn-restart" class="flex-1 py-4 bg-gradient-to-r from-amber-400 to-amber-500 text-slate-950 font-black text-lg rounded-2xl shadow-xl hover:scale-102 active:scale-98 transition-all border-b-4 border-amber-700">
                    もう一回遊ぶ ⚾
                </button>
            </div>
        </div>
    </div>

</div>

<script>
/**
 * Procedural Audio Synthesizer utilizing Web Audio API
 */
class GameSound {
    constructor() {
        this.ctx = null;
    }

    init() {
        if (!this.ctx) {
            this.ctx = new (window.AudioContext || window.webkitAudioContext)();
        }
    }

    // Play Bat Swing Sound
    playSwing() {
        this.init();
        if (!this.ctx) return;
        const now = this.ctx.currentTime;
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();

        osc.type = 'triangle';
        osc.frequency.setValueAtTime(100, now);
        osc.frequency.exponentialRampToValueAtTime(800, now + 0.15);

        gain.gain.setValueAtTime(0.3, now);
        gain.gain.linearRampToValueAtTime(0.01, now + 0.15);

        osc.connect(gain);
        gain.connect(this.ctx.destination);

        osc.start(now);
        osc.stop(now + 0.15);
    }

    // Play Ball hit bat sound (Sharp crack!)
    playHit() {
        this.init();
        if (!this.ctx) return;
        const now = this.ctx.currentTime;
        
        // Wood hit frequency combo
        const osc1 = this.ctx.createOscillator();
        const osc2 = this.ctx.createOscillator();
        const gain = this.ctx.createGain();

        osc1.type = 'sine';
        osc1.frequency.setValueAtTime(440, now);
        osc1.frequency.exponentialRampToValueAtTime(120, now + 0.1);

        osc2.type = 'triangle';
        osc2.frequency.setValueAtTime(880, now);
        osc2.frequency.exponentialRampToValueAtTime(200, now + 0.08);

        gain.gain.setValueAtTime(0.5, now);
        gain.gain.exponentialRampToValueAtTime(0.01, now + 0.12);

        osc1.connect(gain);
        osc2.connect(gain);
        gain.connect(this.ctx.destination);

        osc1.start(now);
        osc2.start(now);
        osc1.stop(now + 0.15);
        osc2.stop(now + 0.15);
    }

    // Play Glass Break sound (Complex high frequency decay with noise)
    playGlassBreak() {
        this.init();
        if (!this.ctx) return;
        const now = this.ctx.currentTime;

        // Multiple high pitch sine waves to simulate splintering glass shards
        const tones = [1200, 1800, 2400, 3200];
        tones.forEach((freq, idx) => {
            const osc = this.ctx.createOscillator();
            const gain = this.ctx.createGain();
            
            osc.type = 'sine';
            osc.frequency.setValueAtTime(freq, now);
            // Splinter slide
            osc.frequency.linearRampToValueAtTime(freq - (300 * (idx + 1)), now + 0.3 + (idx * 0.1));
            
            gain.gain.setValueAtTime(0.2, now);
            gain.gain.exponentialRampToValueAtTime(0.001, now + 0.25 + (idx * 0.1));

            osc.connect(gain);
            gain.connect(this.ctx.destination);

            osc.start(now);
            osc.stop(now + 0.6);
        });

        // Add some noise burst for crack impact
        const bufferSize = this.ctx.sampleRate * 0.2; // 0.2s noise
        const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
        const data = buffer.getChannelData(0);
        for (let i = 0; i < bufferSize; i++) {
            data[i] = Math.random() * 2 - 1;
        }

        const noise = this.ctx.createBufferSource();
        noise.buffer = buffer;

        const filter = this.ctx.createBiquadFilter();
        filter.type = 'bandpass';
        filter.frequency.value = 4000;

        const noiseGain = this.ctx.createGain();
        noiseGain.gain.setValueAtTime(0.3, now);
        noiseGain.gain.exponentialRampToValueAtTime(0.01, now + 0.15);

        noise.connect(filter);
        filter.connect(noiseGain);
        noiseGain.connect(this.ctx.destination);

        noise.start(now);
        noise.stop(now + 0.2);
    }

    // Play Heartbeat Alarm sound (Resident Coming warning)
    playHeartbeat(rate) {
        this.init();
        if (!this.ctx) return;
        const now = this.ctx.currentTime;
        const osc = this.ctx.createOscillator();
        const gain = this.ctx.createGain();

        osc.type = 'sine';
        osc.frequency.setValueAtTime(70, now);
        osc.frequency.exponentialRampToValueAtTime(30, now + 0.25);

        gain.gain.setValueAtTime(0.6, now);
        gain.gain.exponentialRampToValueAtTime(0.01, now + 0.25);

        osc.connect(gain);
        gain.connect(this.ctx.destination);

        osc.start(now);
        osc.stop(now + 0.25);
    }

    // Play "Spotted" Alarm (Gameover horn)
    playSpotted() {
        this.init();
        if (!this.ctx) return;
        const now = this.ctx.currentTime;
        
        const osc1 = this.ctx.createOscillator();
        const osc2 = this.ctx.createOscillator();
        const gain = this.ctx.createGain();

        osc1.type = 'sawtooth';
        osc1.frequency.value = 180;
        osc2.type = 'sawtooth';
        osc2.frequency.value = 183; // detuned detest look

        gain.gain.setValueAtTime(0.4, now);
        gain.gain.linearRampToValueAtTime(0.01, now + 0.8);

        osc1.connect(gain);
        osc2.connect(gain);
        gain.connect(this.ctx.destination);

        osc1.start(now);
        osc2.start(now);
        osc1.stop(now + 0.8);
        osc2.stop(now + 0.8);
    }

    // Play Score Win sound
    playSuccess() {
        this.init();
        if (!this.ctx) return;
        const now = this.ctx.currentTime;
        const notes = [261.63, 329.63, 392.00, 523.25]; // C4, E4, G4, C5
        
        notes.forEach((freq, idx) => {
            const osc = this.ctx.createOscillator();
            const gain = this.ctx.createGain();

            osc.type = 'sine';
            osc.frequency.value = freq;
            
            gain.gain.setValueAtTime(0.15, now + (idx * 0.1));
            gain.gain.exponentialRampToValueAtTime(0.001, now + (idx * 0.1) + 0.15);

            osc.connect(gain);
            gain.connect(this.ctx.destination);

            osc.start(now + (idx * 0.1));
            osc.stop(now + (idx * 0.1) + 0.2);
        });
    }
}

const sfx = new GameSound();
</script>

<script>
// Main Game Logic Wrapper
const canvas = document.getElementById('game-canvas');
const ctx = canvas.getContext('2d');

// Game state variables
let gameState = {
    running: false,
    score: 0,
    combo: 0,
    maxCombo: 0,
    timeLeft: 60.0,
    smashedWindows: 0,
    isHiding: false,
    isSwinging: false,
    swingCooldown: 0,
    
    // Resident threat status
    threatLevel: 0,     // 0 to 100
    residentState: 'IDLE', // 'IDLE' | 'AWARE' (Warning) | 'SPOTTED' (Looking) | 'COOLDOWN'
    residentTimer: 0,      // countdowns
    residentWait: 180,     // random intervals to trigger next awareness
    residentX: -100,       // on-screen position of resident
    residentAlpha: 0,      // visibility
    residentFlashlightAngle: 0,
    residentAlertLevel: 0, // 0 to 100. If 100, spotted!
};

// Layout sizing and environment configuration
const config = {
    mansionRows: 4,
    mansionCols: 5,
    windowWidth: 44,
    windowHeight: 48,
    windowSpacingX: 18,
    windowSpacingY: 18,
    kidX: 220,
    kidY: 0, // dynamic
    playEquipmentX: 0, // dynamic
    playEquipmentWidth: 150,
};

// Elements array
let windows = [];
let balls = [];
let particles = [];
let indicators = []; // score pops, hits, etc.
let animationFrameId = null;

// Audio heartbeat speed tracker
let lastHeartbeatTime = 0;

// Set up resize handler
function resizeCanvas() {
    const parent = canvas.parentElement;
    canvas.width = parent.clientWidth * window.devicePixelRatio;
    canvas.height = parent.clientHeight * window.devicePixelRatio;
    ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
    
    // Recalculate horizontal offsets based on canvas sizing
    config.kidY = canvas.height / window.devicePixelRatio - 80;
    config.playEquipmentX = canvas.width / window.devicePixelRatio - config.playEquipmentWidth - 30;
    config.kidX = config.playEquipmentX - 120; // standing on the left of play equipment
}

window.addEventListener('resize', resizeCanvas);
resizeCanvas();

// Initialize/Reset windows representing the Mansion facing the play park
function initWindows() {
    windows = [];
    const columns = config.mansionCols;
    const rows = config.mansionRows;
    const blockWidth = columns * (config.windowWidth + config.windowSpacingX) - config.windowSpacingX;
    
    // Centered in top half
    const startX = 60; 
    const startY = 50;

    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < columns; c++) {
            windows.push({
                id: r * columns + c,
                x: startX + c * (config.windowWidth + config.windowSpacingX),
                y: startY + r * (config.windowHeight + config.windowSpacingY),
                w: config.windowWidth,
                h: config.windowHeight,
                state: 'INTACT', // 'INTACT' | 'CRACKED' | 'BROKEN'
                value: (rows - r) * 100, // Higher rows earn more points
                hasLight: Math.random() < 0.3, // Random light inside to show people live there
                decor: Math.random() < 0.4 ? 'curtain' : 'none'
            });
        }
    }
}

// Spark glass particle burst
function spawnGlassBreakEffect(x, y) {
    sfx.playGlassBreak();
    
    // Generate shards
    for (let i = 0; i < 22; i++) {
        particles.push({
            x: x,
            y: y,
            vx: (Math.random() - 0.5) * 8,
            vy: (Math.random() - 0.7) * 8 - 2,
            gravity: 0.18,
            color: `rgba(224, 242, 254, ${0.5 + Math.random() * 0.5})`, // Glass sky-blueish translucent
            size: 2 + Math.random() * 5,
            rotation: Math.random() * Math.PI * 2,
            rotSpeed: (Math.random() - 0.5) * 0.2,
            life: 1.0,
            decay: 0.015 + Math.random() * 0.02
        });
    }

    // Generate score indicator popup
    const basePoints = 150;
    const bonus = gameState.combo > 1 ? gameState.combo * 50 : 0;
    const earned = basePoints + bonus;
    gameState.score += earned;
    gameState.smashedWindows++;
    
    // Update top UI instantly
    document.getElementById('score-val').innerText = String(gameState.score).padStart(5, '0');

    indicators.push({
        x: x,
        y: y - 20,
        text: `+${earned}`,
        color: '#f59e0b', // Amber
        size: 18,
        vy: -1.5,
        alpha: 1.0,
        life: 50
    });

    if (gameState.combo > 1) {
        indicators.push({
            x: x,
            y: y - 38,
            text: `${gameState.combo} COMBO!`,
            color: '#ef4444', // Red
            size: 14,
            vy: -1.2,
            alpha: 1.0,
            life: 45
        });
    }
}

// Spawn new balls being tossed from top of the play slide
function spawnBall() {
    if (!gameState.running || gameState.isHiding) return;

    // すべり台（ゆうぐ）の斜面あたりからふわっとトスされる
    const startX = config.playEquipmentX + 30;
    const startY = config.kidY - 50; 

    // 子供のバットの中心位置（ターゲット）
    const targetX = config.kidX + 15;
    const targetY = config.kidY - 20;

    // 水平速度（左方向へ移動するスピード）
    const vx = -3.5 - (Math.random() * 1.0); 
    const distX = startX - targetX;
    const framesToTarget = distX / Math.abs(vx);

    // 物理公式 (y = y0 + vy*t + 0.5*g*t^2) から、バット位置にちょうど重なる初期垂直速度 vy を逆算
    const g = 0.18; // 重力
    let vy = (targetY - startY - 0.5 * g * framesToTarget * framesToTarget) / framesToTarget;
    
    // 軌道に少しだけ高め・低めのブレを追加してゲーム性を向上
    vy += (Math.random() - 0.5) * 0.5;

    balls.push({
        x: startX,
        y: startY,
        vx: vx,
        vy: vy,
        gravity: g,
        radius: 8, // ボールサイズを大きくして視認しやすく調整
        state: 'TOSSED', // 'TOSSED' | 'HIT' | 'MISSED'
        rotation: 0,
        rotSpeed: -0.1
    });
}

// Player action: Swing Bat
function swingBat() {
    if (!gameState.running || gameState.isHiding || gameState.swingCooldown > 0) return;
    
    gameState.isSwinging = true;
    gameState.swingCooldown = 18; // クールダウンを少し短縮して連続で振りやすく
    sfx.playSwing();

    // Check hit collision with any balls in 'TOSSED' state near the kid
    let hitSomething = false;

    balls.forEach(ball => {
        if (ball.state === 'TOSSED') {
            // 子供の打撃ゾーン（バットのスイートスポット）からの距離
            const distX = Math.abs(ball.x - (config.kidX + 15));
            const distY = Math.abs(ball.y - (config.kidY - 20));

            // 当たり判定を 55px に広げ、タイミングを合わせやすく調整
            if (distX < 55 && distY < 55) {
                // Hit is registered!
                hitSomething = true;
                ball.state = 'HIT';
                sfx.playHit();
                
                // ジャストミート判定（バットの中心に近いほど完璧な打球に）
                const offset = ball.x - (config.kidX + 15);
                let hitQuality = "GOOD!";
                let launchAngle = -Math.PI / 3; // 基本の打ち上げ角度
                let launchSpeed = 11 + Math.random() * 3;

                if (Math.abs(offset) < 15) {
                    hitQuality = "PERFECT!!";
                    launchAngle = -Math.PI / 2.3; // 上の階の窓を狙える鋭い角度
                    launchSpeed = 14 + Math.random() * 2;
                    gameState.combo++;
                    showComboUI(true);
                } else if (offset > 15) {
                    hitQuality = "LATE (HIGH)";
                    launchAngle = -Math.PI / 1.6; // 振り遅れ：天井方向
                    launchSpeed = 9 + Math.random() * 2;
                    gameState.combo = Math.max(1, gameState.combo);
                } else {
                    hitQuality = "EARLY (LOW)";
                    launchAngle = -Math.PI / 4.5; // 早振り：低い弾道
                    launchSpeed = 12 + Math.random() * 3;
                    gameState.combo = Math.max(1, gameState.combo);
                }

                ball.vx = Math.cos(launchAngle) * launchSpeed;
                ball.vy = Math.sin(launchAngle) * launchSpeed;

                // Hit indicator
                indicators.push({
                    x: ball.x,
                    y: ball.y - 25,
                    text: hitQuality,
                    color: hitQuality.includes('PERFECT') ? '#f59e0b' : '#38bdf8',
                    size: 16,
                    vy: -1.5,
                    alpha: 1.0,
                    life: 40
                });

                // Spawn spark particles for hit
                for (let i = 0; i < 10; i++) {
                    particles.push({
                        x: ball.x,
                        y: ball.y,
                        vx: (Math.random() - 0.5) * 8,
                        vy: (Math.random() - 0.5) * 8,
                        gravity: 0.08,
                        color: '#fbbf24',
                        size: 3 + Math.random() * 4,
                        rotation: 0,
                        rotSpeed: 0,
                        life: 1.0,
                        decay: 0.04
                    });
                }
            }
        }
    });

    if (!hitSomething) {
        // Strike / Whiff animation feedback
    }
}

// Show combo overlay and score boosting
function showComboUI(show) {
    const container = document.getElementById('combo-container');
    const val = document.getElementById('combo-val');
    
    if (show && gameState.combo > 1) {
        val.innerText = `x${gameState.combo}`;
        container.classList.remove('opacity-0', 'scale-75');
        container.classList.add('opacity-100', 'scale-100');
        if (gameState.combo > gameState.maxCombo) {
            gameState.maxCombo = gameState.combo;
        }
    } else {
        container.classList.add('opacity-0', 'scale-75');
        container.classList.remove('opacity-100', 'scale-100');
    }
}

// Core gameplay loop: handle AI threat and stealth mechanisms
function updateResidentAI() {
    if (!gameState.running) return;

    // Reduce warning intervals or trigger states
    if (gameState.residentState === 'IDLE') {
        gameState.residentWait--;
        if (gameState.residentWait <= 0) {
            // Initiate AWARE (Danger Incoming Alert)
            gameState.residentState = 'AWARE';
            gameState.residentTimer = 220; // 3-4 seconds warning
            triggerCenterMessage("住民の気配…！", "もうすぐ見回りが来ます！遊具に隠れて！", "#fbbf24");
            
            // Speed up threat bar fill
            document.getElementById('warning-status').innerText = '不審な物音 (DANGER)';
            document.getElementById('warning-status').className = 'text-yellow-500 font-bold shake-element';
            document.getElementById('warning-icon').innerText = '🚨';
        }
    } else if (gameState.residentState === 'AWARE') {
        gameState.residentTimer--;
        
        // Progress Danger bar indicator visually
        gameState.threatLevel = Math.min(100, ((220 - gameState.residentTimer) / 220) * 100);
        document.getElementById('danger-bar').style.width = `${gameState.threatLevel}%`;

        // Pulse threatening heartbeats
        const curTime = Date.now();
        const pulseGap = Math.max(150, gameState.residentTimer * 3.5); // faster as timer drops
        if (curTime - lastHeartbeatTime > pulseGap) {
            sfx.playHeartbeat();
            lastHeartbeatTime = curTime;
            // Flash screen red alert border
            const overlay = document.getElementById('danger-overlay');
            overlay.style.opacity = '0.4';
            setTimeout(() => { overlay.style.opacity = '0'; }, 80);
        }

        if (gameState.residentTimer <= 0) {
            // Change to SPOTTED scanning mode
            gameState.residentState = 'SPOTTED';
            gameState.residentTimer = 200; // time window to remain hidden
            gameState.residentAlertLevel = 0;
            triggerCenterMessage("住民現る！！", "動くな！見回りのライトが来ている！", "#ef4444");
            
            document.getElementById('warning-status').innerText = '見回り中！ (SCANNING)';
            document.getElementById('warning-status').className = 'text-red-500 font-black animate-pulse';
            document.getElementById('warning-icon').innerText = '🤬';
        }
    } else if (gameState.residentState === 'SPOTTED') {
        gameState.residentTimer--;
        
        // Resident appears on left margin and sweeps high-tech flashlight over the park
        gameState.residentX = Math.min(30, gameState.residentX + 3.5);
        gameState.residentAlpha = Math.min(1.0, gameState.residentAlpha + 0.1);
        
        // Flashlight sweep angles back and forth (radians around 0 degrees heading right)
        gameState.residentFlashlightAngle = 0.2 * Math.sin(gameState.residentTimer * 0.08);

        // Spot checking mechanics
        if (!gameState.isHiding) {
            // Player standing in playground is highly exposed! Alert bar fills extremely fast
            gameState.residentAlertLevel = Math.min(100, gameState.residentAlertLevel + 3.0);
            
            // Alert warning audio warning pitch
            const overlay = document.getElementById('danger-overlay');
            overlay.style.opacity = (gameState.residentAlertLevel / 100) * 0.7;
            
            if (gameState.residentAlertLevel >= 100) {
                // Game Over - Caught red handed!
                triggerGameOver(false);
                return;
            }
        } else {
            // Safe behind playground slide equipment! Decay alert level if any was accumulated
            gameState.residentAlertLevel = Math.max(0, gameState.residentAlertLevel - 1.5);
            document.getElementById('danger-overlay').style.opacity = '0';
        }

        if (gameState.residentTimer <= 0) {
            // Successfully avoided! Resident walks away
            gameState.residentState = 'COOLDOWN';
            gameState.residentTimer = 90; // Walking out animation
            triggerCenterMessage("ふぅ…去っていった", "今のうちに窓を割ろう！", "#10b981");
        }
    } else if (gameState.residentState === 'COOLDOWN') {
        gameState.residentTimer--;
        // Resident walks left off-screen
        gameState.residentX -= 4.0;
        gameState.residentAlpha = Math.max(0, gameState.residentAlpha - 0.08);

        // Drain Danger gauge back to zero
        gameState.threatLevel = Math.max(0, gameState.threatLevel - 2.5);
        document.getElementById('danger-bar').style.width = `${gameState.threatLevel}%`;

        if (gameState.residentTimer <= 0) {
            gameState.residentState = 'IDLE';
            gameState.residentWait = 350 + Math.random() * 400; // Next check randomly timed
            document.getElementById('warning-status').innerText = '安全 (SAFE)';
            document.getElementById('warning-status').className = 'text-emerald-400';
            document.getElementById('warning-icon').innerText = '🤫';
            // Drop combo if we broke flow, or keep it friendly? Let's keep it to motivate stealth
        }
    }
}

// Show flashy warnings or feedback in center of layout
function triggerCenterMessage(mainText, subText, colorHex) {
    const el = document.getElementById('center-message');
    const head = document.getElementById('center-message-text');
    const sub = document.getElementById('center-message-sub');

    head.innerText = mainText;
    head.style.color = colorHex;
    sub.innerText = subText;

    // Trigger visual fade-in slide
    el.classList.remove('opacity-0', 'scale-75');
    el.classList.add('opacity-100', 'scale-100');

    // Auto fadeout after 2.5s
    setTimeout(() => {
        el.classList.add('opacity-0', 'scale-75');
        el.classList.remove('opacity-100', 'scale-100');
    }, 2800);
}

function updatePhysics() {
    // 1. Balls physics
    for (let i = balls.length - 1; i >= 0; i--) {
        const ball = balls[i];
        ball.vy += ball.gravity;
        ball.x += ball.vx;
        ball.y += ball.vy;
        ball.rotation += ball.rotSpeed;

        // Ball out of bounds check
        if (ball.y > canvas.height / window.devicePixelRatio + 100 || ball.x < -100 || ball.x > canvas.width / window.devicePixelRatio + 100) {
            // If ball missed the swing zone
            if (ball.state === 'TOSSED') {
                gameState.combo = 0; // reset combo on missed ball
                showComboUI(false);
            }
            balls.splice(i, 1);
            continue;
        }

        // Hit ball check window impact collisions
        if (ball.state === 'HIT') {
            windows.forEach(win => {
                if (win.state !== 'BROKEN') {
                    // Check bounding box intersection with ball
                    if (ball.x + ball.radius > win.x && 
                        ball.x - ball.radius < win.x + win.w &&
                        ball.y + ball.radius > win.y && 
                        ball.y - ball.radius < win.y + win.h) {
                        
                        // Break glass state transitions
                        if (win.state === 'INTACT') {
                            win.state = 'BROKEN';
                            spawnGlassBreakEffect(win.x + win.w/2, win.y + win.h/2);
                            // Bounce ball off slightly or deflect away
                            ball.vx = -ball.vx * 0.4;
                            ball.vy = -ball.vy * 0.4 + 2;
                            ball.state = 'MISSED'; // change state so it doesn't break more windows
                        }
                    }
                }
            });
        }
    }

    // 2. Particle decay physics
    for (let i = particles.length - 1; i >= 0; i--) {
        const p = particles[i];
        p.vy += p.gravity;
        p.x += p.vx;
        p.y += p.vy;
        p.rotation += p.rotSpeed;
        p.life -= p.decay;

        if (p.life <= 0) {
            particles.splice(i, 1);
        }
    }

    // 3. Floating scores indicator movement
    for (let i = indicators.length - 1; i >= 0; i--) {
        const ind = indicators[i];
        ind.y += ind.vy;
        ind.life--;
        if (ind.life <= 0) {
            indicators.splice(i, 1);
        }
    }

    // Swing cooldown reduction
    if (gameState.swingCooldown > 0) {
        gameState.swingCooldown--;
        if (gameState.swingCooldown <= 0) {
            gameState.isSwinging = false;
        }
    }
}

// Redraw all items on HTML5 canvas frame
function draw() {
    ctx.clearRect(0, 0, canvas.width / window.devicePixelRatio, canvas.height / window.devicePixelRatio);

    // Dynamic scale helper
    const W = canvas.width / window.devicePixelRatio;
    const H = canvas.height / window.devicePixelRatio;

    // 1. Draw Mansion wall background
    ctx.fillStyle = '#1e293b'; // Slate dark building wall
    ctx.fillRect(20, 30, W - 40, H - 180);

    // Mansion trim & roof line
    ctx.fillStyle = '#0f172a';
    ctx.fillRect(10, 15, W - 20, 15); // Roof line
    ctx.fillStyle = '#475569';
    ctx.fillRect(20, 30, W - 40, 5); // top trim shadow

    // 2. Draw Windows
    windows.forEach(win => {
        // Draw Window background/curtains/lights
        if (win.state === 'BROKEN') {
            ctx.fillStyle = '#020617'; // Totally dark broken window interior
            ctx.fillRect(win.x, win.y, win.w, win.h);
            
            // Draw remaining sharp glass shards outline
            ctx.strokeStyle = 'rgba(14, 165, 233, 0.5)';
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.moveTo(win.x, win.y);
            ctx.lineTo(win.x + 8, win.y);
            ctx.lineTo(win.x, win.y + 12);
            ctx.closePath();
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(win.x + win.w, win.y + win.h);
            ctx.lineTo(win.x + win.w - 12, win.y + win.h);
            ctx.lineTo(win.x + win.w, win.y + win.h - 10);
            ctx.closePath();
            ctx.stroke();
        } else {
            // Intact window with light variation
            ctx.fillStyle = win.hasLight ? '#fef08a' : '#1e293b'; // soft yellow glow or dark blueish glass reflection
            if (!win.hasLight) {
                ctx.fillStyle = 'rgba(30, 41, 59, 0.9)';
            }
            ctx.fillRect(win.x, win.y, win.w, win.h);

            // Draw curtain decoration if any
            if (win.decor === 'curtain') {
                ctx.fillStyle = win.hasLight ? '#fca5a5' : '#94a3b8';
                // Left drape
                ctx.beginPath();
                ctx.moveTo(win.x, win.y);
                ctx.lineTo(win.x + 12, win.y);
                ctx.quadraticCurveTo(win.x + 4, win.y + win.h/2, win.x, win.y + win.h);
                ctx.closePath();
                ctx.fill();
                // Right drape
                ctx.beginPath();
                ctx.moveTo(win.x + win.w, win.y);
                ctx.lineTo(win.x + win.w - 12, win.y);
                ctx.quadraticCurveTo(win.x + win.w - 4, win.y + win.h/2, win.x + win.w, win.y + win.h);
                ctx.closePath();
                ctx.fill();
            }

            // Window Glass reflections (diagonal neon light lines)
            ctx.strokeStyle = win.hasLight ? 'rgba(255, 255, 255, 0.5)' : 'rgba(255, 255, 255, 0.15)';
            ctx.lineWidth = 1;
            ctx.beginPath();
            ctx.moveTo(win.x + 5, win.y + win.h - 5);
            ctx.lineTo(win.x + win.w - 5, win.y + 5);
            ctx.stroke();

            // Window frame lines (Classic grid style)
            ctx.strokeStyle = '#64748b'; // Slate borders
            ctx.lineWidth = 3;
            ctx.strokeRect(win.x, win.y, win.w, win.h);

            // Center window cross dividers
            ctx.strokeStyle = '#64748b';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(win.x + win.w / 2, win.y);
            ctx.lineTo(win.x + win.w / 2, win.y + win.h);
            ctx.moveTo(win.x, win.y + win.h / 2);
            ctx.lineTo(win.x + win.w, win.y + win.h / 2);
            ctx.stroke();
        }
    });

    // 3. Draw Playground Ground/Gravel surface
    ctx.fillStyle = '#064e3b'; // Deep park grass color
    ctx.fillRect(0, H - 85, W, 85);
    ctx.fillStyle = '#0f766e'; // gravel trail outline
    ctx.fillRect(0, H - 40, W, 40);

    // 4. Draw Park Play Equipment (The "Yuugu" slide matching sketch details)
    const pX = config.playEquipmentX;
    const pY = config.kidY + 20;

    // Slide supports
    ctx.strokeStyle = '#475569';
    ctx.lineWidth = 5;
    ctx.beginPath();
    ctx.moveTo(pX + 110, pY - 80);
    ctx.lineTo(pX + 110, pY + 20); // support leg
    ctx.moveTo(pX + 140, pY - 30);
    ctx.lineTo(pX + 140, pY + 20); // stairs support
    ctx.stroke();

    // The Slide slope / hump (sketch: "ゆうぐ" curve mound)
    ctx.fillStyle = '#3b82f6'; // Playful blue slide body
    ctx.beginPath();
    ctx.moveTo(pX + 10, pY + 20); // Slide bottom landing
    ctx.quadraticCurveTo(pX + 30, pY + 15, pX + 50, pY - 20);
    ctx.quadraticCurveTo(pX + 75, pY - 70, pX + 110, pY - 80); // top hump
    ctx.lineTo(pX + 125, pY - 80);
    ctx.quadraticCurveTo(pX + 90, pY - 60, pX + 65, pY - 10);
    ctx.quadraticCurveTo(pX + 45, pY + 20, pX + 10, pY + 20);
    ctx.closePath();
    ctx.fill();

    // Slide side rails (Yellow rails)
    ctx.strokeStyle = '#eab308';
    ctx.lineWidth = 6;
    ctx.beginPath();
    ctx.moveTo(pX + 10, pY + 16);
    ctx.quadraticCurveTo(pX + 30, pY + 11, pX + 50, pY - 24);
    ctx.quadraticCurveTo(pX + 75, pY - 74, pX + 110, pY - 84);
    ctx.stroke();

    // Slide Stairs (Right side steps)
    ctx.fillStyle = '#94a3b8';
    for (let s = 0; s < 5; s++) {
        ctx.fillRect(pX + 115 + (s * 6), pY - 80 + (s * 20), 12, 6);
    }

    // Label play equipment like hand drawn layout
    ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
    ctx.font = 'bold 11px sans-serif';
    ctx.fillText('ゆうぐ (SLIDE)', pX + 50, pY + 30);

    // 5. Draw Kid (Baseball Player) with Bat
    if (!gameState.isHiding) {
        const kX = config.kidX;
        const kY = config.kidY;

        // Draw Player Shadow
        ctx.fillStyle = 'rgba(15, 23, 42, 0.4)';
        ctx.beginPath();
        ctx.ellipse(kX + 5, kY + 20, 20, 6, 0, 0, Math.PI * 2);
        ctx.fill();

        // Kid stick-figure style with adorable proportions like sketch
        ctx.strokeStyle = '#f8fafc';
        ctx.lineWidth = 4;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        // Body trunk
        ctx.beginPath();
        ctx.moveTo(kX, kY - 30);
        ctx.lineTo(kX, kY); // torso
        ctx.stroke();

        // Legs
        ctx.beginPath();
        ctx.moveTo(kX, kY);
        ctx.lineTo(kX - 12, kY + 20); // left leg
        ctx.moveTo(kX, kY);
        ctx.lineTo(kX + 12, kY + 20); // right leg
        ctx.stroke();

        // Arms holding the baseball bat
        ctx.beginPath();
        if (gameState.isSwinging) {
            // Forward swings pose
            ctx.moveTo(kX, kY - 20);
            ctx.lineTo(kX + 22, kY - 15); // front arm extension
            ctx.lineTo(kX + 30, kY - 10);
        } else {
            // Ready batting stance pose
            ctx.moveTo(kX, kY - 20);
            ctx.lineTo(kX - 15, kY - 25); // back hand
            ctx.moveTo(kX, kY - 20);
            ctx.lineTo(kX + 5, kY - 18);  // front hand grip
        }
        ctx.stroke();

        // Kid Head
        ctx.fillStyle = '#f8fafc';
        ctx.beginPath();
        ctx.arc(kX, kY - 42, 14, 0, Math.PI * 2);
        ctx.fill();

        // Face features (Cute smile)
        ctx.strokeStyle = '#0f172a';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(kX + 4, kY - 41, 4, 0, Math.PI); // happy smiling mouth
        ctx.stroke();
        // Dot eyes
        ctx.fillStyle = '#0f172a';
        ctx.beginPath();
        ctx.arc(kX + 2, kY - 46, 1.8, 0, Math.PI * 2);
        ctx.arc(kX + 8, kY - 46, 1.8, 0, Math.PI * 2);
        ctx.fill();

        // Cap/Hat
        ctx.fillStyle = '#ef4444'; // Red cap
        ctx.beginPath();
        ctx.arc(kX, kY - 48, 14, Math.PI, Math.PI * 2); // cap dome
        ctx.fill();
        ctx.fillStyle = '#b91c1c'; // brim visor pointing forward
        ctx.fillRect(kX, kY - 51, 18, 4);

        // Baseball Bat (Sketch: "バット")
        ctx.save();
        ctx.translate(kX, kY - 20);
        if (gameState.isSwinging) {
            // Swing arc rotaion forward
            ctx.rotate(Math.PI / 4 + (gameState.swingCooldown * -0.05));
            // Bat body
            ctx.fillStyle = '#f59e0b'; // Wooden bat gold
            ctx.fillRect(15, -6, 32, 7); // barrel
            ctx.fillStyle = '#d97706'; // handle grip
            ctx.fillRect(0, -4, 15, 4);
        } else {
            // Idle stance bat resting over shoulder
            ctx.rotate(-Math.PI / 3.5);
            ctx.fillStyle = '#f59e0b';
            ctx.fillRect(-8, -48, 7, 34); // barrel pointing up
            ctx.fillStyle = '#d97706';
            ctx.fillRect(-6, -14, 4, 14); // handle grip
        }
        ctx.restore();

        // Label Kid
        ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
        ctx.font = 'bold 11px sans-serif';
        ctx.fillText('子供 (KID)', kX - 25, kY - 65);
    } else {
        // Player is successfully hiding behind play slide
        // Draw little peeking eyes on the edge of the slide to indicate status
        const kX = config.playEquipmentX + 25;
        const kY = config.kidY - 15;
        
        ctx.fillStyle = '#f8fafc';
        ctx.beginPath();
        ctx.arc(kX, kY, 6, 0, Math.PI * 2);
        ctx.arc(kX + 10, kY, 6, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = '#0f172a';
        ctx.beginPath();
        ctx.arc(kX, kY, 2, 0, Math.PI * 2);
        ctx.arc(kX + 10, kY, 2, 0, Math.PI * 2);
        ctx.fill();
    }

    // 6. Draw Angry Resident searching for culprit with Flashlight (Sketch: "住民")
    if (gameState.residentAlpha > 0) {
        ctx.save();
        ctx.globalAlpha = gameState.residentAlpha;

        const rx = gameState.residentX;
        const ry = H - 110;

        // Shadow
        ctx.fillStyle = 'rgba(15, 23, 42, 0.5)';
        ctx.beginPath();
        ctx.ellipse(rx, ry + 30, 16, 5, 0, 0, Math.PI * 2);
        ctx.fill();

        // Resident stick figure body
        ctx.strokeStyle = '#ef4444'; // Red outline of angry alert state
        ctx.lineWidth = 4;
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';
        
        ctx.beginPath();
        ctx.moveTo(rx, ry - 10);
        ctx.lineTo(rx, ry + 15); // spine
        ctx.stroke();

        // Legs walking
        ctx.beginPath();
        const walkCycle = Math.sin(gameState.residentTimer * 0.15) * 10;
        ctx.moveTo(rx, ry + 15);
        ctx.lineTo(rx - walkCycle, ry + 32);
        ctx.moveTo(rx, ry + 15);
        ctx.lineTo(rx + walkCycle, ry + 32);
        ctx.stroke();

        // Head
        ctx.fillStyle = '#ef4444';
        ctx.beginPath();
        ctx.arc(rx, ry - 22, 12, 0, Math.PI * 2);
        ctx.fill();

        // Angy cross eyebrows on face
        ctx.strokeStyle = '#ffffff';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(rx - 6, ry - 25);
        ctx.lineTo(rx - 1, ry - 22);
        ctx.moveTo(rx + 6, ry - 25);
        ctx.lineTo(rx + 1, ry - 22);
        ctx.stroke();

        // Flashlight beam sweeping around
        const beamLength = W - rx - 80;
        const targetY = ry + 20 + Math.sin(gameState.residentFlashlightAngle) * 60;
        
        // Spotlight cone gradient fill
        const grad = ctx.createRadialGradient(rx, ry, 10, rx, ry, beamLength);
        grad.addColorStop(0, 'rgba(253, 224, 71, 0.8)'); // Bright yellow at tip
        grad.addColorStop(0.3, 'rgba(253, 224, 71, 0.3)');
        grad.addColorStop(1, 'rgba(253, 224, 71, 0)'); // Fades completely
        
        ctx.fillStyle = grad;
        ctx.beginPath();
        ctx.moveTo(rx, ry);
        // wide searchlight sweep cone bound points
        ctx.lineTo(W - 40, targetY - 70);
        ctx.lineTo(W - 40, targetY + 70);
        ctx.closePath();
        ctx.fill();

        // Flashlight beam outline
        ctx.strokeStyle = 'rgba(253, 224, 71, 0.3)';
        ctx.lineWidth = 1;
        ctx.stroke();

        // Display Warning Speech bubble "誰だ！" or "コラァ！"
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(rx - 25, ry - 65, 75, 24);
        ctx.beginPath();
        ctx.moveTo(rx + 5, ry - 41);
        ctx.lineTo(rx, ry - 35);
        ctx.lineTo(rx - 5, ry - 41);
        ctx.closePath();
        ctx.fill();

        ctx.fillStyle = '#000000';
        ctx.font = 'bold 11px sans-serif';
        ctx.fillText('コラァッ！💢', rx - 18, ry - 50);

        ctx.restore();
    }

    // 7. Draw Balls (Sketch: "ボール")
    balls.forEach(ball => {
        ctx.save();
        ctx.translate(ball.x, ball.y);
        ctx.rotate(ball.rotation);

        // Ball Shadow
        ctx.fillStyle = 'rgba(15, 23, 42, 0.3)';
        ctx.beginPath();
        ctx.arc(2, 6, ball.radius, 0, Math.PI * 2);
        ctx.fill();

        // White baseball body with red seams lines
        ctx.fillStyle = '#f8fafc';
        ctx.beginPath();
        ctx.arc(0, 0, ball.radius, 0, Math.PI * 2);
        ctx.fill();

        // Seams
        ctx.strokeStyle = '#ef4444';
        ctx.lineWidth = 1;
        ctx.beginPath();
        ctx.arc(-ball.radius, 0, ball.radius * 1.1, -0.6, 0.6);
        ctx.stroke();
        ctx.beginPath();
        ctx.arc(ball.radius, 0, ball.radius * 1.1, Math.PI - 0.6, Math.PI + 0.6);
        ctx.stroke();

        ctx.restore();

        // Draw helper hitting spot ring if ball is tossed near kid
        if (ball.state === 'TOSSED' && ball.x < config.kidX + 90 && ball.x > config.kidX - 40) {
            ctx.strokeStyle = 'rgba(251, 191, 36, 0.35)'; // gold timing aid
            ctx.lineWidth = 2.5;
            ctx.setLineDash([4, 4]);
            ctx.beginPath();
            ctx.arc(config.kidX + 15, config.kidY - 20, 26, 0, Math.PI * 2);
            ctx.stroke();
            ctx.setLineDash([]);
        }
    });

    // 8. Draw Glass Break Shards / Spark Particles
    particles.forEach(p => {
        ctx.save();
        ctx.translate(p.x, p.y);
        ctx.rotate(p.rotation);
        ctx.globalAlpha = p.life;
        ctx.fillStyle = p.color;
        
        // Draw triangular glass splinter shapes
        ctx.beginPath();
        ctx.moveTo(0, -p.size);
        ctx.lineTo(p.size/2, p.size);
        ctx.lineTo(-p.size/2, p.size);
        ctx.closePath();
        ctx.fill();
        ctx.restore();
    });

    // 9. Draw floating score indicators or text tags
    indicators.forEach(ind => {
        ctx.save();
        ctx.globalAlpha = ind.alpha;
        ctx.fillStyle = ind.color;
        ctx.font = `black ${ind.size}px 'Mochiy Pop One', sans-serif`;
        ctx.textAlign = 'center';
        ctx.fillText(ind.text, ind.x, ind.y);
        ctx.restore();
    });
}

// Primary animation cycle tick
function tick() {
    if (!gameState.running) return;

    // Timer management
    gameState.timeLeft -= 1 / 60; // assume constant 60 fps
    if (gameState.timeLeft <= 0) {
        gameState.timeLeft = 0;
        triggerGameOver(true); // Win by timeout surviving!
        return;
    }

    // Update displays
    document.getElementById('timer-val').innerText = `${gameState.timeLeft.toFixed(1)}s`;

    // Process gameplay ticks
    updateResidentAI();
    updatePhysics();
    draw();

    // Spawn ball toss occasionally if playing
    if (Math.random() < 0.013 && balls.filter(b => b.state === 'TOSSED').length === 0 && gameState.residentState !== 'SPOTTED') {
        spawnBall();
    }

    animationFrameId = requestAnimationFrame(tick);
}

// Initialize and Start Game Session
function startGame() {
    // Hide UI menus
    document.getElementById('menu-overlay').classList.add('hidden');
    document.getElementById('result-overlay').classList.add('hidden');

    // Reset scores & threat statuses
    gameState.score = 0;
    gameState.combo = 0;
    gameState.maxCombo = 0;
    gameState.timeLeft = 60.0;
    gameState.smashedWindows = 0;
    gameState.isHiding = false;
    gameState.isSwinging = false;
    gameState.swingCooldown = 0;
    gameState.threatLevel = 0;
    gameState.residentState = 'IDLE';
    gameState.residentWait = 180; // short first wait to speed up start action
    gameState.residentX = -120;
    gameState.residentAlpha = 0;
    gameState.residentFlashlightAngle = 0;
    gameState.residentAlertLevel = 0;

    // Reset components arrays
    balls = [];
    particles = [];
    indicators = [];

    // Reset layouts
    resizeCanvas();
    initWindows();

    // Trigger HUD redraws
    document.getElementById('score-val').innerText = '00000';
    document.getElementById('danger-bar').style.width = '0%';
    document.getElementById('warning-status').innerText = '安全 (SAFE)';
    document.getElementById('warning-status').className = 'text-emerald-400';
    document.getElementById('warning-icon').innerText = '🤫';
    document.getElementById('stealth-overlay').style.opacity = '0';
    document.getElementById('danger-overlay').style.opacity = '0';
    showComboUI(false);

    // BGM & Audio context prep
    sfx.init();

    // Set active running
    gameState.running = true;
    tick();

    // Initial message splash
    triggerCenterMessage("バッティング開始！", "飛んでくるボールをカキーンと打とう！", "#fbbf24");
    setTimeout(spawnBall, 1500); // toss first ball soon
}

// End game trigger
function triggerGameOver(isCompletedTime) {
    gameState.running = false;
    if (animationFrameId) {
        cancelAnimationFrame(animationFrameId);
    }

    // Sound effect
    if (isCompletedTime) {
        sfx.playSuccess();
    } else {
        sfx.playSpotted();
    }

    // Populate screens elements
    const badge = document.getElementById('result-badge');
    const title = document.getElementById('result-title');
    const desc = document.getElementById('result-desc');
    const rank = document.getElementById('result-rank');

    if (isCompletedTime) {
        badge.innerText = 'TIME UP CLEAR!';
        badge.className = 'bg-emerald-500 text-slate-900 text-xs font-black uppercase px-3 py-1 rounded-full tracking-wider inline-block animate-bounce';
        title.innerText = '時間切れ！大逃げ成功！';
        title.className = 'text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-emerald-400 to-amber-300';
        desc.innerText = '無事に捕まらずにイタズラを完了しました！';
    } else {
        badge.innerText = 'GAME OVER';
        badge.className = 'bg-red-600 text-white text-xs font-black uppercase px-3 py-1 rounded-full tracking-wider inline-block';
        title.innerText = '見つかっちゃった！';
        title.className = 'text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-red-500 to-rose-400';
        desc.innerText = '住民の懐中電灯に見つかり、激しく怒られてしまいました。';
    }

    document.getElementById('result-windows').innerText = `${gameState.smashedWindows} 枚`;
    document.getElementById('result-score').innerText = String(gameState.score).padStart(5, '0');
    document.getElementById('result-combo').innerText = `${gameState.maxCombo} combo`;

    // Dynamic humorous titles based on final smash score achievements
    let scoreRank = "公園の置き物";
    if (gameState.score >= 5000) scoreRank = "スーパープロ野球少年";
    else if (gameState.score >= 3000) scoreRank = "窓ガラスの破壊魔人";
    else if (gameState.score >= 1500) scoreRank = "わんぱく悪ガキ";
    else if (gameState.score >= 500) scoreRank = "見習いバッター";
    
    rank.innerText = `「${scoreRank}」`;

    // Reveal Result screen
    document.getElementById('result-overlay').classList.remove('hidden');
}

// Interactive triggers setup
const btnSwing = document.getElementById('btn-swing');
const btnHide = document.getElementById('btn-hide');
const btnStart = document.getElementById('btn-start');
const btnRestart = document.getElementById('btn-restart');

// Action 1: SWING
function handleSwingStart(e) {
    if (e) e.preventDefault();
    swingBat();
}
btnSwing.addEventListener('touchstart', handleSwingStart, { passive: false });
btnSwing.addEventListener('mousedown', handleSwingStart);

// Action 2: HIDE STEALTH (Hold to remain hiding behind slide)
function setHidingState(isHiding) {
    if (!gameState.running) return;
    gameState.isHiding = isHiding;
    
    const overlay = document.getElementById('stealth-overlay');
    if (isHiding) {
        overlay.style.opacity = '1';
        // Hide existing tossed balls so they can't strike us easily, and prevent hit combos
        balls = balls.filter(b => b.state !== 'TOSSED');
    } else {
        overlay.style.opacity = '0';
    }
}

// Touch/Click triggers for Hide
btnHide.addEventListener('touchstart', (e) => {
    e.preventDefault();
    setHidingState(true);
}, { passive: false });

btnHide.addEventListener('touchend', (e) => {
    e.preventDefault();
    setHidingState(false);
}, { passive: false });

btnHide.addEventListener('mousedown', () => setHidingState(true));
btnHide.addEventListener('mouseup', () => setHidingState(false));
btnHide.addEventListener('mouseleave', () => setHidingState(false));

// Keyboard Listeners (Space = swing, Shift = Hide)
window.addEventListener('keydown', (e) => {
    if (!gameState.running) return;
    if (e.code === 'Space') {
        e.preventDefault();
        swingBat();
    }
    if (e.code === 'ShiftLeft' || e.code === 'ShiftRight') {
        e.preventDefault();
        setHidingState(true);
    }
});

window.addEventListener('keyup', (e) => {
    if (e.code === 'ShiftLeft' || e.code === 'ShiftRight') {
        e.preventDefault();
        setHidingState(false);
    }
});

// Avoid canvas right click contexts if utilizing desktop clicks
canvas.addEventListener('contextmenu', (e) => {
    e.preventDefault();
});

// Click on canvas to swing easily too
canvas.addEventListener('mousedown', (e) => {
    if (e.button === 0) { // left click
        swingBat();
    } else if (e.button === 2) { // right click
        setHidingState(true);
    }
});

canvas.addEventListener('mouseup', (e) => {
    if (e.button === 2) {
        setHidingState(false);
    }
});

// Menu button click mappings
btnStart.addEventListener('click', startGame);
btnRestart.addEventListener('click', startGame);

// Initialize visual scene ready at start
initWindows();
resizeCanvas();
draw();
</script>
</body>
</html>
