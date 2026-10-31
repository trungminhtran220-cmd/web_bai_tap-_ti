<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Tarot AR — HTML Quiz</title>
<style>
    body { margin: 0; overflow: hidden; background-color: #050505; font-family: 'Times New Roman', serif; color: #fff; }

    #input-video {
        position: fixed; top: 0; left: 0;
        width: 100%; height: 100%; object-fit: cover;
        opacity: 0; z-index: -10; pointer-events: none;
    }

    #canvas-container { width: 100vw; height: 100vh; position: absolute; top: 0; left: 0; z-index: 1; }
    #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 2; pointer-events: none; }

    #guide-text {
        position: absolute; width: 100%; text-align: center;
        color: #fff; letter-spacing: 3px; font-weight: normal;
        text-shadow: 0 0 10px rgba(255, 255, 255, 0.5); pointer-events: none;
        transition: opacity 0.5s; text-transform: uppercase;
        font-size: 0.6rem; bottom: 3%;
    }

    #counter {
        position: absolute; top: 30px; right: 40px;
        font-size: 1.2rem; color: #d4af37; font-weight: bold;
        border: 1px solid rgba(212, 175, 55, 0.6); padding: 5px 15px; border-radius: 30px;
        background: rgba(0, 0, 0, 0.4); backdrop-filter: blur(5px);
    }

    #credit-text {
        position: absolute;
        color: rgba(255, 255, 255, 0.6); font-family: sans-serif; font-size: 0.8rem;
        pointer-events: none; z-index: 5; letter-spacing: 1px;
        text-shadow: 0 0 5px rgba(0,0,0,0.5);
        bottom: 10px; right: 5px;
    }

    #loading {
        position: absolute; top: 0; left: 0; width: 100%; height: 100%;
        background: #000; z-index: 10; display: flex; flex-direction: column;
        justify-content: center; align-items: center; text-align: center;
        transition: opacity 1.5s ease-out; pointer-events: auto;
    }
    .loader-content { opacity: 0.8; animation: pulse 2s infinite; }
    @keyframes pulse { 0% { opacity: 0.5; } 50% { opacity: 1; } 100% { opacity: 0.5; } }

    #virtual-cursor {
        position: absolute; width: 30px; height: 30px;
        border: 2px solid rgba(255, 255, 255, 0.9); border-radius: 50%;
        transform: translate(-50%, -50%); pointer-events: none;
        z-index: 100; display: none;
        box-shadow: 0 0 10px rgba(255,255,255,0.6);
        transition: background 0.15s, width 0.15s, height 0.15s;
    }
    #virtual-cursor.pinching {
        background: #fff; width: 15px; height: 15px; border: none;
        box-shadow: 0 0 20px #fff, 0 0 40px #fff;
    }

    /* ===== MODAL ===== */
    #result-modal {
        position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
        width: 80%; max-width: 900px; max-height: 85vh; overflow-y: auto;
        background: rgba(8, 8, 8, 0.97); border: 1px solid #d4af37;
        padding: 40px 45px; display: none; pointer-events: auto;
        box-shadow: 0 0 100px rgba(212,175,55,0.15); border-radius: 4px; z-index: 20;
        box-sizing: border-box;
    }
    #result-modal::-webkit-scrollbar { width: 6px; }
    #result-modal::-webkit-scrollbar-thumb { background: #444; border-radius: 3px; }

    #result-modal h2 {
        color: #fff; text-align: center; border-bottom: 1px solid #333;
        padding-bottom: 20px; margin-top: 0; text-transform: uppercase;
        letter-spacing: 4px; font-weight: normal; font-size: 1.6rem;
    }

    /* ===== COUNTDOWN BANNER ===== */
    #countdown-banner {
        display: none;
        text-align: center;
        padding: 14px 20px;
        margin-bottom: 25px;
        background: rgba(212, 175, 55, 0.08);
        border: 1px solid rgba(212, 175, 55, 0.35);
        border-radius: 4px;
        color: #d4af37;
        font-size: 1rem;
        letter-spacing: 1px;
        animation: pulseGold 2s infinite;
        transition: all 0.4s;
    }
    #countdown-time {
        font-size: 1.3rem;
        font-weight: bold;
        color: #fff;
        margin: 0 6px;
        font-variant-numeric: tabular-nums;
    }
    @keyframes pulseGold {
        0%, 100% { box-shadow: 0 0 10px rgba(212,175,55,0.15); }
        50%      { box-shadow: 0 0 25px rgba(212,175,55,0.35); }
    }
    #skip-btn {
        display: inline-block;
        margin-left: 14px;
        padding: 5px 14px;
        background: transparent;
        border: 1px solid rgba(212, 175, 55, 0.6);
        color: #d4af37;
        border-radius: 20px;
        cursor: pointer;
        font-family: 'Times New Roman', serif;
        font-size: 0.75rem;
        letter-spacing: 1px;
        transition: all 0.3s;
    }
    #skip-btn:hover {
        background: rgba(212, 175, 55, 0.15);
        border-color: #d4af37;
    }

    /* ===== READING SECTIONS ===== */
    .reading-section {
        margin-bottom: 35px;
        border-bottom: 1px solid #1a1a1a;
        padding-bottom: 25px;
        animation: fadeIn 1s ease-out;
    }
    .reading-section:last-of-type { border-bottom: none; }

    .reading-label {
        color: #888;
        font-size: 0.7rem;
        letter-spacing: 2px;
        text-transform: uppercase;
        margin-bottom: 15px;
        display: block;
    }

    .reading-body { display: flex; gap: 25px; align-items: start; }

    .card-thumb {
        flex: 0 0 auto;
        width: 60px;
        height: 100px;
        background: linear-gradient(135deg, #0a0a0a 0%, #1a1a1a 100%);
        border: 2px solid #d4af37;
        border-radius: 4px;
        display: flex;
        align-items: center;
        justify-content: center;
        color: #d4af37;
        font-size: 1.8rem;
        font-weight: bold;
        font-family: 'Times New Roman', serif;
        box-shadow: 0 0 15px rgba(212, 175, 55, 0.25);
    }

    .reading-content { flex: 1; min-width: 0; }
    .reading-content h3 { margin: 0 0 10px 0; color: #fff; font-size: 1.3rem; font-weight: normal; }
    .reading-desc {
        color: #ccc; line-height: 1.6; font-size: 1rem;
        text-align: justify; font-weight: 300;
        word-wrap: break-word;
    }

    .answer-block {
        display: none;
        margin-top: 20px;
        padding: 16px 20px;
        background: linear-gradient(135deg, rgba(26, 20, 5, 0.6), rgba(15, 12, 2, 0.6));
        border: 1px solid rgba(212, 175, 55, 0.4);
        border-radius: 4px;
        transform-origin: top center;
    }
    .answer-block.revealed {
        display: block;
        animation: flipInAnswer 0.85s cubic-bezier(0.34, 1.2, 0.64, 1);
    }
    .answer-label {
        color: #ffd966;
        font-size: 0.7rem;
        letter-spacing: 2px;
        text-transform: uppercase;
        margin-bottom: 8px;
        font-weight: bold;
    }
    .answer-text {
        color: #f5e6b8;
        line-height: 1.6;
        font-size: 1rem;
        text-align: justify;
    }

    @keyframes flipInAnswer {
        from {
            transform: perspective(800px) rotateX(-75deg) translateY(-10px);
            opacity: 0;
        }
        to {
            transform: perspective(800px) rotateX(0deg) translateY(0);
            opacity: 1;
        }
    }
    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(15px); }
        to   { opacity: 1; transform: translateY(0); }
    }

    /* ===== BUTTONS ===== */
    button.restart-btn {
        display: block; margin: 30px auto 0; padding: 14px 45px;
        background: transparent; border: 1px solid #fff; color: #fff;
        cursor: pointer; font-family: serif; letter-spacing: 3px; text-transform: uppercase;
        font-size: 0.85rem; transition: all 0.4s;
    }
    button.restart-btn:hover { background: #fff; color: #000; box-shadow: 0 0 20px #fff; }

    @media (max-width: 600px) {
        #result-modal { padding: 22px 20px; width: 94%; }
        #result-modal h2 { font-size: 1.15rem; letter-spacing: 2px; }
        .reading-body { gap: 14px; }
        .card-thumb { width: 50px; height: 85px; font-size: 1.4rem; }
        .reading-content h3 { font-size: 1.05rem; }
        .reading-desc, .answer-text { font-size: 0.9rem; }
        #counter { top: 16px; right: 16px; font-size: 1rem; }
        #countdown-banner { font-size: 0.85rem; padding: 10px 14px; }
        #countdown-time { font-size: 1.1rem; }
        #skip-btn { margin-left: 8px; font-size: 0.7rem; padding: 4px 10px; }
    }
</style>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/tween.js/18.6.4/tween.umd.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
<script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>
</head>
<body>
    <video id="input-video" playsinline autoplay muted></video>
    <div id="canvas-container"></div>
    <div id="virtual-cursor"></div>

    <div id="ui-layer">
        <div id="counter">0 / 3</div>
        <div id="guide-text">Đưa tay lên trước Camera để bắt đầu</div>
        <div id="credit-text">tiktok: binnanee</div>

        <div id="result-modal">
            <h2>Lời Tiên Tri</h2>
            <div id="countdown-banner">
                ⏳ Thời gian suy nghĩ: <span id="countdown-time">2:00</span>
                <button id="skip-btn" type="button">Xem đáp án ⏩</button>
            </div>
            <div id="final-reading"></div>
            <button class="restart-btn" onclick="location.reload()">Trải Bài Mới</button>
        </div>
    </div>

    <div id="loading">
        <div class="loader-content">
            <div style="font-size: 2rem; color: #d4af37;">✦</div>
            <div style="font-size: 0.8rem; color: #888; letter-spacing: 4px; margin-top: 10px;">LOADING...</div>
        </div>
    </div>

<script>
/* ============================================================
   ⚠️  CÂU HỎI VÀ ĐÁP ÁN — CHỈNH SỬA TẠI ĐÂY
   Điền đáp án vào trường "answer" cho từng câu hỏi.
   Để trống "" nếu chưa có đáp án (sẽ hiện "(Chưa có đáp án)").
   ============================================================ */
const QUESTIONS = [
    {
        id: 1,
        text: "Thẻ HTML (tag) được viết trong cặp dấu gì và có phân biệt chữ hoa chữ thường không?",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 1
    },
    {
        id: 2,
        text: "Thế nào là thẻ đôi và thẻ đơn HTML? Cho ví dụ minh hoạ.",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 2
    },
    {
        id: 3,
        text: "Thông tin thuộc tính (attribute) trong thẻ HTML được dùng để làm gì? Hãy lấy ví dụ.",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 3
    },
    {
        id: 4,
        text: "Sự giống và khác nhau cơ bản giữa “Thẻ HTML” (tag) và “Phần tử HTML” (element) là gì?",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 4
    },
    {
        id: 5,
        text: "Trình duyệt web hiển thị nội dung web dựa trên cơ sở nào?",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 5
    },
    {
        id: 6,
        text: "Cấu trúc hình cây cơ bản của một tệp HTML bao gồm những phần tử gốc nào? Quan hệ giữa các phần tử này là gì?",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 6
    },
    {
        id: 7,
        text: "Trong cấu trúc tệp HTML, các thẻ <meta>, <title> thường nằm trong phần tử nào? Và thẻ <h1>, <p>, <div> thường nằm trong phần tử nào?",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 7
    },
    {
        id: 8,
        text: "Đoạn mã HTML sau đây hiển thị dòng chữ “Trang Web và HTML” ở định dạng nào?  <h1>Trang Web và HTML</h1>",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 8
    },
    {
        id: 9,
        text: "Kể tên các phần mềm soạn thảo HTML chuyên nghiệp nguồn mở và phần mềm cơ bản được nhắc đến trong bài.",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 9
    },
    {
        id: 10,
        text: "Ngoài phần mềm cài đặt trên máy tính, bài học giới thiệu những trang web nào hỗ trợ soạn thảo HTML trực tuyến?",
        answer: "" // <-- ĐIỀN ĐÁP ÁN CÂU 10
    }
];

const MAX_PICKS = 3;
const REVEAL_DELAY_SECONDS = 120; // 2 phút (đổi thành 5 nếu muốn test nhanh)

/* ============================================================
   CONFIG
   ============================================================ */
const CONFIG = {
    cardWidth: 1.4,
    cardHeight: 2.4,
    radius: 5.5,
    cameraZ: 9.0,
    inspectPos: { x: 0, y: 0, z: 7.4 },
    inspectScale: 0.42,
    storageZ: 7.6,
    storageScale: 0.085,
    storeMarginX: 0.12,
    storeMarginTop: 0.35,
    storeGapY: 0.24
};

const IDLE_SPEED = 0.0015;
const FAST_SPEED = 0.035;

/* ============================================================
   STATE
   ============================================================ */
let scene, camera, renderer, cardGroup, textureLoader;
let raycaster, mouse;
let starFieldMesh;
let explosions = [];
let storedCards = [];
let inspectingCard = null;
let isGameActive = true;
let readingScheduled = false;
let countdownFinished = false;
let countdownInterval = null;
let particleTexture, backTexture;

let handX = 0.5;
let handActive = false;
let isPinching = false;
let cursorPosition = { x: 0, y: 0 };
let rotationSpeed = IDLE_SPEED;

/* ============================================================
   HELPERS
   ============================================================ */
function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
}

function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}

function wrapText(ctx, text, maxWidth) {
    const words = (text || '').split(' ');
    const lines = [];
    let current = '';
    for (const word of words) {
        const test = current ? current + ' ' + word : word;
        if (ctx.measureText(test).width > maxWidth && current) {
            lines.push(current);
            current = word;
        } else {
            current = test;
        }
    }
    if (current) lines.push(current);
    return lines;
}

/* ============================================================
   TEXTURE GENERATION
   ============================================================ */
function generateParticleTexture() {
    const canvas = document.createElement('canvas');
    canvas.width = canvas.height = 64;
    const ctx = canvas.getContext('2d');
    const grad = ctx.createRadialGradient(32, 32, 0, 32, 32, 32);
    grad.addColorStop(0.0, 'rgba(255,255,255,1)');
    grad.addColorStop(0.4, 'rgba(255,255,255,0.5)');
    grad.addColorStop(1.0, 'rgba(255,255,255,0)');
    ctx.fillStyle = grad;
    ctx.fillRect(0, 0, 64, 64);
    const tex = new THREE.CanvasTexture(canvas);
    tex.needsUpdate = true;
    return tex;
}

function createCardBackTexture() {
    const S = 512;
    const canvas = document.createElement('canvas');
    canvas.width = canvas.height = S;
    const ctx = canvas.getContext('2d');

    const g = ctx.createRadialGradient(S / 2, S / 2, 0, S / 2, S / 2, S * 0.7);
    g.addColorStop(0, '#1a1a2e');
    g.addColorStop(0.5, '#0d0d1a');
    g.addColorStop(1, '#000000');
    ctx.fillStyle = g;
    ctx.fillRect(0, 0, S, S);

    ctx.strokeStyle = '#d4af37';
    ctx.lineWidth = 6;
    ctx.strokeRect(14, 14, S - 28, S - 28);
    ctx.lineWidth = 1.5;
    ctx.strokeRect(30, 30, S - 60, S - 60);

    ctx.save();
    ctx.translate(S / 2, S / 2);
    ctx.strokeStyle = 'rgba(212,175,55,0.85)';
    ctx.lineWidth = 2.5;

    for (let i = 0; i < 8; i++) {
        const a = (i * Math.PI) / 4;
        const inner = 40;
        const outer = i % 2 === 0 ? 160 : 110;
        ctx.beginPath();
        ctx.moveTo(Math.cos(a) * inner, Math.sin(a) * inner);
        ctx.lineTo(Math.cos(a) * outer, Math.sin(a) * outer);
        ctx.stroke();
    }

    ctx.beginPath(); ctx.arc(0, 0, 55, 0, Math.PI * 2); ctx.stroke();
    ctx.beginPath(); ctx.arc(0, 0, 110, 0, Math.PI * 2); ctx.stroke();
    ctx.beginPath(); ctx.arc(0, 0, 160, 0, Math.PI * 2); ctx.stroke();

    ctx.fillStyle = '#d4af37';
    ctx.beginPath(); ctx.arc(0, 0, 16, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#0d0d1a';
    ctx.beginPath(); ctx.arc(6, -4, 14, 0, Math.PI * 2); ctx.fill();
    ctx.restore();

    const tex = new THREE.CanvasTexture(canvas);
    tex.encoding = THREE.sRGBEncoding;
    tex.anisotropy = 4;
    return tex;
}

function createCardTexture(questionNum, bodyText, isAnswer) {
    const W = 512, H = 880;
    const canvas = document.createElement('canvas');
    canvas.width = W; canvas.height = H;
    const ctx = canvas.getContext('2d');

    // Nền đen
    const bg = ctx.createRadialGradient(W / 2, H / 2, 0, W / 2, H / 2, H * 0.7);
    bg.addColorStop(0, '#1a1a2e');
    bg.addColorStop(0.55, '#0a0a14');
    bg.addColorStop(1, '#000000');
    ctx.fillStyle = bg;
    ctx.fillRect(0, 0, W, H);

    // Viền vàng
    ctx.strokeStyle = '#d4af37';
    ctx.lineWidth = 6;
    ctx.strokeRect(14, 14, W - 28, H - 28);
    ctx.lineWidth = 1.5;
    ctx.strokeRect(30, 30, W - 60, H - 60);

    // Tiêu đề
    ctx.fillStyle = '#d4af37';
    ctx.font = 'bold 30px "Times New Roman", serif';
    ctx.textAlign = 'center';
    ctx.fillText(isAnswer ? 'ĐÁP ÁN' : 'CÂU HỎI', W / 2, 90);

    // Số câu trong vòng tròn
    ctx.beginPath();
    ctx.arc(W / 2, 155, 34, 0, Math.PI * 2);
    ctx.strokeStyle = '#d4af37';
    ctx.lineWidth = 3;
    ctx.stroke();
    ctx.fillStyle = '#fff';
    ctx.font = 'bold 36px "Times New Roman", serif';
    ctx.fillText(questionNum.toString(), W / 2, 168);

    // Đường kẻ
    ctx.beginPath();
    ctx.moveTo(80, 215);
    ctx.lineTo(W - 80, 215);
    ctx.strokeStyle = 'rgba(212,175,55,0.4)';
    ctx.lineWidth = 1;
    ctx.stroke();

    // Nội dung
    const displayText = (bodyText && bodyText.trim())
        ? bodyText
        : '(Chưa có đáp án)';

    ctx.fillStyle = isAnswer ? '#f5e6b8' : '#e0e0e0';
    ctx.font = '23px "Times New Roman", serif';
    ctx.textAlign = 'left';

    const maxWidth = W - 100;
    const lines = wrapText(ctx, displayText, maxWidth);
    const lineHeight = 34;
    const startY = 285;
    const maxLines = Math.floor((H - startY - 120) / lineHeight);

    for (let i = 0; i < lines.length && i < maxLines; i++) {
        ctx.fillText(lines[i], 50, startY + i * lineHeight);
    }
    if (lines.length > maxLines) {
        ctx.fillText('...', 50, startY + maxLines * lineHeight);
    }

    // Footer
    ctx.fillStyle = 'rgba(212,175,55,0.7)';
    ctx.font = '24px serif';
    ctx.textAlign = 'center';
    ctx.fillText('✦ ✦ ✦', W / 2, H - 60);

    const tex = new THREE.CanvasTexture(canvas);
    tex.encoding = THREE.sRGBEncoding;
    tex.anisotropy = 4;
    return tex;
}

/* ============================================================
   KHỞI TẠO
   ============================================================ */
function init() {
    const container = document.getElementById('canvas-container');

    scene = new THREE.Scene();
    scene.fog = new THREE.FogExp2(0x050505, 0.018);

    camera = new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 200);
    camera.position.set(0, 0, CONFIG.cameraZ);

    renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    renderer.outputEncoding = THREE.sRGBEncoding;
    container.appendChild(renderer.domElement);

    scene.add(new THREE.AmbientLight(0xffffff, 0.75));

    const key = new THREE.SpotLight(0xffffff, 2.0);
    key.position.set(0, 12, 14);
    scene.add(key);

    const rim = new THREE.DirectionalLight(0x8899ff, 0.5);
    rim.position.set(-8, -4, 6);
    scene.add(rim);

    textureLoader = new THREE.TextureLoader();
    particleTexture = generateParticleTexture();
    backTexture = createCardBackTexture();

    createCards();
    createBackgroundStars();

    raycaster = new THREE.Raycaster();
    mouse = new THREE.Vector2();

    document.addEventListener('mousedown', onMouseDown);
    document.addEventListener('touchstart', onTouchStart, { passive: true });
    window.addEventListener('resize', onResize);

    // Skip button
    document.getElementById('skip-btn').addEventListener('click', () => {
        if (countdownFinished) return;
        if (countdownInterval) clearInterval(countdownInterval);
        revealAnswers();
    });

    animate(0);
    setTimeout(hideLoading, 2200);
}

function hideLoading() {
    const el = document.getElementById('loading');
    el.style.opacity = 0;
    setTimeout(() => { el.style.display = 'none'; }, 1500);
}

/* ============================================================
   BACKGROUND
   ============================================================ */
function createBackgroundStars() {
    const starCount = 800;
    const geo = new THREE.BufferGeometry();
    const positions = [];
    const sizes = [];

    for (let i = 0; i < starCount; i++) {
        positions.push(
            (Math.random() - 0.5) * 70,
            (Math.random() - 0.5) * 45,
            (Math.random() - 0.5) * 50 - 15
        );
        sizes.push(Math.random() * 0.5 + 0.1);
    }

    geo.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
    geo.setAttribute('size', new THREE.Float32BufferAttribute(sizes, 1));

    const mat = new THREE.PointsMaterial({
        size: 0.8,
        map: particleTexture,
        transparent: true,
        opacity: 0.9,
        depthWrite: false,
        blending: THREE.AdditiveBlending,
        color: 0xffffff
    });

    starFieldMesh = new THREE.Points(geo, mat);
    starFieldMesh.frustumCulled = false;
    scene.add(starFieldMesh);
}

/* ============================================================
   CARDS
   ============================================================ */
function createCards() {
    cardGroup = new THREE.Group();
    scene.add(cardGroup);

    const geo = new THREE.BoxGeometry(CONFIG.cardWidth, CONFIG.cardHeight, 0.02);
    const matEdge = new THREE.MeshStandardMaterial({ color: 0xd4af37, metalness: 0.85, roughness: 0.25 });

    const total = QUESTIONS.length;

    QUESTIONS.forEach((q, i) => {
        const frontTex = createCardTexture(q.id, q.text, false);
        const answerTex = createCardTexture(q.id, q.answer, true);

        const matFront = new THREE.MeshStandardMaterial({ map: frontTex, roughness: 0.7, metalness: 0.05 });
        const matBack = new THREE.MeshStandardMaterial({ map: backTexture, roughness: 0.6, metalness: 0.1 });

        // BoxGeometry: [+X, -X, +Y, -Y, +Z, -Z]
        // -Z = mặt trước (câu hỏi), +Z = mặt sau (thiết kế ban đầu)
        const materials = [matEdge, matEdge, matEdge, matEdge, matBack, matFront];

        const card = new THREE.Mesh(geo, materials);

        const angle = (i / total) * Math.PI * 2;
        card.position.set(Math.sin(angle) * CONFIG.radius, 0, Math.cos(angle) * CONFIG.radius);
        card.lookAt(0, 0, 0);

        card.userData = {
            id: i,
            question: q,
            isPicked: false,
            answerTexture: answerTex,
            backMeshMaterial: matBack
        };
        cardGroup.add(card);
    });
}

/* ============================================================
   PARTICLES
   ============================================================ */
function createExplosion(position) {
    const particleCount = 150;
    const geo = new THREE.BufferGeometry();
    const positions = [];
    const velocities = [];
    const sizes = [];

    for (let i = 0; i < particleCount; i++) {
        positions.push(position.x, position.y, position.z);

        const theta = Math.random() * Math.PI * 2;
        const phi = Math.acos(2 * Math.random() - 1);
        const speed = Math.random() * 0.03 + 0.01;

        velocities.push(
            speed * Math.sin(phi) * Math.cos(theta),
            speed * Math.sin(phi) * Math.sin(theta),
            speed * Math.cos(phi)
        );

        sizes.push(Math.random() * 0.15 + 0.05);
    }

    geo.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
    geo.setAttribute('size', new THREE.Float32BufferAttribute(sizes, 1));

    const mat = new THREE.PointsMaterial({
        size: 0.2,
        map: particleTexture,
        transparent: true,
        opacity: 0.9,
        depthWrite: false,
        blending: THREE.AdditiveBlending,
        color: 0xffffff
    });

    const points = new THREE.Points(geo, mat);
    points.frustumCulled = false;
    scene.add(points);
    explosions.push({ mesh: points, velocities: velocities, age: 0 });
}

function updateExplosions() {
    for (let i = explosions.length - 1; i >= 0; i--) {
        const ex = explosions[i];
        const positions = ex.mesh.geometry.attributes.position.array;
        ex.age += 1;

        for (let j = 0; j < ex.velocities.length / 3; j++) {
            positions[j * 3]     += ex.velocities[j * 3];
            positions[j * 3 + 1] += ex.velocities[j * 3 + 1];
            positions[j * 3 + 2] += ex.velocities[j * 3 + 2];
        }

        ex.mesh.geometry.attributes.position.needsUpdate = true;
        ex.mesh.material.opacity = Math.max(0, 1 - ex.age / 60);
        ex.mesh.scale.setScalar(1 + ex.age * 0.01);

        if (ex.age > 60) {
            scene.remove(ex.mesh);
            ex.mesh.geometry.dispose();
            ex.mesh.material.dispose();
            explosions.splice(i, 1);
        }
    }
}

/* ============================================================
   LUỒNG CHỌN BÀI
   ============================================================ */
function pickCard(card) {
    if (!isGameActive || inspectingCard || storedCards.length >= MAX_PICKS) return;
    if (!card || card.userData.isPicked) return;

    card.userData.isPicked = true;
    inspectingCard = card;
    scene.attach(card);

    new TWEEN.Tween(card.position)
        .to(CONFIG.inspectPos, 1000)
        .easing(TWEEN.Easing.Cubic.Out)
        .start();

    new TWEEN.Tween(card.rotation)
        .to({ x: 0, y: Math.PI, z: 0 }, 1000)
        .easing(TWEEN.Easing.Back.Out)
        .start();

    const s = CONFIG.inspectScale;
    new TWEEN.Tween(card.scale)
        .to({ x: s, y: s, z: s }, 1000)
        .easing(TWEEN.Easing.Cubic.Out)
        .onComplete(() => {
            setTimeout(() => {
                if (!inspectingCard) return;
                createExplosion(card.position);
                setTimeout(storeCard, 100);
            }, 1400);
        })
        .start();
}

function getSafeVerticalLeftPosition(index) {
    const dist = camera.position.z - CONFIG.storageZ;
    const vFOV = THREE.MathUtils.degToRad(camera.fov);
    const visibleHeight = 2 * Math.tan(vFOV / 2) * dist;
    const visibleWidth = visibleHeight * camera.aspect;

    const leftEdge = -visibleWidth / 2;
    const topEdge = visibleHeight / 2;

    return {
        x: leftEdge + CONFIG.storeMarginX,
        y: topEdge - CONFIG.storeMarginTop - index * CONFIG.storeGapY,
        z: CONFIG.storageZ
    };
}

function storeCard() {
    if (!inspectingCard) return;

    const card = inspectingCard;
    inspectingCard = null;
    storedCards.push(card);

    updateCounter();
    repositionStoredCards();
}

function repositionStoredCards() {
    const s = CONFIG.storageScale;
    storedCards.forEach((c, i) => {
        const pos = getSafeVerticalLeftPosition(i);
        new TWEEN.Tween(c.position).to(pos, 600).easing(TWEEN.Easing.Exponential.Out).start();
        new TWEEN.Tween(c.scale).to({ x: s, y: s, z: s }, 600).start();
    });
}

function updateCounter() {
    document.getElementById('counter').innerText = `${storedCards.length} / ${MAX_PICKS}`;

    if (storedCards.length >= MAX_PICKS && !readingScheduled) {
        readingScheduled = true;
        isGameActive = false;
        document.getElementById('guide-text').innerText = 'Định mệnh đã an bài...';
        document.getElementById('virtual-cursor').style.display = 'none';
        setTimeout(revealReading, 1600);
    }
}

/* ============================================================
   MODAL & COUNTDOWN
   ============================================================ */
function revealReading() {
    const content = document.getElementById('final-reading');
    const labels = ['LÁ BÀI 1', 'LÁ BÀI 2', 'LÁ BÀI 3'];
    let html = '';

    storedCards.forEach((c, i) => {
        const q = c.userData.question;
        html += `
        <div class="reading-section" data-index="${i}">
            <span class="reading-label">✦ ${labels[i]}</span>
            <div class="reading-body">
                <div class="card-thumb">${q.id}</div>
                <div class="reading-content">
                    <h3>Câu ${q.id}</h3>
                    <div class="reading-desc">${escapeHtml(q.text)}</div>
                    <div class="answer-block" id="answer-${i}">
                        <div class="answer-label">✦ Đáp án</div>
                        <div class="answer-text">${escapeHtml(q.answer && q.answer.trim() ? q.answer : '(Chưa có đáp án)')}</div>
                    </div>
                </div>
            </div>
        </div>`;
    });

    content.innerHTML = html;

    document.getElementById('guide-text').style.opacity = 0;
    document.getElementById('ui-layer').style.pointerEvents = 'auto';
    document.getElementById('result-modal').style.display = 'block';
    document.getElementById('countdown-banner').style.display = 'block';

    startCountdown();
}

function startCountdown() {
    let remaining = REVEAL_DELAY_SECONDS;
    const timeEl = document.getElementById('countdown-time');

    function updateTime() {
        const m = Math.floor(remaining / 60);
        const s = remaining % 60;
        timeEl.textContent = `${m}:${s.toString().padStart(2, '0')}`;
    }
    updateTime();

    countdownInterval = setInterval(() => {
        remaining--;
        updateTime();
        if (remaining <= 0) {
            clearInterval(countdownInterval);
            revealAnswers();
        }
    }, 1000);
}

function revealAnswers() {
    if (countdownFinished) return;
    countdownFinished = true;

    // 1) Flip 3 lá bài đã chọn để lộ đáp án trên mặt sau
    storedCards.forEach((card, i) => {
        // Đổi material mặt sau thành đáp án
        card.material[4].map = card.userData.answerTexture;
        card.material[4].needsUpdate = true;

        // Xoay lá bài để hiện mặt sau (y từ Math.PI -> 0)
        new TWEEN.Tween(card.rotation)
            .to({ y: 0 }, 1100)
            .delay(i * 250)
            .easing(TWEEN.Easing.Cubic.InOut)
            .start();
    });

    // 2) Hiện đáp án trong modal với hiệu ứng lật
    setTimeout(() => {
        document.querySelectorAll('.answer-block').forEach((el, i) => {
            setTimeout(() => { el.classList.add('revealed'); }, i * 350);
        });

        const banner = document.getElementById('countdown-banner');
        banner.innerHTML = '✨ Đáp án đã được hé lộ ✨';
        banner.style.color = '#ffd966';
        banner.style.animation = 'none';
        banner.style.borderColor = 'rgba(255, 217, 102, 0.6)';
        banner.style.background = 'rgba(255, 217, 102, 0.1)';
    }, 700);
}

/* ============================================================
   INPUT: CHUỘT / TOUCH
   ============================================================ */
function setRayFromScreen(px, py) {
    mouse.x = (px / window.innerWidth) * 2 - 1;
    mouse.y = -(py / window.innerHeight) * 2 + 1;
}

function tryPickAtCursor() {
    if (!isGameActive) return;
    raycaster.setFromCamera(mouse, camera);
    const intersects = raycaster.intersectObjects(cardGroup.children, false);
    if (intersects.length > 0) pickCard(intersects[0].object);
}

function onMouseDown(e) {
    if (!isGameActive) return;
    setRayFromScreen(e.clientX, e.clientY);
    tryPickAtCursor();
}

function onTouchStart(e) {
    if (!isGameActive || !e.touches.length) return;
    setRayFromScreen(e.touches[0].clientX, e.touches[0].clientY);
    tryPickAtCursor();
}

/* ============================================================
   HAND TRACKING
   ============================================================ */
function onResults(results) {
    const cursor = document.getElementById('virtual-cursor');
    const guide = document.getElementById('guide-text');

    const hasHand = results.multiHandLandmarks && results.multiHandLandmarks.length > 0;

    if (!hasHand) {
        handActive = false;
        handX = 0.5;
        isPinching = false;
        cursor.classList.remove('pinching');
        cursor.style.display = 'none';
        return;
    }

    handActive = true;
    const lm = results.multiHandLandmarks[0];

    if (isGameActive && storedCards.length === 0) {
        guide.innerText = 'Chụm ngón cái và trỏ để chọn';
    }

    handX = 1 - lm[9].x;

    const x = (lm[8].x + lm[4].x) / 2;
    const y = (lm[8].y + lm[4].y) / 2;

    const screenX = (1 - x) * window.innerWidth;
    const screenY = y * window.innerHeight;

    cursor.style.display = 'block';
    cursor.style.left = `${screenX}px`;
    cursor.style.top = `${screenY}px`;

    cursorPosition.x = (screenX / window.innerWidth) * 2 - 1;
    cursorPosition.y = -(screenY / window.innerHeight) * 2 + 1;

    const distance = Math.hypot(lm[8].x - lm[4].x, lm[8].y - lm[4].y);
    const pinchingNow = distance < 0.07;

    if (pinchingNow && !isPinching) {
        cursor.classList.add('pinching');
        if (isGameActive) {
            raycaster.setFromCamera(cursorPosition, camera);
            const intersects = raycaster.intersectObjects(cardGroup.children, false);
            if (intersects.length > 0) pickCard(intersects[0].object);
        }
    } else if (!pinchingNow) {
        cursor.classList.remove('pinching');
    }

    isPinching = pinchingNow;
}

/* ============================================================
   RESIZE
   ============================================================ */
function onResize() {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
    repositionStoredCards();
}

/* ============================================================
   RENDER LOOP
   ============================================================ */
function animate(time) {
    requestAnimationFrame(animate);
    TWEEN.update(time);
    updateExplosions();

    if (isGameActive) {
        let target = IDLE_SPEED;
        if (handActive) {
            if (handX < 0.42) target = -FAST_SPEED;
            else if (handX > 0.58) target = FAST_SPEED;
        }

        rotationSpeed += (target - rotationSpeed) * 0.06;
        cardGroup.rotation.y += rotationSpeed;

        const t = time * 0.0015;
        cardGroup.children.forEach((c, i) => {
            if (!c.userData.isPicked) {
                c.position.y = Math.sin(t + i) * 0.15;
            }
        });
    }

    if (starFieldMesh) {
        starFieldMesh.rotation.y = time * 0.00005;
        starFieldMesh.material.size = Math.sin(time * 0.002) * 0.1 + 0.9;
    }

    renderer.render(scene, camera);
}

/* ============================================================
   BOOT
   ============================================================ */
init();

const hands = new Hands({
    locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
});

hands.setOptions({
    maxNumHands: 1,
    modelComplexity: 1,
    minDetectionConfidence: 0.5,
    minTrackingConfidence: 0.5
});

hands.onResults(onResults);

const videoElement = document.getElementById('input-video');
const cam = new Camera(videoElement, {
    onFrame: async () => { await hands.send({ image: videoElement }); },
    width: 640,
    height: 480
});

cam.start().catch((err) => {
    console.warn('Không mở được camera:', err);
    document.getElementById('guide-text').innerText = 'Không có quyền Camera — dùng chuột để chọn bài';
});
</script>
</body>
</html>
