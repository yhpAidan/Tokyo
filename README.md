<html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>2026 踐踏日本國土之旅 | 雲端同步管家</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
        
        :root { 
            --wa-red: #ff4d6d; 
            --wa-blue: #2a4073; 
            --wa-gold: #c5a059; 
            --wa-bg: #fdfaf6;
            --wa-sakura: #ffebf0;
            --wa-teal: #0d9488;
        }

        body { 
            font-family: 'Noto Sans TC', sans-serif; 
            background-color: var(--wa-bg); 
            background-image: url('https://www.transparenttextures.com/patterns/paper-fibers.png');
            color: #1a1a1a;
            -webkit-tap-highlight-color: transparent;
        }

        .interactive-btn { transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1); cursor: pointer; }
        .interactive-btn:hover { transform: scale(1.05); filter: brightness(1.05); }
        .interactive-btn:active { transform: scale(0.95); }

        #sidebar { transition: transform 0.3s ease; z-index: 200; }
        #sidebar.closed { transform: translateX(-100%); }
        .menu-item.active { background: var(--wa-blue) !important; color: white !important; box-shadow: 0 8px 15px -3px rgba(42, 64, 115, 0.3); }

        .timeline-line {
            position: absolute;
            left: 55px;
            top: 0;
            bottom: 0;
            width: 2px;
            background: repeating-linear-gradient(to bottom, #e2e8f0, #e2e8f0 4px, transparent 4px, transparent 8px);
            z-index: 0;
        }

        .dragging { opacity: 0.5; transform: rotate(2deg); border: 2px dashed var(--wa-gold) !important; }
        .editable:focus { outline: none; border-bottom: 2px solid var(--wa-gold); background: #fffcf0; }

        .illustration-fuji {
            position: fixed;
            bottom: -20px;
            right: -20px;
            width: 150px;
            opacity: 0.1;
            pointer-events: none;
            z-index: 0;
        }

        .toolbox-nav-btn.active { color: var(--wa-blue); border-bottom: 3px solid var(--wa-blue); }
        
        /* 底部按鈕美化樣式 */
        .nav-btn-active-itinerary { background: var(--wa-teal) !important; color: white !important; box-shadow: 0 10px 20px -5px rgba(13, 148, 136, 0.4); }
        .nav-btn-active-toolbox { background: var(--wa-blue) !important; color: white !important; box-shadow: 0 10px 20px -5px rgba(42, 64, 115, 0.4); }
    </style>
</head>
<body class="pb-32">

    <div id="loading-overlay" class="fixed inset-0 bg-white z-[1000] flex flex-col items-center justify-center">
        <div class="w-12 h-12 border-4 border-rose-400 border-t-transparent rounded-full animate-spin"></div>
        <p class="mt-4 text-xs font-bold text-slate-400 tracking-widest">正在載入 2026 日本旅程...</p>
    </div>

    <div id="sidebar-overlay" class="fixed inset-0 bg-black/40 hidden z-[190]" onclick="toggleSidebar()"></div>
    <aside id="sidebar" class="fixed top-0 left-0 h-full w-72 bg-white shadow-2xl flex flex-col closed">
        <div class="p-8 border-b border-slate-50 flex justify-between items-center bg-rose-50">
            <div>
                <h2 class="text-xl font-bold text-wa-blue">旅程目錄</h2>
                <p class="text-[10px] text-rose-400 mt-1 uppercase font-black tracking-widest">9-Day TOKYO Itinerary</p>
            </div>
            <button onclick="toggleSidebar()" class="text-slate-300 hover:text-wa-red transition-colors"><i class="fa-solid fa-xmark"></i></button>
        </div>
        <div class="flex-1 overflow-y-auto p-4 space-y-2" id="side-nav-list"></div>
    </aside>

    <header class="bg-white/90 backdrop-blur-md sticky top-0 z-[100] h-30 flex items-center px-4 border-b border-rose-100 shadow-sm">
        <button onclick="toggleSidebar()" class="interactive-btn w-10 h-10 flex items-center justify-center text-wa-blue rounded-xl hover:bg-rose-50">
            <i class="fa-solid fa-bars-staggered"></i>
        </button>
        <div class="flex-1 px-2 text-center">
            <h1 id="main-title" class="text-sm font-bold text-slate-800 editable" contenteditable="true">踐踏日本國土之旅</h1>
            <p id="top-subtitle" class="text-[9px] text-wa-gold font-black uppercase tracking-widest">Day 1 · 5/21</p>
        </div>
        <button id="save-all-btn" onclick="saveAllToCloud()" class="interactive-btn w-10 h-10 flex items-center justify-center text-rose-500 rounded-xl bg-rose-50 border border-rose-100">
            <i class="fa-solid fa-cloud-arrow-up"></i>
        </button>
    </header>

    <main class="max-w-md mx-auto p-4 relative z-10">
        <img src="https://cdn-icons-png.flaticon.com/512/628/628555.png" class="illustration-fuji" alt="Fuji">

        <div id="page-itinerary" class="page-content active">
            <div id="itinerary-wrapper" class="relative min-h-[500px]">
                <div class="timeline-line"></div>
                <div id="current-day-cards" class="space-y-2" ondragover="handleDragOver(event)"></div>
            </div>
        </div>

        <div id="page-toolbox" class="page-content hidden space-y-6">
            <div class="flex gap-4 overflow-x-auto no-scrollbar border-b border-rose-100 pb-2 mb-4 sticky top-16 bg-white/80 backdrop-blur z-20">
                <button onclick="filterToolbox('all')" class="toolbox-nav-btn active py-2 px-3 text-xs font-bold" data-filter="all">全部</button>
                <button onclick="filterToolbox('flight')" class="toolbox-nav-btn py-2 px-3 text-xs font-bold text-slate-400" data-filter="flight">航班</button>
                <button onclick="filterToolbox('hotel')" class="toolbox-nav-btn py-2 px-3 text-xs font-bold text-slate-400" data-filter="hotel">住宿</button>
                <button onclick="filterToolbox('activity')" class="toolbox-nav-btn py-2 px-3 text-xs font-bold text-slate-400" data-filter="activity">活動</button>
                <button onclick="filterToolbox('budget')" class="toolbox-nav-btn py-2 px-3 text-xs font-bold text-slate-400" data-filter="budget">預算</button>
            </div>
            
            <div id="toolbox-list" class="space-y-6" ondragover="handleDragOver(event)"></div>
            
            <div class="grid grid-cols-2 gap-3 mt-8">
                <button onclick="addToolboxCard('flight')" class="interactive-btn p-4 bg-blue-50 text-blue-600 rounded-2xl text-[10px] font-black border border-blue-100">+ 航班資訊</button>
                <button onclick="addToolboxCard('hotel')" class="interactive-btn p-4 bg-indigo-50 text-indigo-600 rounded-2xl text-[10px] font-black border border-indigo-100">+ 主要住宿</button>
                <button onclick="addToolboxCard('activity')" class="interactive-btn p-4 bg-emerald-50 text-emerald-600 rounded-2xl text-[10px] font-black border border-emerald-100">+ 其他活動</button>
                <button onclick="addToolboxCard('budget')" class="interactive-btn p-4 bg-amber-50 text-amber-600 rounded-2xl text-[10px] font-black border border-amber-100">+ 預算追蹤</button>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-6 left-1/2 -translate-x-1/2 w-[90%] max-w-sm bg-white/90 backdrop-blur-xl rounded-3xl p-2 flex justify-around items-center shadow-2xl z-[150] border border-rose-50">
        <button onclick="switchPage('itinerary')" id="nav-itinerary" class="interactive-btn flex items-center gap-3 px-8 py-3 rounded-2xl transition-all nav-btn-active-itinerary">
            <i class="fa-solid fa-map-location-dot"></i>
            <span class="text-xs font-bold">行程規劃</span>
        </button>
        <button onclick="switchPage('toolbox')" id="nav-toolbox" class="interactive-btn flex items-center gap-3 px-8 py-3 rounded-2xl transition-all text-slate-400">
            <i class="fa-solid fa-toolbox"></i>
            <span class="text-xs font-bold">工具箱</span>
        </button>
    </nav>

    <div id="cat-editor-modal" class="fixed inset-0 z-[400] hidden flex items-center justify-center bg-black/40 backdrop-blur-sm">
        <div class="bg-white p-6 rounded-[2.5rem] shadow-2xl w-[320px] space-y-6">
            <h3 class="text-center text-xs font-bold text-slate-400 uppercase tracking-widest">標籤與色彩</h3>
            <input type="text" id="cat-name-input" class="w-full text-center text-lg font-bold border-b-2 border-rose-100 pb-2 focus:border-wa-blue outline-none" placeholder="輸入標籤">
            <div class="grid grid-cols-4 gap-3 justify-items-center" id="color-palette">
                <div class="w-8 h-8 rounded-full bg-[#ff4d6d] cursor-pointer color-option" data-color="#ff4d6d"></div>
                <div class="w-8 h-8 rounded-full bg-[#2a4073] cursor-pointer color-option" data-color="#2a4073"></div>
                <div class="w-8 h-8 rounded-full bg-[#10b981] cursor-pointer color-option" data-color="#10b981"></div>
                <div class="w-8 h-8 rounded-full bg-[#f59e0b] cursor-pointer color-option" data-color="#f59e0b"></div>
                <div class="w-8 h-8 rounded-full bg-[#8b5cf6] cursor-pointer color-option" data-color="#8b5cf6"></div>
                <div class="w-8 h-8 rounded-full bg-[#ec4899] cursor-pointer color-option" data-color="#ec4899"></div>
                <div class="w-8 h-8 rounded-full bg-[#06b6d4] cursor-pointer color-option" data-color="#06b6d4"></div>
                <div class="w-8 h-8 rounded-full bg-[#fb923c] cursor-pointer color-option" data-color="#fb923c"></div>
            </div>
            <div class="flex gap-2">
                <button onclick="closeCatEditor()" class="flex-1 py-3 text-xs font-bold text-slate-400">取消</button>
                <button onclick="applyCatChanges()" class="flex-1 py-3 text-xs font-bold text-white bg-wa-blue rounded-2xl">應用修改</button>
            </div>
        </div>
    </div>

    <input type="file" id="global-uploader" class="hidden" onchange="handleFileSelect(event)">

    <div id="save-toast" class="fixed top-8 left-1/2 -translate-x-1/2 bg-slate-900 text-white px-6 py-3 rounded-full shadow-2xl z-[200] opacity-0 transition-all duration-500 pointer-events-none">
        <span class="text-xs font-bold">同步中...</span>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = {  
            apiKey: "AIzaSyDhNTj3_fzYAToqq3X3IrC-FRkDBlWZShM",  
            authDomain: "tokyo-f003f.firebaseapp.com",  
            projectId: "tokyo-f003f",  
            storageBucket: "tokyo-f003f.firebasestorage.app",  
            messagingSenderId: "962379464999",  
            appId: "1:962379464999:web:8776eae28424d0b0e1a902"  
        };

        const fbApp = initializeApp(firebaseConfig);
        const fbAuth = getAuth(fbApp);
        const fbDb = getFirestore(fbApp);
        const APP_ID = 'tokyo-trip-2026';

        window.fbCore = { db: fbDb, auth: fbAuth, appId: APP_ID, doc, getDoc, setDoc };

        signInAnonymously(fbAuth).then(() => {
            onAuthStateChanged(fbAuth, (user) => { if (user) loadFromCloud(); });
        });
    </script>

    <script>
        const TRIP_DATES = ["5/21", "5/22", "5/23", "5/24", "5/25", "5/26", "5/27", "5/28", "5/29"];
        let appState = {
            mainTitle: "踐踏日本國土之旅",
            currentDay: 1,
            itineraries: {
                1: [
                    { time: "00:55", timeEnd: "06:15", title: "GK28 HKG -> NRT", note: "成田抵達，預計06:15落地", cat: "航班", type: "flight", color: "#2a4073" },
                    { time: "07:30", timeEnd: "08:30", title: "Skyliner 前往上野", note: "辦理入境後搭乘，約41分鐘車程", cat: "交通", type: "activity", color: "#64748b" },
                    { time: "09:00", timeEnd: "13:00", title: "上野阿美橫丁/購物", note: "行李寄放置物櫃。重點：阿美橫、丸井OIOI、atre", cat: "購物", type: "activity", color: "#10b981" },
                    { time: "13:00", timeEnd: "16:00", title: "淺草行程/五反田休息", note: "銀座線5分到淺草，或提早回五反田取行李", cat: "行程", type: "activity", color: "#8b5cf6" },
                    { time: "16:00", title: "SLEEP INN GOTANDA", note: "Check-in 休息。JR山手線五反田站30分", cat: "住宿", type: "hotel", color: "#2a4073" },
                    { time: "18:00", title: "新宿/銀座購物", note: "新宿百貨密集；銀座旗艦店(Uniqlo)直達", cat: "購物", type: "activity", color: "#10b981" }
                ],
                2: [
                    { time: "10:00", timeEnd: "13:00", title: "東京鐵塔拍照", note: "芝公園/增上寺/赤羽橋路口拍攝點", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "13:00", timeEnd: "15:00", title: "前往澀谷 & 午餐", note: "地鐵前往澀谷，快速用餐", cat: "行程", type: "activity", color: "#8b5cf6" },
                    { time: "15:20", timeEnd: "16:20", title: "SHIBUYA STREET RIDE", note: "提前15分到澀谷Fukuras 1F", cat: "體驗", type: "activity", color: "#f59e0b" },
                    { time: "17:30", title: "SHIBUYA SKY 拍照", note: "日落時間約18:00多", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "19:00", title: "澀谷區購物", note: "PARCO、Hikarie、109，營業至21:00", cat: "購物", type: "activity", color: "#10b981" }
                ],
                3: [
                    { time: "08:30", timeEnd: "13:30", title: "前往鎌倉", note: "經大崎搭JR湘南新宿線。重點：鶴岡八幡宮、小町通", cat: "交通", type: "activity", color: "#64748b" },
                    { time: "13:30", title: "鎌倉高校前平交道", note: "江之電拍照行程", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "14:00", title: "江之島巡禮", note: "江之島大橋、神社、展望塔", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "19:00", title: "返回市區晚餐", note: "JR或小田急電鐵返回", cat: "行程", type: "activity", color: "#8b5cf6" }
                ],
                4: [
                    { time: "08:00", title: "東京租車 → 伊豆", note: "自駕行程開始 (1.5-2h)", cat: "自駕", type: "activity", color: "#fb923c" },
                    { time: "10:00", title: "大室山", note: "預計停留1.5小時。可選：城崎海岸/三島步道", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "13:00", title: "伊豆 → 熱海", note: "約1小時車程", cat: "自駕", type: "activity", color: "#fb923c" },
                    { time: "15:00", title: "熱海租和服 & 佔位", note: "Sun Beach 沙灘區準備看花火", cat: "體驗", type: "activity", color: "#f59e0b" },
                    { time: "20:20", timeEnd: "20:40", title: "熱海海上花火大會", note: "熱海灣沙灘觀賞，空中及海面倒影", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "21:30", title: "入住：登坂酒店", note: "河口湖/熱海區域住宿", cat: "住宿", type: "hotel", color: "#2a4073" }
                ],
                5: [
                    { time: "09:00", title: "前往富士急樂園", note: "自駕前往", cat: "自駕", type: "activity", color: "#fb923c" },
                    { time: "09:30", title: "富士急樂園", note: "Fujiyama、高飛車、戰栗迷宮", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "14:00", title: "前往河口湖", note: "約15分車程", cat: "自駕", type: "activity", color: "#fb923c" },
                    { time: "14:30", title: "河口湖觀光", note: "富士山景觀、靜岡夢之大橋", cat: "景點", type: "activity", color: "#ff4d6d" }
                ],
                6: [
                    { time: "08:00", title: "河口湖 → 高山", note: "長途自駕約4小時", cat: "自駕", type: "activity", color: "#fb923c" },
                    { time: "12:00", title: "高山午餐", note: "當地美食探索", cat: "飲食", type: "activity", color: "#ec4899" },
                    { time: "13:30", title: "高山 → 白川鄉", note: "約1小時車程", cat: "自駕", type: "activity", color: "#fb923c" },
                    { time: "14:30", title: "白川鄉合掌村", note: "展望台、參觀民宿。5月綠意美景", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "17:00", title: "返回高山", note: "18:00 晚餐", cat: "行程", type: "activity", color: "#8b5cf6" }
                ],
                7: [
                    { time: "09:00", title: "前往立山站", note: "安排「大町TRAFFIC」代移車服務", cat: "自駕", type: "activity", color: "#fb923c" },
                    { time: "11:00", title: "立山黑部阿爾卑斯路線", note: "纜車/巴士/室堂雪之大谷/黑部水壩", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "18:00", title: "驅車前往名古屋", note: "約3小時車程", cat: "自駕", type: "activity", color: "#fb923c" }
                ],
                8: [
                    { time: "10:00", title: "名古屋市區觀光", note: "自由安排", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "12:00", title: "吉卜力樂園", note: "龍貓森林、魔女之谷。停愛地球博公園", cat: "景點", type: "activity", color: "#ff4d6d" },
                    { time: "19:00", title: "大須商店街", note: "晚餐及最後購物", cat: "購物", type: "activity", color: "#10b981" }
                ],
                9: [
                    { time: "10:00", title: "Check-out 離開名古屋", note: "準備返回東京方向", cat: "行程", type: "activity", color: "#8b5cf6" },
                    { time: "14:30", title: "Outlet 購物", note: "最後衝刺", cat: "購物", type: "activity", color: "#10b981" },
                    { time: "16:00", title: "還車手續", note: "抵達還車處", cat: "交通", type: "activity", color: "#64748b" },
                    { time: "17:30", title: "到達成田機場", note: "辦理登機", cat: "行程", type: "activity", color: "#8b5cf6" },
                    { time: "20:10", timeEnd: "23:55", title: "GK29 NRT -> HKG", note: "搭機返港", cat: "航班", type: "flight", color: "#2a4073" }
                ]
            },
            toolbox: []
        };

        async function loadFromCloud() {
            const { db, appId, doc, getDoc } = window.fbCore;
            const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'itinerary', 'current');
            try {
                const snap = await getDoc(docRef);
                if (snap.exists()) appState = snap.data();
            } catch (e) { console.log("使用本地預設數據"); }
            
            document.getElementById('loading-overlay').classList.add('hidden');
            initUI();
        }

        async function saveAllToCloud() {
            const { db, appId, auth, doc, setDoc } = window.fbCore;
            if (!auth.currentUser) return;
            showToast("正在保存至雲端...");
            
            appState.mainTitle = document.getElementById('main-title').innerText;
            syncCurrentPageData();

            try {
                await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'itinerary', 'current'), appState);
                showToast("同步成功 ✨");
            } catch (e) { showToast("同步失敗 ❌"); }
        }

        function initUI() {
            document.getElementById('main-title').innerText = appState.mainTitle;
            renderSidebar();
            renderDayCards(appState.currentDay || 1);
            renderToolbox();
            
            document.querySelectorAll('.color-option').forEach(opt => {
                opt.onclick = () => {
                    document.querySelectorAll('.color-option').forEach(o => o.style.transform = 'scale(1)');
                    opt.style.transform = 'scale(1.3)';
                    window.selectedHex = opt.dataset.color;
                };
            });
        }

        function createCard(data) {
            const card = document.createElement('div');
            card.className = 'flex gap-6 relative draggable-item group py-2';
            card.draggable = true;
            card.dataset.type = data.type || 'activity';

            const isBudget = data.type === 'budget';
            // 增強地點偵測：優先取標題空格前的關鍵詞
            const locationMatch = data.title.match(/^[^0-9\s(->:)]+/);
            const location = locationMatch ? locationMatch[0] : data.title.split(' ')[0];

            card.innerHTML = `
                <div class="w-14 shrink-0 flex flex-col items-center pt-2">
                    <div class="time-main text-[11px] font-black text-slate-800 editable" contenteditable="true">${data.time || '--:--'}</div>
                    <div class="time-sub text-[9px] font-bold text-rose-400 mt-1 editable" contenteditable="true">${data.timeEnd || ''}</div>
                    <div class="w-3 h-3 rounded-full bg-rose-400 border-2 border-white shadow-sm mt-3 relative z-10"></div>
                </div>

                <div class="flex-1 bg-white rounded-[2rem] p-5 shadow-sm border border-rose-50 relative hover:shadow-md transition-shadow">
                    <div class="absolute top-4 right-4 opacity-0 group-hover:opacity-100 flex gap-2 transition-opacity">
                        <i class="fa-solid fa-grip-vertical text-slate-200 cursor-grab"></i>
                        <button onclick="this.closest('.draggable-item').remove()" class="text-rose-200 hover:text-rose-500"><i class="fa-solid fa-circle-xmark"></i></button>
                    </div>

                    <div class="mb-2">
                        <span onclick="openCatEditor(this)" class="cat-tag text-[8px] font-black text-white px-3 py-1 rounded-full uppercase" style="background-color: ${data.color || '#64748b'}">${data.cat || '一般'}</span>
                    </div>

                    <h3 class="card-title text-base font-bold text-slate-800 editable leading-snug" contenteditable="true">${data.title}</h3>
                    <p class="card-note text-[11px] text-slate-400 mt-2 editable whitespace-pre-wrap" contenteditable="true">${data.note || ''}</p>

                    ${isBudget ? `
                        <div class="mt-4 bg-rose-50/50 rounded-2xl p-4">
                            <div class="budget-rows space-y-2"></div>
                            <button onclick="addBudgetRow(this)" class="w-full py-2 text-[10px] font-bold text-rose-300 border border-dashed border-rose-200 rounded-xl mt-2">+ 新增明細</button>
                            <div class="pt-3 border-t border-rose-100 mt-3 flex justify-between font-black text-slate-800 text-xs">
                                <span>TOTAL</span><span class="total-val">¥ 0</span>
                            </div>
                        </div>
                    ` : ''}

                    <div class="file-list flex flex-wrap gap-2 mt-3">
                        ${(data.files || []).map(f => `<div class="bg-slate-100 text-slate-500 text-[9px] px-2 py-1 rounded-lg flex items-center gap-1">
                            <i class="fa-solid fa-paperclip"></i><span>${f}</span>
                            <i class="fa-solid fa-xmark cursor-pointer" onclick="this.parentElement.remove()"></i>
                        </div>`).join('')}
                    </div>

                    <div class="grid grid-cols-2 gap-2 mt-4">
                        <button onclick="triggerUpload(this)" class="interactive-btn py-2 bg-slate-50 text-slate-500 rounded-xl text-[10px] font-bold"><i class="fa-solid fa-upload mr-1"></i>文件</button>
                        ${!isBudget ? `<button onclick="window.open('https://www.google.com/maps/search/${encodeURIComponent(location)}')" class="interactive-btn py-2 bg-rose-50 text-rose-500 rounded-xl text-[10px] font-bold"><i class="fa-solid fa-location-arrow mr-1"></i>導航</button>` : ''}
                    </div>
                </div>
            `;

            if(isBudget && data.budgetItems) {
                const list = card.querySelector('.budget-rows');
                data.budgetItems.forEach(bi => list.appendChild(createBudgetRowEl(bi.name, bi.amount)));
                updateTotal(card);
            }

            card.addEventListener('dragstart', () => { window.draggedItem = card; card.classList.add('dragging'); });
            card.addEventListener('dragend', () => card.classList.remove('dragging'));
            return card;
        }

        function renderDayCards(day) {
            const container = document.getElementById('current-day-cards');
            container.innerHTML = '';
            appState.currentDay = day;
            document.getElementById('top-subtitle').innerText = `Day ${day} · ${TRIP_DATES[day-1]}`;
            
            const cards = appState.itineraries[day] || [];
            cards.forEach(c => container.appendChild(createCard(c)));

            const addBtn = document.createElement('button');
            addBtn.className = 'w-full py-6 border-2 border-dashed border-rose-100 rounded-[2rem] text-rose-300 font-bold text-xs mt-4 interactive-btn';
            addBtn.innerHTML = `<i class="fa-solid fa-plus mr-2"></i>新增行程項目`;
            addBtn.onclick = () => container.insertBefore(createCard({ title: "新目的地", time: "12:00", cat: "行程", color: "#8b5cf6" }), addBtn);
            container.appendChild(addBtn);
        }

        function renderSidebar() {
            const list = document.getElementById('side-nav-list');
            list.innerHTML = '';
            TRIP_DATES.forEach((date, i) => {
                const day = i + 1;
                const btn = document.createElement('button');
                btn.className = `menu-item interactive-btn flex items-center gap-4 w-full p-4 rounded-2xl text-sm font-bold ${day === appState.currentDay ? 'active' : 'text-slate-500'}`;
                btn.onclick = () => {
                    syncCurrentPageData();
                    document.querySelectorAll('.menu-item').forEach(b => b.classList.remove('active'));
                    btn.classList.add('active');
                    renderDayCards(day);
                    toggleSidebar();
                };
                btn.innerHTML = `<div class="w-10 h-10 rounded-full flex items-center justify-center text-[10px] ${day === appState.currentDay ? 'bg-white/20' : 'bg-rose-50 text-rose-300'}">D${day}</div><span>${date} 行程</span>`;
                list.appendChild(btn);
            });
        }

        function renderToolbox() {
            const container = document.getElementById('toolbox-list');
            container.innerHTML = '';
            (appState.toolbox || []).forEach(d => container.appendChild(createCard(d)));
        }

        function addToolboxCard(type) {
            const config = {
                flight: { cat: '航班', color: '#2a4073' },
                hotel: { cat: '住宿', color: '#fb923c' },
                activity: { cat: '活動', color: '#10b981' },
                budget: { cat: '預算', color: '#ec4899' }
            }[type];
            const card = createCard({ title: `新${config.cat}資訊`, type, ...config });
            document.getElementById('toolbox-list').appendChild(card);
            filterToolbox('all');
        }

        function filterToolbox(type) {
            document.querySelectorAll('.toolbox-nav-btn').forEach(b => b.classList.remove('active', 'text-wa-blue'));
            event.target.classList.add('active', 'text-wa-blue');
            document.querySelectorAll('#toolbox-list .draggable-item').forEach(c => {
                c.style.display = (type === 'all' || c.dataset.type === type) ? 'flex' : 'none';
            });
        }

        function createBudgetRowEl(name="項目", amount=0) {
            const div = document.createElement('div');
            div.className = 'budget-row flex justify-between text-[11px] group/row';
            div.innerHTML = `
                <span class="b-name editable flex-1" contenteditable="true">${name}</span>
                <div class="flex items-center gap-2">
                    <span class="text-rose-300">¥</span>
                    <span class="b-val editable font-black" contenteditable="true" oninput="updateTotal(this.closest('.draggable-item'))">${amount}</span>
                    <i class="fa-solid fa-circle-minus text-rose-100 hover:text-rose-400 cursor-pointer text-[9px]" onclick="const card=this.closest('.draggable-item'); this.parentElement.parentElement.remove(); updateTotal(card)"></i>
                </div>`;
            return div;
        }
        function addBudgetRow(btn) { btn.previousElementSibling.appendChild(createBudgetRowEl()); }
        function updateTotal(card) {
            let sum = 0;
            card.querySelectorAll('.b-val').forEach(v => sum += parseFloat(v.innerText.replace(/,/g,'')) || 0);
            card.querySelector('.total-val').innerText = '¥ ' + sum.toLocaleString();
        }

        function syncCurrentPageData() {
            const cards = document.querySelectorAll('#current-day-cards .draggable-item');
            if(cards.length > 0) appState.itineraries[appState.currentDay] = Array.from(cards).map(c => getCardData(c));
            const toolbox = document.querySelectorAll('#toolbox-list .draggable-item');
            appState.toolbox = Array.from(toolbox).map(c => getCardData(c));
        }

        function getCardData(card) {
            const tag = card.querySelector('.cat-tag');
            return {
                title: card.querySelector('.card-title').innerText,
                note: card.querySelector('.card-note').innerText,
                time: card.querySelector('.time-main')?.innerText,
                timeEnd: card.querySelector('.time-sub')?.innerText,
                cat: tag.innerText,
                color: tag.style.backgroundColor,
                type: card.dataset.type,
                files: Array.from(card.querySelectorAll('.file-name')).map(f => f.innerText),
                budgetItems: card.dataset.type === 'budget' ? Array.from(card.querySelectorAll('.budget-row')).map(r => ({
                    name: r.querySelector('.b-name').innerText,
                    amount: r.querySelector('.b-val').innerText
                })) : []
            };
        }

        function handleDragOver(e) {
            e.preventDefault();
            const container = e.currentTarget;
            const after = [...container.querySelectorAll('.draggable-item:not(.dragging)')].find(c => e.clientY < c.getBoundingClientRect().top + c.getBoundingClientRect().height/2);
            if (window.draggedItem) container.insertBefore(window.draggedItem, after);
        }

        function triggerUpload(btn) { 
            window.targetFileList = btn.closest('.draggable-item').querySelector('.file-list');
            document.getElementById('global-uploader').click(); 
        }
        function handleFileSelect(e) {
            if(!e.target.files[0]) return;
            const div = document.createElement('div');
            div.className = "bg-slate-100 text-slate-500 text-[9px] px-2 py-1 rounded-lg flex items-center gap-1";
            div.innerHTML = `<i class="fa-solid fa-paperclip"></i><span class="file-name">${e.target.files[0].name}</span><i class="fa-solid fa-xmark cursor-pointer" onclick="this.parentElement.remove()"></i>`;
            window.targetFileList.appendChild(div);
        }

        function switchPage(page) {
            document.querySelectorAll('.page-content').forEach(p => p.classList.add('hidden'));
            document.getElementById('page-' + page).classList.remove('hidden');
            
            const btnItinerary = document.getElementById('nav-itinerary');
            const btnToolbox = document.getElementById('nav-toolbox');
            
            btnItinerary.className = 'interactive-btn flex items-center gap-3 px-8 py-3 rounded-2xl transition-all text-slate-400';
            btnToolbox.className = 'interactive-btn flex items-center gap-3 px-8 py-3 rounded-2xl transition-all text-slate-400';
            
            if (page === 'itinerary') {
                btnItinerary.classList.add('nav-btn-active-itinerary');
                btnItinerary.classList.remove('text-slate-400');
            } else {
                btnToolbox.classList.add('nav-btn-active-toolbox');
                btnToolbox.classList.remove('text-slate-400');
            }
            syncCurrentPageData();
        }

        function openCatEditor(el) {
            window.editingCat = el;
            document.getElementById('cat-name-input').value = el.innerText;
            document.getElementById('cat-editor-modal').classList.remove('hidden');
        }
        function closeCatEditor() { document.getElementById('cat-editor-modal').classList.add('hidden'); }
        function applyCatChanges() {
            window.editingCat.innerText = document.getElementById('cat-name-input').value;
            if(window.selectedHex) window.editingCat.style.backgroundColor = window.selectedHex;
            closeCatEditor();
        }

        function toggleSidebar() {
            document.getElementById('sidebar').classList.toggle('closed');
            document.getElementById('sidebar-overlay').classList.toggle('hidden');
        }

        function showToast(msg) {
            const t = document.getElementById('save-toast');
            t.querySelector('span').innerText = msg;
            t.style.opacity = '1';
            setTimeout(() => t.style.opacity = '0', 2500);
        }
    </script>
</body>
</html>
