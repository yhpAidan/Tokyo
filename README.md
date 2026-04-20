<html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>2026 踐踏日本國土之旅 | 雲端同步管家</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    
    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = { 
            apiKey: "AIzaSyDhNTj3_fzYAToqq3X3IrC-FRkDBlWZShM", 
            authDomain: "tokyo-f003f.firebaseapp.com", 
            projectId: "tokyo-f003f", 
            storageBucket: "tokyo-f003f.firebasestorage.app", 
            messagingSenderId: "962379464999", 
            appId: "1:962379464999:web:8776eae28424d0b0e1a902" 
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        
        const appId = "tokyo-trip-2026-shared"; 

        window.travelDB = { db, auth, appId, doc, getDoc, setDoc, onSnapshot };
        
        signInAnonymously(auth).catch((error) => {
            console.error("Firebase 登入失敗:", error);
        });
    </script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@400;700&family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
        :root { --wa-red: #d93535; --wa-blue: #2a4073; --wa-gold: #c5a059; --wa-bg: #fdfaf5; }
        body { font-family: 'Noto Sans TC', sans-serif; background-color: var(--wa-bg); color: #2d2d2d; overflow-x: hidden; }
        h1, h2, .font-serif { font-family: 'Noto Serif TC', serif; }
        
        #sidebar { transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1); z-index: 200; }
        #sidebar.closed { transform: translateX(-100%); }
        
        .page-content { display: none; opacity: 0; transform: translateY(15px); transition: all 0.4s ease-out; }
        .page-content.active { display: block; opacity: 1; transform: translateY(0); }
        
        .interactive-btn { transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); }
        .interactive-btn:hover { transform: scale(1.05); }
        .interactive-btn:active { transform: scale(0.95); }

        .menu-item.active { background: var(--wa-blue) !important; color: white !important; box-shadow: 0 10px 15px -3px rgba(42, 64, 115, 0.3); }
        .menu-item.active span:first-child { background: rgba(255,255,255,0.2) !important; color: white !important; }

        .nav-tab.active { box-shadow: 0 8px 20px rgba(42, 64, 115, 0.4); }

        .editable { outline: none; border-bottom: 1px dashed transparent; }
        .editable:hover { background-color: rgba(197, 160, 89, 0.08); border-radius: 4px; }
        
        #loading-overlay { position: fixed; inset: 0; background: white; z-index: 1000; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        
        .dragging { opacity: 0.5; transform: scale(0.98); border: 2px dashed var(--wa-gold) !important; }
        
        .toolbox-nav-btn { position: relative; cursor: pointer; }
        .toolbox-nav-btn.active { color: var(--wa-blue); }
        .toolbox-nav-btn.active::after { content: ''; position: absolute; bottom: 0; left: 0; width: 100%; h: 2px; background: var(--wa-blue); border-radius: 2px; }

        @keyframes popIn { from { opacity: 0; transform: scale(0.9); } to { opacity: 1; transform: scale(1); } }
        .pop-in { animation: popIn 0.2s ease-out forwards; }
    </style>
</head>
<body class="pb-32">

    <div id="loading-overlay">
        <div class="w-12 h-12 border-4 border-wa-blue border-t-transparent rounded-full animate-spin"></div>
        <p class="mt-4 text-[10px] font-bold text-slate-400 tracking-widest uppercase">載入雲端資產中</p>
    </div>

    <!-- 側邊選單 -->
    <div id="sidebar-overlay" class="fixed inset-0 bg-black/40 hidden opacity-0 z-[190]" onclick="toggleSidebar()"></div>
    <aside id="sidebar" class="fixed top-0 left-0 h-full w-72 bg-white shadow-2xl flex flex-col closed">
        <div class="p-8 border-b border-slate-50">
            <h2 class="text-2xl font-bold text-wa-blue font-serif">旅程清單</h2>
            <p class="text-[10px] text-slate-400 mt-1 uppercase tracking-widest font-bold">2026 TOKYO ITINERARY</p>
        </div>
        <div class="flex-1 overflow-y-auto p-4 space-y-2 no-scrollbar" id="side-nav-list"></div>
    </aside>

    <!-- 頂部導航 -->
    <header class="bg-white/90 backdrop-blur-md border-b border-slate-100 sticky top-0 z-[100] h-30 flex items-center px-4 shadow-sm">
        <button onclick="toggleSidebar()" class="interactive-btn w-10 h-10 flex items-center justify-center text-slate-600 rounded-full hover:bg-slate-50">
            <i class="fa-solid fa-bars-staggered"></i>
        </button>
        <div class="flex-1 px-4 overflow-hidden text-center">
            <h1 id="main-title" class="text-base font-bold text-slate-800 editable truncate" contenteditable="true">踐踏日本國土之旅</h1>
            <p id="top-subtitle" class="text-[10px] text-wa-gold font-bold uppercase tracking-wider">Day 1 · 5/21</p>
        </div>
        <button id="save-all-btn" onclick="saveAllToCloud()" class="interactive-btn w-10 h-10 flex items-center justify-center text-emerald-500 rounded-full bg-emerald-50 shadow-sm border border-emerald-100">
            <i class="fa-solid fa-cloud-arrow-up"></i>
        </button>
    </header>

    <main class="max-w-md mx-auto p-4 pt-6">
        <div id="page-itinerary" class="page-content active">
            <div id="itinerary-wrapper"></div>
        </div>

        <!-- 工具箱頁面 -->
        <div id="page-toolbox" class="page-content space-y-6">
            <div class="flex items-center gap-4 overflow-x-auto no-scrollbar border-b border-slate-100 pb-2 mb-4 sticky top-16 bg-white/80 backdrop-blur z-10">
                <button onclick="filterToolbox('all')" class="toolbox-nav-btn active px-4 py-2 text-sm font-bold transition-all" data-filter="all">全部</button>
                <button onclick="filterToolbox('flight')" class="toolbox-nav-btn px-4 py-2 text-sm font-bold text-slate-400 transition-all" data-filter="flight">航班</button>
                <button onclick="filterToolbox('hotel')" class="toolbox-nav-btn px-4 py-2 text-sm font-bold text-slate-400 transition-all" data-filter="hotel">住宿</button>
                <button onclick="filterToolbox('activity')" class="toolbox-nav-btn px-4 py-2 text-sm font-bold text-slate-400 transition-all" data-filter="activity">活動</button>
                <button onclick="filterToolbox('budget')" class="toolbox-nav-btn px-4 py-2 text-sm font-bold text-slate-400 transition-all" data-filter="budget">預算</button>
            </div>
            
            <div id="toolbox-list" class="space-y-4 min-h-[100px]"></div>
            
            <div class="grid grid-cols-2 gap-3 pt-6">
                <button onclick="addToolboxItem('flight')" class="p-4 bg-blue-50 text-blue-600 rounded-2xl text-xs font-bold interactive-btn border border-blue-100">+ 航班資訊</button>
                <button onclick="addToolboxItem('hotel')" class="p-4 bg-indigo-50 text-indigo-600 rounded-2xl text-xs font-bold interactive-btn border border-indigo-100">+ 住宿安排</button>
                <button onclick="addToolboxItem('activity')" class="p-4 bg-emerald-50 text-emerald-600 rounded-2xl text-xs font-bold interactive-btn border border-emerald-100">+ 其他活動</button>
                <button onclick="addToolboxItem('budget')" class="p-4 bg-amber-50 text-amber-600 rounded-2xl text-xs font-bold interactive-btn border border-amber-100">+ 預算追蹤</button>
            </div>
        </div>
    </main>

    <!-- 底部導航 -->
    <nav class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[85%] max-w-sm bg-slate-900/90 backdrop-blur-xl rounded-full p-2 flex justify-around items-center shadow-2xl z-[150] border border-white/10">
        <button onclick="switchPage('itinerary')" id="nav-itinerary" class="nav-tab active flex items-center gap-3 px-6 py-3 rounded-full transition-all bg-wa-blue text-white">
            <i class="fa-solid fa-map-location-dot text-lg"></i>
            <span class="text-xs font-bold">行程</span>
        </button>
        <button onclick="switchPage('toolbox')" id="nav-toolbox" class="nav-tab flex items-center gap-3 px-6 py-3 rounded-full transition-all text-slate-400">
            <i class="fa-solid fa-toolbox text-lg"></i>
            <span class="text-xs font-bold">工具箱</span>
        </button>
    </nav>

    <!-- 分類編輯器 -->
    <div id="cat-editor-modal" class="fixed inset-0 z-[400] hidden flex items-center justify-center bg-black/40 backdrop-blur-sm">
        <div class="bg-white p-6 rounded-[2.5rem] shadow-2xl w-[280px] pop-in space-y-6">
            <div class="text-center">
                <p class="text-[10px] font-black text-slate-300 uppercase tracking-widest mb-4">編輯行程分類</p>
                <input type="text" id="cat-name-input" class="w-full text-center text-lg font-bold text-slate-800 border-b-2 border-slate-100 pb-2 focus:border-wa-blue outline-none" placeholder="分類名稱">
            </div>
            <div class="grid grid-cols-4 gap-4 justify-items-center">
                <div class="w-8 h-8 rounded-full bg-wa-red cursor-pointer" data-color="#d93535" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-wa-blue cursor-pointer" data-color="#2a4073" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-emerald-500 cursor-pointer" data-color="#10b981" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-amber-500 cursor-pointer" data-color="#f59e0b" onclick="selectCatColor(this)"></div>
            </div>
            <div class="flex gap-2">
                <button onclick="closeCatEditor()" class="flex-1 py-3 text-xs font-bold text-slate-400 bg-slate-50 rounded-2xl">取消</button>
                <button onclick="applyCatChanges()" class="flex-1 py-3 text-xs font-bold text-slate-400 bg-slate-50 rounded-2xl">確定</button>
            </div>
        </div>
    </div>

    <input type="file" id="global-uploader" class="hidden" onchange="handleFileSelect(event)">
    <div id="save-toast" class="fixed top-8 left-1/2 -translate-x-1/2 bg-slate-900 text-white px-6 py-3 rounded-full shadow-2xl z-[200] opacity-0 pointer-events-none transition-all duration-500">
        <span class="text-xs font-bold tracking-widest uppercase">同步成功</span>
    </div>

    <script>
        const tripDates = ["5/21", "5/22", "5/23", "5/24", "5/25", "5/26", "5/27", "5/28", "5/29"];
        
        // 預填行程資料
        let appState = {
            mainTitle: "踐踏日本國土之旅",
            itineraries: {
                1: [
                    { cat: "航班", type: "flight", time: "--:--", title: "GK28 HKG -> NRT", note: "香港飛往成田機場", color: "#2a4073" },
                    { cat: "交通", type: "activity", time: "--:--", title: "機場巴士/京成線去新宿", note: "車程約 1.5h", color: "#94a3b8" },
                    { cat: "景點", type: "activity", time: "下午", title: "新宿觀光購物", note: "東京都廳展望台、歌舞伎町、伊勢丹/高島屋", color: "#10b981" },
                    { cat: "景點", type: "activity", time: "晚上", title: "涉谷觀光", note: "涉谷十字路口、澀谷Sky、逛街吃飯", color: "#10b981" },
                    { cat: "住宿", type: "hotel", time: "--:--", title: "新宿住宿", note: "待定", color: "#2a4073" }
                ],
                2: [
                    { cat: "景點", type: "activity", time: "--:--", title: "淺草全日購物逛街", note: "淺草寺、仲見世街、晴空塔、隅田川散步", color: "#10b981" },
                    { cat: "景點", type: "activity", time: "下午", title: "阿美橫丁/上野", note: "逛街購物", color: "#10b981" },
                    { cat: "住宿", type: "hotel", time: "--:--", title: "新宿住宿", note: "待定", color: "#2a4073" }
                ],
                3: [
                    { cat: "交通", type: "activity", time: "早上", title: "JR 去鎌倉", note: "新宿出發", color: "#94a3b8" },
                    { cat: "景點", type: "activity", time: "上午", title: "鎌倉觀光", note: "大佛、長谷寺、鶴岡八幡宮", color: "#10b981" },
                    { cat: "景點", type: "activity", time: "下午", title: "小町通/若宮大路", note: "工藝品、甜點購物", color: "#10b981" },
                    { cat: "住宿", type: "hotel", time: "--:--", title: "新宿住宿", note: "待定", color: "#2a4073" }
                ],
                4: [
                    { cat: "交通", type: "activity", time: "早上", title: "租車自駕去伊豆", note: "東京租車，開車約 1.5-2h", color: "#94a3b8" },
                    { cat: "景點", type: "activity", time: "--:--", title: "大室山 / 城崎海岸", note: "三島天空步道 (選填)", color: "#10b981" },
                    { cat: "活動", type: "activity", time: "20:20", title: "熱海海上花火大會", note: "Sun Beach 沙灘區佔位觀賞", color: "#f59e0b" },
                    { cat: "住宿", type: "hotel", time: "晚上", title: "登坂酒店 (THE NOBORISAKA)", note: "河口湖附近住宿", color: "#2a4073" }
                ],
                5: [
                    { cat: "活動", type: "activity", time: "早上", title: "富士急樂園", note: "雲霄飛車、Thomas Land", color: "#f59e0b" },
                    { cat: "景點", type: "activity", time: "下午", title: "河口湖睇富士山", note: "觀光拍照", color: "#10b981" },
                    { cat: "住宿", type: "hotel", time: "晚上", title: "登坂酒店 (THE NOBORISAKA)", note: "續住", color: "#2a4073" }
                ],
                6: [
                    { cat: "交通", type: "activity", time: "早上", title: "河口湖 -> 名古屋", note: "長途行車", color: "#94a3b8" },
                    { cat: "景點", type: "activity", time: "下午", title: "大須商店街", note: "名古屋逛街", color: "#10b981" },
                    { cat: "住宿", type: "hotel", time: "--:--", title: "名古屋住宿", note: "待定", color: "#2a4073" }
                ],
                7: [
                    { cat: "景點", type: "activity", time: "上午", title: "白川鄉合掌村", note: "展望台拍照、村落散步", color: "#10b981" },
                    { cat: "景點", type: "activity", time: "下午", title: "立山黑部全日", note: "雪之大谷、室堂、黑部水壩", color: "#10b981" },
                    { cat: "住宿", type: "hotel", time: "--:--", title: "名古屋住宿", note: "待定", color: "#2a4073" }
                ],
                8: [
                    { cat: "活動", type: "activity", time: "中午", title: "吉卜力樂園", note: "龍貓森林、魔女之谷", color: "#f59e0b" },
                    { cat: "景點", type: "activity", time: "晚上", title: "名古屋城/大須商店街", note: "晚餐吃鰻魚飯", color: "#10b981" },
                    { cat: "住宿", type: "hotel", time: "--:--", title: "名古屋住宿", note: "待定", color: "#2a4073" }
                ],
                9: [
                    { cat: "景點", type: "activity", time: "早上", title: "市川市動物園", note: "開車前往", color: "#10b981" },
                    { cat: "航班", type: "flight", time: "--:--", title: "GK29 NRT -> HKG", note: "還車後登機返港", color: "#2a4073" }
                ]
            },
            toolbox: []
        };

        let currentDay = 1;
        let isSaving = false;
        let editingCatElement = null;
        let selectedHex = "#94a3b8";

        function getDocRef() {
            const { db, appId, doc } = window.travelDB;
            return doc(db, 'artifacts', appId, 'public', 'data', 'itinerary', 'current');
        }

        async function initCloud() {
            const checkFirebase = setInterval(async () => {
                if (window.travelDB && window.travelDB.auth.currentUser) {
                    clearInterval(checkFirebase);
                    try {
                        const { getDoc } = window.travelDB;
                        const docSnap = await getDoc(getDocRef());
                        if (docSnap.exists()) {
                            // 這裡可以選擇合併雲端資料或使用預設資料
                            // appState = docSnap.data(); 
                        }
                    } catch (e) { console.error("讀取失敗:", e); }
                    hideLoading();
                    initializeAppUI();
                }
            }, 500);
        }

        function hideLoading() {
            const overlay = document.getElementById('loading-overlay');
            if (overlay) { overlay.style.opacity = '0'; setTimeout(() => overlay.remove(), 500); }
        }

        function initializeAppUI() {
            document.getElementById('main-title').innerText = appState.mainTitle || "2026 日本深度遊";
            renderSidebar();
            renderAllItineraries();
            renderToolbox();
            updateHeader(1);
        }

        async function saveAllToCloud() {
            if (isSaving || !window.travelDB) return;
            isSaving = true;
            const saveBtn = document.getElementById('save-all-btn');
            saveBtn.innerHTML = '<i class="fa-solid fa-spinner animate-spin"></i>';

            appState.mainTitle = document.getElementById('main-title').innerText;
            tripDates.forEach((_, i) => {
                const day = i + 1;
                const list = document.querySelector(`#day-view-${day} .itinerary-list`);
                if (list) appState.itineraries[day] = Array.from(list.querySelectorAll('.draggable-item')).map(card => getCardData(card));
            });
            const toolboxList = document.getElementById('toolbox-list');
            appState.toolbox = Array.from(toolboxList.querySelectorAll('.draggable-item')).map(card => getCardData(card));

            try {
                const { setDoc } = window.travelDB;
                await setDoc(getDocRef(), appState);
                showToast("雲端同步成功");
            } catch (e) { showToast("同步失敗"); }
            finally { isSaving = false; saveBtn.innerHTML = '<i class="fa-solid fa-cloud-arrow-up"></i>'; }
        }

        function getCardData(card) {
            const catEl = card.querySelector('.cat-tag');
            return {
                cat: catEl.innerText,
                time: card.querySelector('.time-text')?.innerText || '--:--',
                title: card.querySelector('h3').innerText,
                note: card.querySelector('p').innerText,
                color: catEl.style.backgroundColor,
                type: card.dataset.type || 'none',
                files: Array.from(card.querySelectorAll('.file-tag span')).map(s => s.innerText),
                budgetItems: card.dataset.type === 'budget' ? Array.from(card.querySelectorAll('.budget-item')).map(row => ({
                    name: row.querySelector('.b-name').innerText,
                    amount: parseFloat(row.querySelector('.b-amount').innerText) || 0
                })) : []
            };
        }

        function createCard(data) {
            const div = document.createElement('div');
            div.className = `bg-white rounded-[2.2rem] shadow-sm border border-slate-50 p-6 space-y-4 relative draggable-item`;
            div.draggable = true;
            div.dataset.type = data.type || 'none';
            
            const isBudget = data.type === 'budget';
            const showNav = (data.type === 'hotel' || data.type === 'activity' || data.type === 'flight');

            div.innerHTML = `
                <div class="absolute top-6 right-6 text-slate-100 cursor-grab"><i class="fa-solid fa-grip-vertical"></i></div>
                <div class="flex justify-between items-center pr-8">
                    <span onclick="openCatEditor(this)" class="cat-tag text-[9px] font-black text-white px-3 py-1 rounded-full cursor-pointer" style="background-color: ${data.color || '#94a3b8'}">${data.cat}</span>
                    <span class="time-text text-[10px] text-slate-300 font-black font-mono editable" contenteditable="true">${data.time || '--:--'}</span>
                </div>
                <div>
                    <h3 class="text-lg font-bold text-slate-800 editable" contenteditable="true">${data.title}</h3>
                    <p class="text-xs text-slate-400 mt-2 leading-relaxed editable" contenteditable="true">${data.note}</p>
                </div>
                ${isBudget ? `<div class="bg-slate-50 rounded-2xl p-4 space-y-2">
                    <div class="budget-list space-y-2"></div>
                    <button onclick="addBudgetItem(this)" class="w-full py-2 text-[10px] font-bold text-slate-400 border border-dashed border-slate-200 rounded-lg">+ 新增支出</button>
                    <div class="pt-2 border-t border-slate-200 flex justify-between font-bold text-slate-800 text-xs"><span>總計</span><span class="total-amount">0</span></div>
                </div>` : ''}
                <div class="flex gap-2 pt-1">
                    <button onclick="triggerUpload(this)" class="flex-1 py-2 bg-slate-50 text-slate-400 rounded-xl text-[10px] font-bold">上傳</button>
                    ${showNav ? `<button onclick="openMap(this)" class="flex-1 py-2 bg-blue-50 text-blue-600 rounded-xl text-[10px] font-bold">導航</button>` : ''}
                    <button onclick="this.closest('.draggable-item').remove()" class="w-8 h-8 bg-rose-50 text-rose-400 rounded-xl text-xs flex items-center justify-center"><i class="fa-solid fa-trash-can"></i></button>
                </div>
                <div class="file-container flex flex-wrap gap-2"></div>
            `;

            if (isBudget) {
                const list = div.querySelector('.budget-list');
                (data.budgetItems || []).forEach(bi => list.appendChild(createBudgetItemRow(bi.name, bi.amount)));
                updateBudgetTotal(div);
            }
            if (data.files) data.files.forEach(name => div.querySelector('.file-container').appendChild(createFileTag(name)));
            
            addDragEvents(div);
            return div;
        }

        function filterToolbox(type) {
            document.querySelectorAll('.toolbox-nav-btn').forEach(btn => {
                if (btn.dataset.filter === type) {
                    btn.classList.add('active', 'text-wa-blue');
                    btn.classList.remove('text-slate-400');
                } else {
                    btn.classList.remove('active', 'text-wa-blue');
                    btn.classList.add('text-slate-400');
                }
            });
            const cards = document.querySelectorAll('#toolbox-list .draggable-item');
            cards.forEach(card => {
                if (type === 'all' || card.dataset.type === type) {
                    card.style.display = 'block';
                    card.classList.add('pop-in');
                } else {
                    card.style.display = 'none';
                    card.classList.remove('pop-in');
                }
            });
        }

        function renderSidebar() {
            const list = document.getElementById('side-nav-list');
            list.innerHTML = '';
            tripDates.forEach((date, i) => {
                const dayNum = i + 1;
                const btn = document.createElement('button');
                btn.className = `menu-item flex items-center gap-4 w-full p-4 rounded-2xl text-sm font-bold transition-all ${dayNum === currentDay ? 'active' : 'text-slate-600 bg-transparent'}`;
                btn.id = `side-nav-day-${dayNum}`;
                btn.innerHTML = `<span class="w-10 h-10 rounded-full flex items-center justify-center text-xs ${dayNum === currentDay ? 'bg-white/20 text-white' : 'bg-slate-100 text-slate-400'}">D${dayNum}</span><span>${date} 行程</span>`;
                btn.onclick = () => { 
                    updateActiveSidebarItem(dayNum);
                    currentDay = dayNum; 
                    showDay(dayNum); 
                    toggleSidebar(); 
                };
                list.appendChild(btn);
            });
        }

        // 核心修正：點擊時更新 Highlight 狀態
        function updateActiveSidebarItem(dayNum) {
            document.querySelectorAll('.menu-item').forEach(btn => {
                btn.classList.remove('active');
                btn.classList.add('text-slate-600', 'bg-transparent');
                const span = btn.querySelector('span:first-child');
                span.classList.remove('bg-white/20', 'text-white');
                span.classList.add('bg-slate-100', 'text-slate-400');
            });
            const activeBtn = document.getElementById(`side-nav-day-${dayNum}`);
            if (activeBtn) {
                activeBtn.classList.add('active');
                activeBtn.classList.remove('text-slate-600', 'bg-transparent');
                const span = activeBtn.querySelector('span:first-child');
                span.classList.add('bg-white/20', 'text-white');
                span.classList.remove('bg-slate-100', 'text-slate-400');
            }
        }

        function renderAllItineraries() {
            const wrapper = document.getElementById('itinerary-wrapper');
            wrapper.innerHTML = '';
            tripDates.forEach((_, i) => {
                const dayNum = i + 1;
                const section = document.createElement('div');
                section.id = `day-view-${dayNum}`;
                section.className = `day-view space-y-4 ${dayNum === currentDay ? 'block' : 'hidden'}`;
                section.innerHTML = `<div class="itinerary-list space-y-4" ondragover="handleDragOver(event)"></div><button onclick="addCard(${dayNum})" class="w-full py-5 border-2 border-dashed border-slate-100 rounded-[2.5rem] text-slate-300 text-xs font-bold interactive-btn">+ 新增 D${dayNum} 行程</button>`;
                (appState.itineraries[dayNum] || []).forEach(data => section.querySelector('.itinerary-list').appendChild(createCard(data)));
                wrapper.appendChild(section);
            });
        }

        function renderToolbox() {
            const list = document.getElementById('toolbox-list');
            list.innerHTML = '';
            (appState.toolbox || []).forEach(data => list.appendChild(createCard(data)));
            filterToolbox('all');
        }

        function showDay(dayNum) {
            document.querySelectorAll('.day-view').forEach(v => v.classList.add('hidden'));
            document.getElementById(`day-view-${dayNum}`).classList.remove('hidden');
            updateHeader(dayNum);
            switchPage('itinerary');
        }

        function updateHeader(dayNum) { 
            document.getElementById('top-subtitle').innerText = `Day ${dayNum} · ${tripDates[dayNum-1]}`; 
        }

        function switchPage(page) {
            document.querySelectorAll('.page-content').forEach(p => p.classList.remove('active'));
            document.querySelectorAll('.nav-tab').forEach(t => {
                t.classList.remove('bg-wa-blue', 'text-white');
                t.classList.add('text-slate-400');
            });
            document.getElementById('page-' + page).classList.add('active');
            document.getElementById('nav-' + page).classList.add('bg-wa-blue', 'text-white');
        }

        function addToolboxItem(type) {
            const list = document.getElementById('toolbox-list');
            const data = { cat: "新增項目", type, title: "點擊編輯", note: "細節...", color: "#94a3b8" };
            const card = createCard(data);
            list.insertBefore(card, list.firstChild);
            filterToolbox('all');
        }

        function addCard(dayNum) {
            const list = document.querySelector(`#day-view-${dayNum} .itinerary-list`);
            list.appendChild(createCard({ cat: "新行程", time: "--:--", title: "行程標題", note: "描述...", color: "#94a3b8" }));
        }

        function openCatEditor(el) { editingCatElement = el; selectedHex = el.style.backgroundColor; document.getElementById('cat-name-input').value = el.innerText; document.getElementById('cat-editor-modal').classList.remove('hidden'); }
        function selectCatColor(el) { selectedHex = el.dataset.color; }
        function closeCatEditor() { document.getElementById('cat-editor-modal').classList.add('hidden'); }
        function applyCatChanges() { if(editingCatElement){ editingCatElement.innerText = document.getElementById('cat-name-input').value; editingCatElement.style.backgroundColor = selectedHex; } closeCatEditor(); }
        
        function updateBudgetTotal(card) {
            let t = 0; card.querySelectorAll('.b-amount').forEach(a => t += parseFloat(a.innerText) || 0);
            card.querySelector('.total-amount').innerText = t.toLocaleString();
        }
        function createBudgetItemRow(name="新項目", amount=0) {
            const d = document.createElement('div'); d.className = "budget-item flex justify-between text-[11px] py-1";
            d.innerHTML = `<span class="b-name editable flex-1" contenteditable="true">${name}</span><span class="b-amount editable text-right font-bold" contenteditable="true" oninput="updateBudgetTotal(this.closest('.draggable-item'))">${amount}</span>`;
            return d;
        }
        function addBudgetItem(btn) { const c = btn.closest('.draggable-item'); c.querySelector('.budget-list').appendChild(createBudgetItemRow()); updateBudgetTotal(c); }

        function toggleSidebar() { 
            const s = document.getElementById('sidebar'); const o = document.getElementById('sidebar-overlay');
            if(s.classList.contains('closed')){ s.classList.remove('closed'); o.classList.remove('hidden'); setTimeout(()=>o.classList.add('opacity-100'),10); }
            else { s.classList.add('closed'); o.classList.remove('opacity-100'); setTimeout(()=>o.classList.add('hidden'),300); }
        }

        let dragElement = null;
        function addDragEvents(el) {
            el.addEventListener('dragstart', (e) => { dragElement = el; setTimeout(() => el.classList.add('dragging'), 0); });
            el.addEventListener('dragend', () => { el.classList.remove('dragging'); dragElement = null; });
        }
        function handleDragOver(e) {
            e.preventDefault();
            const list = e.currentTarget;
            const elements = [...list.querySelectorAll('.draggable-item:not(.dragging)')];
            const next = elements.find(el => e.clientY < el.getBoundingClientRect().top + el.getBoundingClientRect().height / 2);
            list.insertBefore(dragElement, next);
        }

        function triggerUpload(btn) { window.currentFileTarget = btn.closest('.draggable-item').querySelector('.file-container'); document.getElementById('global-uploader').click(); }
        function handleFileSelect(e) { if (e.target.files[0]) window.currentFileTarget.appendChild(createFileTag(e.target.files[0].name)); }
        function createFileTag(name) {
            const t = document.createElement('div'); t.className = 'file-tag text-[10px] bg-slate-50 p-2 rounded-lg flex items-center gap-2';
            t.innerHTML = `<i class="fa-solid fa-file"></i> <span>${name}</span> <i class="fa-solid fa-xmark opacity-30 cursor-pointer" onclick="this.parentElement.remove()"></i>`;
            return t;
        }
        function showToast(msg) {
            const t = document.getElementById('save-toast'); t.querySelector('span').innerText = msg;
            t.classList.replace('opacity-0', 'opacity-100'); setTimeout(() => t.classList.replace('opacity-100', 'opacity-0'), 2000);
        }
        function openMap(btn) { const c = btn.closest('.draggable-item'); window.open(`https://www.google.com/maps/search/${encodeURIComponent(c.querySelector('h3').innerText)}`); }

        window.onload = initCloud;
    </script>
</body>
</html>
