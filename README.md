```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>나만의 RPG 퀘스트</title>
    
    <!-- 모바일 앱(PWA) 설치용 태그 추가 -->
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="퀘스트 앱">
    <link rel="apple-touch-icon" href="https://placehold.co/192x192/0f3460/ffffff?text=Q">

    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Do+Hyeon&family=Press+Start+2P&display=swap" rel="stylesheet">
    <!-- FontAwesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        :root {
            --bg-color: #1a1a2e;
            --panel-bg: #16213e;
            --accent: #e94560;
            --text-main: #e2e8f0;
        }
        
        body {
            background-color: var(--bg-color);
            color: var(--text-main);
            font-family: 'Do Hyeon', sans-serif;
            background-image: 
                linear-gradient(rgba(26, 26, 46, 0.9), rgba(26, 26, 46, 0.9)),
                url('https://www.transparenttextures.com/patterns/cubes.png');
            overflow-x: hidden;
            -webkit-tap-highlight-color: transparent;
        }

        .font-retro { font-family: 'Press Start 2P', cursive; }

        /* UI Panel Styling */
        .game-panel {
            background-color: var(--panel-bg);
            border: 4px solid #0f3460;
            border-radius: 12px;
            box-shadow: 0 8px 0 #0a192f, inset 0 0 20px rgba(0,0,0,0.5);
            transition: transform 0.2s, box-shadow 0.2s;
        }

        .retro-input {
            background: #0a192f;
            border: 2px solid #0f3460;
            color: white;
            outline: none;
            transition: all 0.3s ease;
        }
        .retro-input:focus { border-color: #e94560; }

        /* Progress Bars */
        .bar-container {
            width: 100%;
            background-color: #000;
            border: 2px solid #333;
            border-radius: 8px;
            overflow: hidden;
            height: 20px;
            position: relative;
        }
        .hp-bar { background: linear-gradient(90deg, #ff0000, #ff5e5e); height: 100%; transition: width 0.4s; }
        .exp-bar { background: linear-gradient(90deg, #00d2ff, #3a7bd5); height: 100%; transition: width 0.4s; }
        
        /* Animations */
        @keyframes floatUp { 0% { transform: translateY(0) scale(1); opacity: 1; } 100% { transform: translateY(-50px) scale(1.2); opacity: 0; } }
        .animate-float-up { animation: floatUp 1s ease-out forwards; text-shadow: 2px 2px 0 #000, -1px -1px 0 #000, 1px -1px 0 #000, -1px 1px 0 #000; }
        @keyframes shake { 0%, 100% { transform: translateX(0); } 20%, 60% { transform: translateX(-10px); } 40%, 80% { transform: translateX(10px); } }
        .shake-animation { animation: shake 0.4s ease-in-out; }
        @keyframes levelUpGlow { 0% { box-shadow: 0 0 10px #ffd700; } 50% { box-shadow: 0 0 40px #ffd700, 0 0 80px #ff8c00; } 100% { box-shadow: 0 0 10px #ffd700; } }
        .level-up-anim { animation: levelUpGlow 1.5s ease-out; }

        /* Checkbox */
        .quest-checkbox {
            appearance: none; width: 28px; height: 28px; border: 2px solid #00d2ff; border-radius: 6px;
            cursor: pointer; position: relative; background: rgba(0,0,0,0.5); transition: all 0.2s;
        }
        .quest-checkbox:checked { background: #00d2ff; }
        .quest-checkbox:checked::after {
            content: '✓'; position: absolute; color: #000; font-size: 20px; font-weight: bold;
            top: 50%; left: 50%; transform: translate(-50%, -50%);
        }

        /* Calendar */
        .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 4px; }
        .calendar-day-header { text-align: center; font-weight: bold; padding: 6px 0; background-color: rgba(0,0,0,0.3); border-radius: 4px; font-size: 0.9rem;}
        .calendar-cell { aspect-ratio: 1; background-color: rgba(255,255,255,0.05); border: 1px solid #2d3748; border-radius: 6px; display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; }
        .calendar-cell.today { border-color: #e94560; background-color: rgba(233, 69, 96, 0.1); }
        .calendar-date-num { position: absolute; top: 4px; left: 6px; font-size: 0.75rem; color: #a0aec0; }
        
        /* Loading Overlay */
        #loading-screen { background-color: var(--bg-color); z-index: 100; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .spinner { border: 4px solid rgba(255,255,255,0.1); width: 40px; height: 40px; border-radius: 50%; border-left-color: #00d2ff; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: #0a192f; }
        ::-webkit-scrollbar-thumb { background: #0f3460; border-radius: 3px; }
    </style>
</head>
<body class="min-h-screen p-3 md:p-6 flex justify-center items-start">

    <div id="loading-screen" class="fixed inset-0 text-white">
        <div class="spinner mb-4"></div>
        <div class="font-retro text-lg text-cyan-400">데이터 동기화 중...</div>
    </div>

    <div class="max-w-md w-full flex flex-col gap-4 opacity-0 transition-opacity duration-500" id="game-container">
        
        <!-- Header -->
        <div class="text-center mt-2 mb-2">
            <h1 class="text-3xl md:text-4xl text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 to-blue-500 font-retro mb-1">
                QUEST APP
            </h1>
            <p class="text-gray-400 text-sm"><span id="display-nickname" class="text-cyan-300 font-bold"></span> 용사님 환영합니다!</p>
        </div>

        <!-- Navigation Tabs -->
        <div class="flex gap-2">
            <button id="tab-btn-board" class="flex-1 bg-[#0f3460] text-white border-2 border-blue-800 py-2 rounded-lg font-bold shadow-md">퀘스트</button>
            <button id="tab-btn-calendar" class="flex-1 bg-gray-800 text-gray-400 border-2 border-gray-700 py-2 rounded-lg font-bold shadow-md">달력</button>
            <button id="tab-btn-shop" class="flex-1 bg-gray-800 text-gray-400 border-2 border-gray-700 py-2 rounded-lg font-bold shadow-md text-purple-300">상점</button>
        </div>

        <!-- Player Stats Panel -->
        <div class="game-panel p-4 relative" id="player-stats-panel">
            <div class="flex items-center gap-4">
                <!-- Avatar & Level -->
                <div class="flex flex-col items-center">
                    <div class="w-16 h-16 bg-gray-800 rounded-full border-4 border-yellow-400 flex items-center justify-center text-3xl shadow-lg relative bg-cover bg-center" style="background-image: url('https://placehold.co/100x100/2d3748/ffffff?text=ME');">
                        <span id="player-avatar">🧙‍♂️</span>
                        <div class="absolute -bottom-2 bg-yellow-500 text-black font-retro text-[10px] px-1.5 py-0.5 rounded border border-yellow-700">
                            Lv.<span id="stat-level">1</span>
                        </div>
                    </div>
                </div>

                <!-- HP & EXP Bars -->
                <div class="flex-1 flex flex-col gap-2">
                    <div>
                        <div class="flex justify-between mb-0.5 text-xs text-red-400 font-bold">
                            <span><i class="fa-solid fa-heart"></i> HP</span>
                            <span id="stat-hp-text">100/100</span>
                        </div>
                        <div class="bar-container h-3">
                            <div class="hp-bar" id="stat-hp-bar" style="width: 100%"></div>
                        </div>
                    </div>
                    <div>
                        <div class="flex justify-between mb-0.5 text-xs text-blue-400 font-bold">
                            <span><i class="fa-solid fa-star"></i> EXP</span>
                            <span id="stat-exp-text">0/100</span>
                        </div>
                        <div class="bar-container h-3">
                            <div class="exp-bar" id="stat-exp-bar" style="width: 0%"></div>
                        </div>
                    </div>
                </div>

                <!-- Gold -->
                <div class="flex flex-col items-center justify-center bg-black bg-opacity-40 p-2 rounded-lg border border-gray-700 min-w-[70px]">
                    <div class="text-yellow-400 text-xl"><i class="fa-solid fa-coins"></i></div>
                    <div class="text-sm font-retro text-yellow-300 mt-1" id="stat-gold">0 G</div>
                </div>
            </div>
        </div>

        <div id="view-board" class="flex flex-col gap-4 w-full view-section">
            <div class="game-panel p-4">
                <h2 class="text-xl mb-3 border-b-2 border-gray-700 pb-2 flex items-center justify-between">
                    <span><i class="fa-solid fa-scroll mr-2 text-yellow-400"></i>오늘의 미션</span>
                    <span id="quest-count" class="text-xs bg-gray-800 px-2 py-1 rounded-full text-gray-300 border border-gray-600">0 건</span>
                </h2>
                <ul id="quest-list" class="flex flex-col gap-3"></ul>
                <div id="empty-state" class="text-center py-6 text-gray-500 hidden">
                    <i class="fa-solid fa-bed text-4xl mb-2 text-gray-600"></i>
                    <p class="text-sm">관리자가 등록한 미션이 없습니다.</p>
                </div>
            </div>

            <!-- Completed Quests -->
            <div class="game-panel p-4 bg-opacity-50">
                <h2 class="text-md mb-3 border-b border-gray-700 pb-1 text-gray-400"><i class="fa-solid fa-check-double mr-1 text-green-500"></i>완료 기록</h2>
                <ul id="completed-quest-list" class="flex flex-col gap-2 max-h-[200px] overflow-y-auto pr-2"></ul>
            </div>
        </div>

        <div id="view-calendar" class="game-panel p-4 w-full hidden view-section">
            <div class="flex justify-between items-center mb-4">
                <button id="cal-prev" class="text-xl text-gray-400 px-3 py-1"><i class="fa-solid fa-caret-left"></i></button>
                <h2 id="cal-month-title" class="text-lg font-bold text-yellow-400">2026년 9월</h2>
                <button id="cal-next" class="text-xl text-gray-400 px-3 py-1"><i class="fa-solid fa-caret-right"></i></button>
            </div>
            
            <div class="calendar-grid mb-1">
                <div class="calendar-day-header text-red-400">일</div>
                <div class="calendar-day-header">월</div>
                <div class="calendar-day-header">화</div>
                <div class="calendar-day-header">수</div>
                <div class="calendar-day-header">목</div>
                <div class="calendar-day-header">금</div>
                <div class="calendar-day-header text-blue-400">토</div>
            </div>
            <div id="calendar-body" class="calendar-grid mb-3"></div>
            
            <div class="p-3 border border-gray-700 rounded-lg bg-black bg-opacity-30">
                <h3 id="cal-detail-title" class="text-yellow-400 text-sm font-bold border-b border-gray-700 pb-1 mb-2">날짜를 클릭하세요</h3>
                <ul id="cal-detail-list" class="flex flex-col gap-1 text-sm">
                    <li class="text-gray-500 text-center py-2 text-xs">기록을 확인할 날짜를 선택하세요.</li>
                </ul>
            </div>
        </div>

        <div id="view-shop" class="game-panel p-4 w-full hidden view-section">
            <div class="text-center mb-4 border-b border-gray-700 pb-3">
                <h2 class="text-xl font-bold text-purple-400 mb-1"><i class="fa-solid fa-store mr-1"></i>포인트 교환소</h2>
                <p class="text-xs text-gray-400">열심히 모은 골드를 실제 상품권으로!</p>
            </div>
            <div class="grid grid-cols-2 gap-3" id="shop-container">
                <!-- Generated by JS -->
            </div>
        </div>
    </div>

    <div id="modal-overlay" class="fixed inset-0 bg-black bg-opacity-90 z-50 hidden flex items-center justify-center p-4 backdrop-blur-sm transition-opacity duration-300 opacity-0">
        
        <!-- Info Modal -->
        <div id="info-modal" class="hidden game-panel p-6 text-center max-w-sm w-full transform scale-90 transition-transform duration-300">
            <h2 id="info-modal-title" class="text-3xl font-bold text-yellow-400 mb-3 font-retro">TITLE</h2>
            <p id="info-modal-desc" class="text-lg text-white mb-4 whitespace-pre-line"></p>
            <div id="info-modal-stats" class="bg-black bg-opacity-50 p-3 rounded mb-5 text-left text-sm space-y-1"></div>
            <button id="info-modal-close" class="bg-gradient-to-b from-blue-400 to-blue-600 border-2 border-blue-800 text-white font-bold py-3 px-8 rounded shadow-[0_4px_0_#1e3a8a] active:shadow-none active:translate-y-1 transition-all w-full">확인</button>
        </div>

        <!-- Confirm/Abandon Modal -->
        <div id="confirm-modal" class="hidden game-panel p-5 text-center max-w-sm w-full transform scale-90 transition-transform duration-300">
            <h2 class="text-xl font-bold text-red-500 mb-3"><i class="fa-solid fa-triangle-exclamation mr-1"></i>경고</h2>
            <p id="confirm-modal-desc" class="text-sm text-white mb-5 whitespace-pre-line"></p>
            <div class="flex gap-3">
                <button id="confirm-modal-cancel" class="flex-1 bg-gray-600 border border-gray-800 text-white py-2 rounded">취소</button>
                <button id="confirm-modal-ok" class="flex-1 bg-red-600 border-b-4 border-red-800 text-white font-bold py-2 rounded active:border-b-0 active:translate-y-1 transition-all">포기하기</button>
            </div>
        </div>

        <!-- Initial Registration Modal -->
        <div id="nickname-modal" class="hidden game-panel p-6 text-center max-w-sm w-full transform scale-90 transition-transform duration-300">
            <h2 class="text-2xl font-bold text-cyan-400 mb-2"><i class="fa-solid fa-user-astronaut mr-2"></i>용사 등록</h2>
            <p class="text-xs text-gray-400 mb-5">사용하실 닉네임을 입력해주세요.</p>
            <input type="text" id="nickname-input" class="retro-input w-full p-3 rounded text-center text-lg mb-5" placeholder="닉네임 (최대 6자)" maxlength="6">
            <button id="nickname-ok" class="bg-cyan-600 border-b-4 border-cyan-800 text-white font-bold py-3 w-full rounded active:border-b-0 active:translate-y-1 transition-all">모험 시작</button>
        </div>
        
        <!-- Quiz Action Modal -->
        <div id="quiz-modal" class="hidden game-panel p-6 text-center max-w-sm w-full transform scale-90 transition-transform duration-300 border-amber-500 shadow-[0_0_20px_#f59e0b]">
            <h2 class="text-2xl font-bold text-amber-500 mb-2 font-retro">QUIZ EVENT!</h2>
            <p class="text-sm text-gray-300 mb-4" id="quiz-question-text">질문이 여기에 표시됩니다.</p>
            <input type="text" id="quiz-answer-input" class="retro-input w-full p-3 rounded text-center text-lg mb-4" placeholder="정답 입력">
            <button id="quiz-submit-btn" class="bg-amber-600 border-b-4 border-amber-800 text-white font-bold py-3 w-full rounded active:border-b-0 active:translate-y-1 transition-all">제출하기</button>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, onSnapshot, collection } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const appId = typeof __app_id !== 'undefined' ? __app_id : 'quest-platform-v2';
        let firebaseConfig = {};
        try { if(typeof __firebase_config !== 'undefined') firebaseConfig = JSON.parse(__firebase_config); } catch(e) {}
        
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        let userId = null;

        // Configuration
        const RANKS = {
            'D': { label: 'D랭크', color: 'bg-gray-600 text-white', exp: 10, gold: 5, damage: 10, icon: 'fa-star' },
            'C': { label: 'C랭크', color: 'bg-green-600 text-white', exp: 20, gold: 15, damage: 20, icon: 'fa-shield' },
            'A': { label: 'A랭크', color: 'bg-purple-600 text-white', exp: 50, gold: 40, damage: 40, icon: 'fa-khanda' },
            'S': { label: 'S랭크', color: 'bg-yellow-500 text-black', exp: 100, gold: 100, damage: 80, icon: 'fa-dragon' }
        };

        let player = {
            nickname: '', level: 1, hp: 100, maxHp: 100, exp: 0, maxExp: 100, gold: 0, quests: []
        };
        
        let globalQuizzes = [];
        let globalShop = [];
        let currentCalDate = new Date();
        let selectedCalDateStr = null;
        let confirmCallback = null;
        let activeQuiz = null; // Stores quiz data during level up

        function getDateString(date) {
            const y = date.getFullYear();
            const m = String(date.getMonth() + 1).padStart(2, '0');
            const d = String(date.getDate()).padStart(2, '0');
            return `${y}-${m}-${d}`;
        }
        const TODAY_STR = getDateString(new Date());

        // --- Init & Auth ---
        async function init() {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
                userId = auth.currentUser.uid;
                
                // Fetch User Profile
                const userRef = doc(db, 'artifacts', appId, 'public', 'data', 'users', userId);
                const userSnap = await getDoc(userRef);
                
                document.getElementById('loading-screen').style.display = 'none';

                if (userSnap.exists() && userSnap.data().nickname) {
                    player = { ...player, ...userSnap.data() };
                    if(!player.quests) player.quests = [];
                    startGameEngine();
                } else {
                    // Ask Nickname first time
                    openOverlay();
                    const nModal = document.getElementById('nickname-modal');
                    nModal.classList.remove('hidden');
                    setTimeout(() => nModal.classList.replace('scale-90', 'scale-100'), 10);
                    
                    document.getElementById('nickname-ok').onclick = async () => {
                        const n = document.getElementById('nickname-input').value.trim() || '무명용사';
                        player.nickname = n;
                        await saveToCloud();
                        closeAllModals();
                        startGameEngine();
                    };
                }
            } catch(e) {
                console.error(e);
                document.getElementById('loading-screen').style.display = 'none';
                alert("통신 중 오류가 발생했습니다. 새로고침 해주세요.");
            }
        }

        // --- Core Engine ---
        function startGameEngine() {
            document.getElementById('game-container').classList.remove('opacity-0');
            document.getElementById('display-nickname').innerText = player.nickname;

            // Setup Tab Listeners
            setupTabs();
            setupCalendarListeners();

            // Setup Cloud Listeners
            listenToCloudData();
            
            // Initial Renders
            updateUI();
            renderCalendar();
        }

        function listenToCloudData() {
            if(!userId) return;

            // Listen to Admin Global Quests
            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'quests'), (snap) => {
                let changed = false;
                const activeGlobalIds = snap.docs.map(d => d.id);

                // 1. 관리자가 삭제한 퀘스트 실시간 동기화 (오늘 아직 완료하지 않은 퀘스트 대상)
                const initialLen = player.quests.length;
                player.quests = player.quests.filter(q => {
                    if (q.createdAt === TODAY_STR && !q.completed && q.globalId) {
                        return activeGlobalIds.includes(q.globalId);
                    }
                    return true;
                });
                if (player.quests.length !== initialLen) changed = true;

                // 2. 관리자가 새로 추가한 퀘스트 동기화
                snap.docs.forEach(docSnap => {
                    const qData = docSnap.data();
                    // If not already in player's list FOR TODAY, add it.
                    const alreadyHas = player.quests.some(q => q.globalId === docSnap.id && q.createdAt === TODAY_STR);
                    if(!alreadyHas) {
                        player.quests.push({
                            id: 'q_' + Date.now() + Math.random(),
                            globalId: docSnap.id,
                            text: qData.text,
                            rank: qData.rank,
                            completed: false,
                            createdAt: TODAY_STR,
                            timestamp: Date.now()
                        });
                        changed = true;
                    }
                });
                
                if(changed) saveToCloud();
                renderQuests();
            }, (e) => console.error(e));

            // Listen to Shop
            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'shop'), (snap) => {
                globalShop = snap.docs.map(d => ({id: d.id, ...d.data()}));
                renderShop();
            }, (e) => console.error(e));

            // Listen to Quizzes
            onSnapshot(collection(db, 'artifacts', appId, 'public', 'data', 'quizzes'), (snap) => {
                globalQuizzes = snap.docs.map(d => ({id: d.id, ...d.data()}));
            }, (e) => console.error(e));
        }

        async function saveToCloud() {
            if(!userId) return;
            const ref = doc(db, 'artifacts', appId, 'public', 'data', 'users', userId);
            await setDoc(ref, player).catch(e => console.error(e));
            updateUI();
        }

        // --- UI Updates ---
        function updateUI() {
            document.getElementById('stat-level').innerText = player.level;
            
            // HP
            document.getElementById('stat-hp-text').innerText = `${Math.floor(player.hp)}/${player.maxHp}`;
            let hpPct = Math.max(0, Math.min(100, (player.hp / player.maxHp) * 100));
            document.getElementById('stat-hp-bar').style.width = `${hpPct}%`;

            // EXP
            document.getElementById('stat-exp-text').innerText = `${Math.floor(player.exp)}/${player.maxExp}`;
            let expPct = Math.max(0, Math.min(100, (player.exp / player.maxExp) * 100));
            document.getElementById('stat-exp-bar').style.width = `${expPct}%`;

            // Gold
            document.getElementById('stat-gold').innerText = `${player.gold} G`;
        }

        function renderQuests() {
            const listEl = document.getElementById('quest-list');
            const compEl = document.getElementById('completed-quest-list');
            listEl.innerHTML = ''; compEl.innerHTML = '';
            
            const todayQuests = player.quests.filter(q => q.createdAt === TODAY_STR);
            const active = todayQuests.filter(q => !q.completed);
            
            // Render Active
            if(active.length === 0) {
                document.getElementById('empty-state').classList.remove('hidden');
            } else {
                document.getElementById('empty-state').classList.add('hidden');
                active.forEach(q => {
                    const li = document.createElement('li');
                    li.className = "bg-gray-800 border border-gray-700 rounded-lg p-3 flex items-center justify-between shadow-sm";
                    const rData = RANKS[q.rank] || RANKS['C'];
                    li.innerHTML = `
                        <div class="flex items-center gap-3 flex-1 overflow-hidden">
                            <input type="checkbox" class="quest-checkbox shrink-0" onclick="window.completeQuest('${q.id}', event)">
                            <div class="flex flex-col truncate">
                                <span class="text-sm font-bold text-white truncate">${q.text}</span>
                                <div class="flex gap-2 mt-1 text-[10px]">
                                    <span class="px-1.5 py-0.5 rounded font-bold ${rData.color}">${rData.label}</span>
                                    <span class="text-blue-400">+${rData.exp}</span>
                                    <span class="text-yellow-400">+${rData.gold}G</span>
                                </div>
                            </div>
                        </div>
                        <button onclick="window.abandonQuest('${q.id}')" class="shrink-0 text-gray-500 hover:text-red-500 bg-gray-700 w-8 h-8 rounded-full flex items-center justify-center">
                            <i class="fa-solid fa-flag text-xs"></i>
                        </button>
                    `;
                    listEl.appendChild(li);
                });
            }
            document.getElementById('quest-count').innerText = `${active.length} 건`;

            // Render Completed (All time, latest first)
            const completed = player.quests.filter(q => q.completed).sort((a,b) => b.timestamp - a.timestamp).slice(0, 20); // show last 20
            completed.forEach(q => {
                const li = document.createElement('li');
                li.className = "bg-gray-900 border border-gray-800 rounded p-2 flex items-center gap-2 opacity-60";
                li.innerHTML = `<i class="fa-solid fa-check text-green-500"></i><span class="text-gray-400 line-through truncate text-xs">${q.text}</span>`;
                compEl.appendChild(li);
            });
        }

        // --- Interaction Logic ---
        window.completeQuest = function(id, event) {
            if(event.target.disabled) return;
            event.target.disabled = true;

            const q = player.quests.find(q => q.id === id);
            if(!q) return;
            const rData = RANKS[q.rank];

            player.exp += rData.exp;
            player.gold += rData.gold;
            player.hp = Math.min(player.maxHp, player.hp + (rData.damage * 0.2)); 
            q.completed = true;
            q.timestamp = Date.now();

            const li = event.target.closest('li');
            li.style.transition = 'all 0.4s'; li.style.opacity = '0'; li.style.transform = 'translateX(30px)';

            setTimeout(() => {
                checkLevelUp();
                saveToCloud();
                renderQuests();
                if(!document.getElementById('view-calendar').classList.contains('hidden')) renderCalendar();
            }, 400);
        };

        window.abandonQuest = function(id) {
            const q = player.quests.find(q => q.id === id);
            if(!q) return;
            const rData = RANKS[q.rank];

            showConfirm(`[${q.text}]\n포기 시 체력이 ${rData.damage} 감소합니다.`, () => {
                player.quests = player.quests.filter(item => item.id !== id);
                takeDamage(rData.damage);
                saveToCloud();
                renderQuests();
            });
        };

        function takeDamage(dmg) {
            player.hp -= dmg;
            const pnl = document.getElementById('player-stats-panel');
            pnl.classList.add('shake-animation');
            pnl.style.borderColor = 'red';
            setTimeout(() => { pnl.classList.remove('shake-animation'); pnl.style.borderColor = '#0f3460'; }, 400);

            if(player.hp <= 0) {
                player.hp = player.maxHp;
                let lost = Math.floor(player.gold * 0.3);
                player.gold -= lost;
                showInfo("GAME OVER", "text-red-500", "체력이 모두 소진되어 패널티를 받았습니다.", `
                    <div>잃은 골드: <span class="text-red-400">-${lost} G</span></div>
                `);
            }
        }

        // --- Level Up & Quizzes ---
        function checkLevelUp() {
            let leveled = false;
            let oldLv = player.level;
            
            while(player.exp >= player.maxExp) {
                player.exp -= player.maxExp;
                player.level++;
                player.maxExp = Math.floor(player.maxExp * 1.3);
                player.maxHp += 20;
                player.hp = player.maxHp;
                leveled = true;
            }

            if(leveled) {
                document.getElementById('player-stats-panel').classList.add('level-up-anim');
                setTimeout(() => document.getElementById('player-stats-panel').classList.remove('level-up-anim'), 1500);

                let extraHtml = '';
                if(player.level % 30 === 0) {
                    player.gold += 5000;
                    extraHtml = `<div class="mt-2 text-pink-400 font-bold">🎉 레벨 30 달성 보상: +5000 G</div>`;
                }

                if(globalQuizzes.length > 0) {
                    activeQuiz = globalQuizzes[Math.floor(Math.random() * globalQuizzes.length)];
                    activeQuiz.oldLv = oldLv;
                    activeQuiz.extra = extraHtml;
                    
                    document.getElementById('quiz-question-text').innerText = "Q. " + activeQuiz.question;
                    document.getElementById('quiz-answer-input').value = '';
                    
                    openOverlay();
                    const qModal = document.getElementById('quiz-modal');
                    qModal.classList.remove('hidden');
                    setTimeout(() => qModal.classList.replace('scale-90', 'scale-100'), 10);
                } else {
                    showLevelUpInfo(oldLv, extraHtml);
                }
            }
        }

        document.getElementById('quiz-submit-btn').addEventListener('click', () => {
            const ans = document.getElementById('quiz-answer-input').value.trim();
            const qModal = document.getElementById('quiz-modal');
            qModal.classList.replace('scale-100', 'scale-90');
            setTimeout(() => qModal.classList.add('hidden'), 300);

            let resHtml = '';
            if(ans === activeQuiz.answer) {
                player.gold += Number(activeQuiz.reward);
                resHtml = `<div class="mt-2 text-green-400 font-bold">정답입니다! 보상: +${activeQuiz.reward} G</div>`;
            } else {
                resHtml = `<div class="mt-2 text-red-400 font-bold">오답입니다. (정답: ${activeQuiz.answer})</div>`;
            }

            setTimeout(() => {
                showLevelUpInfo(activeQuiz.oldLv, activeQuiz.extra + resHtml);
                activeQuiz = null;
            }, 300);
        });

        function showLevelUpInfo(old, extra) {
            showInfo("LEVEL UP!", "text-yellow-400", "레벨이 올랐습니다!\n체력이 완전히 회복되었습니다.", `
                <div>레벨: <span class="text-yellow-400 font-bold">${old} ➔ ${player.level}</span></div>
                ${extra}
            `);
            saveToCloud();
        }

        // --- Shop logic ---
        function renderShop() {
            const container = document.getElementById('shop-container');
            container.innerHTML = '';
            globalShop.forEach(item => {
                const div = document.createElement('div');
                div.className = "bg-gray-800 border border-gray-700 rounded-lg p-4 flex flex-col items-center text-center";
                div.innerHTML = `
                    <div class="text-4xl text-purple-400 mb-2"><i class="fa-solid ${item.icon}"></i></div>
                    <div class="text-sm font-bold text-gray-200 mb-1">${item.name}</div>
                    <div class="text-yellow-400 text-xs font-bold mb-3">${item.price} G</div>
                    <button onclick="window.buyItem('${item.id}')" class="w-full bg-gray-700 text-gray-300 text-xs py-2 rounded hover:bg-gray-600 transition-colors">구매요청</button>
                `;
                container.appendChild(div);
            });
        }
        
        window.buyItem = function(id) {
            const item = globalShop.find(i => i.id === id);
            if(player.gold >= item.price) {
                showConfirm(`[${item.name}]\n정말 ${item.price}G 에 구매하시겠습니까?`, () => {
                    player.gold -= item.price;
                    saveToCloud();
                    showInfo("구매 성공!", "text-green-400", "상품을 성공적으로 구매했습니다.\n관리자에게 보상을 요청하세요.", "");
                });
            } else {
                showInfo("골드 부족", "text-red-500", "골드가 부족하여 구매할 수 없습니다.", "");
            }
        };

        // --- Calendar & Tabs ---
        function setupTabs() {
            document.querySelectorAll('[id^="tab-btn-"]').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    const targetViewId = e.currentTarget.id.replace('tab-btn-', 'view-');
                    
                    document.querySelectorAll('.view-section').forEach(v => v.classList.add('hidden'));
                    document.getElementById(targetViewId).classList.remove('hidden');

                    document.querySelectorAll('[id^="tab-btn-"]').forEach(b => {
                        b.className = "flex-1 bg-gray-800 text-gray-400 border-2 border-gray-700 py-2 rounded-lg font-bold shadow-md transition-colors";
                    });
                    
                    e.currentTarget.className = "flex-1 bg-[#0f3460] text-white border-2 border-blue-800 py-2 rounded-lg font-bold shadow-md transition-colors";

                    if(targetViewId === 'view-calendar') renderCalendar();
                });
            });
        }

        function setupCalendarListeners() {
            document.getElementById('cal-prev').addEventListener('click', () => {
                currentCalDate.setMonth(currentCalDate.getMonth() - 1); renderCalendar();
            });
            document.getElementById('cal-next').addEventListener('click', () => {
                currentCalDate.setMonth(currentCalDate.getMonth() + 1); renderCalendar();
            });
        }

        function renderCalendar() {
            const year = currentCalDate.getFullYear();
            const month = currentCalDate.getMonth();
            document.getElementById('cal-month-title').innerText = `${year}년 ${month + 1}월`;
            
            const calBody = document.getElementById('calendar-body');
            calBody.innerHTML = '';
            
            const firstDay = new Date(year, month, 1).getDay();
            const daysInMonth = new Date(year, month + 1, 0).getDate();

            for(let i=0; i<firstDay; i++) {
                const cell = document.createElement('div');
                cell.className = "calendar-cell border-none bg-transparent";
                calBody.appendChild(cell);
            }

            for(let i=1; i<=daysInMonth; i++) {
                const cellDate = new Date(year, month, i);
                const dateStr = getDateString(cellDate);
                
                const cell = document.createElement('div');
                cell.className = "calendar-cell cursor-pointer hover:bg-gray-800 transition-colors";
                if(dateStr === TODAY_STR) cell.classList.add('today');
                if(dateStr === selectedCalDateStr) cell.style.borderColor = '#e94560';

                cell.innerHTML = `<div class="calendar-date-num">${i}</div>`;

                const dailyQs = player.quests.filter(q => q.createdAt === dateStr);
                if(dailyQs.length > 0) {
                    const total = dailyQs.length;
                    const comp = dailyQs.filter(q => q.completed).length;
                    const isPast = cellDate < new Date(new Date().setHours(0,0,0,0));
                    
                    const resDiv = document.createElement('div');
                    resDiv.className = "text-[10px] md:text-xs font-bold mt-2";
                    
                    if(comp === total) {
                        resDiv.innerText = '최고야!'; resDiv.classList.add('text-blue-400');
                    } else if(comp > 0) {
                        resDiv.innerText = '잘했어!'; resDiv.classList.add('text-green-400');
                    } else if (isPast) {
                        resDiv.innerText = '도전해!'; resDiv.classList.add('text-red-400');
                    } else {
                        resDiv.innerText = '진행중'; resDiv.classList.add('text-gray-400');
                    }
                    cell.appendChild(resDiv);
                }

                cell.addEventListener('click', () => {
                    selectedCalDateStr = dateStr;
                    renderCalendar();
                    renderCalendarDetail(dateStr);
                });

                calBody.appendChild(cell);
            }
        }

        function renderCalendarDetail(dateStr) {
            document.getElementById('cal-detail-title').innerText = `${dateStr} 미션 리스트`;
            const listEl = document.getElementById('cal-detail-list');
            listEl.innerHTML = '';
            
            const daily = player.quests.filter(q => q.createdAt === dateStr);
            if(daily.length === 0) {
                listEl.innerHTML = '<li class="text-gray-500 text-center py-2 text-xs">해당 날짜의 미션이 없습니다.</li>';
                return;
            }
            
            daily.forEach(q => {
                const li = document.createElement('li');
                li.className = "flex justify-between border-b border-gray-700 py-1";
                const stat = q.completed ? '<span class="text-green-400 font-bold text-xs">완료</span>' : '<span class="text-red-400 font-bold text-xs">미완료</span>';
                li.innerHTML = `<span class="text-gray-300">${q.text}</span> ${stat}`;
                listEl.appendChild(li);
            });
        }

        // --- Modals ---
        function openOverlay() {
            const ov = document.getElementById('modal-overlay');
            ov.classList.remove('hidden');
            setTimeout(() => { ov.classList.remove('opacity-0'); ov.classList.add('opacity-100'); }, 10);
        }

        function closeAllModals() {
            const ov = document.getElementById('modal-overlay');
            ov.classList.remove('opacity-100'); ov.classList.add('opacity-0');
            ['info-modal', 'confirm-modal', 'nickname-modal', 'quiz-modal'].forEach(id => {
                const m = document.getElementById(id);
                m.classList.remove('scale-100'); m.classList.add('scale-90');
            });
            setTimeout(() => {
                ov.classList.add('hidden');
                ['info-modal', 'confirm-modal', 'nickname-modal', 'quiz-modal'].forEach(id => document.getElementById(id).classList.add('hidden'));
            }, 300);
        }

        function showInfo(title, colorCls, desc, statsHtml) {
            document.getElementById('info-modal-title').innerText = title;
            document.getElementById('info-modal-title').className = `text-3xl font-bold mb-3 font-retro ${colorCls}`;
            document.getElementById('info-modal-desc').innerText = desc;
            document.getElementById('info-modal-stats').innerHTML = statsHtml;
            openOverlay();
            const m = document.getElementById('info-modal');
            m.classList.remove('hidden');
            setTimeout(() => m.classList.replace('scale-90', 'scale-100'), 10);
        }

        function showConfirm(desc, cb) {
            document.getElementById('confirm-modal-desc').innerText = desc;
            confirmCallback = cb;
            openOverlay();
            const m = document.getElementById('confirm-modal');
            m.classList.remove('hidden');
            setTimeout(() => m.classList.replace('scale-90', 'scale-100'), 10);
        }

        document.getElementById('info-modal-close').addEventListener('click', closeAllModals);
        document.getElementById('confirm-modal-cancel').addEventListener('click', closeAllModals);
        document.getElementById('confirm-modal-ok').addEventListener('click', () => { if(confirmCallback) confirmCallback(); closeAllModals(); });

        window.onload = init;
    </script>
</body>
</html>
```
