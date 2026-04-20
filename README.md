<!DOCTYPE html>
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
        
        // 修正路徑段數：使用符合規範的偶數段路徑
        // 結構：artifacts (col) -> {appId} (doc) -> public (col) -> data (doc) -> itinerary (col) -> current (doc)
        const appId = "tokyo-trip-2026-shared"; 

        window.travelDB = { db, auth, appId, doc, getDoc, setDoc, onSnapshot };
        
        // 核心修正：先確保匿名登入成功
        signInAnonymously(auth).then(() => {
            console.log("Firebase 匿名登入成功");
        }).catch((error) => {
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

        .menu-item { transition: all 0.3s ease; position: relative; overflow: hidden; }
        .menu-item:hover { transform: translateX(8px) scale(1.02); background: rgba(42, 64, 115, 0.05); }
        .menu-item.active { background: var(--wa-blue); color: white; box-shadow: 0 10px 15px -3px rgba(42, 64, 115, 0.3); }

        .nav-tab { transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .nav-tab:hover { transform: translateY(-4px); }
        .nav-tab.active { box-shadow: 0 8px 20px rgba(42, 64, 115, 0.4); }

        .editable { outline: none; border-bottom: 1px dashed transparent; transition: background 0.2s; }
        .editable:hover { background-color: rgba(197, 160, 89, 0.08); border-radius: 4px; }
        
        #loading-overlay { position: fixed; inset: 0; background: white; z-index: 1000; display: flex; flex-direction: column; align-items: center; justify-content: center; transition: opacity 0.5s ease; }
        
        .dragging { opacity: 0.5; transform: scale(0.98); border: 2px dashed var(--wa-gold) !important; }
        
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

        <div id="page-toolbox" class="page-content space-y-6">
            <div class="flex items-center gap-4 overflow-x-auto no-scrollbar border-b border-slate-100 pb-2">
                <button onclick="filterToolbox('all')" class="toolbox-nav-btn active flex-shrink-0 px-2 py-2 text-sm font-bold" data-type="all">全部</button>
                <button onclick="filterToolbox('flight')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="flight">航班</button>
                <button onclick="filterToolbox('hotel')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="hotel">住宿</button>
                <button onclick="filterToolbox('activity')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="activity">活動</button>
                <button onclick="filterToolbox('budget')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="budget">預算</button>
            </div>
            <div id="toolbox-list" class="space-y-4"></div>
            <div class="grid grid-cols-2 gap-3 pt-4">
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
                <div class="w-8 h-8 rounded-full bg-wa-red cursor-pointer hover:scale-125 transition-all" data-color="#d93535" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-wa-blue cursor-pointer hover:scale-125 transition-all" data-color="#2a4073" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-emerald-500 cursor-pointer hover:scale-125 transition-all" data-color="#10b981" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-amber-500 cursor-pointer hover:scale-125 transition-all" data-color="#f59e0b" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-indigo-500 cursor-pointer hover:scale-125 transition-all" data-color="#6366f1" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-rose-400 cursor-pointer hover:scale-125 transition-all" data-color="#fb7185" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-slate-400 cursor-pointer hover:scale-125 transition-all" data-color="#94a3b8" onclick="selectCatColor(this)"></div>
                <div class="w-8 h-8 rounded-full bg-black cursor-pointer hover:scale-125 transition-all" data-color="#000000" onclick="selectCatColor(this)"></div>
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
        let appState = {
            mainTitle: "踐踏日本國土之旅",
            itineraries: {
                1: [{ cat: "交通", time: "08:30", title: "GK28 香港 -> 成田", note: "預計中午落地", color: "#2a4073", files: [] }]
            },
            toolbox: []
        };

        let currentDay = 1;
        let isSaving = false;
        let editingCatElement = null;
        let selectedHex = "#94a3b8";

        // 核心修正：獲取路徑引用
        function getDocRef() {
            const { db, appId, doc } = window.travelDB;
            // 路徑段數：artifacts(1)/{appId}(2)/public(3)/data(4)/itinerary(5)/current(6) -> 偶數，正確
            return doc(db, 'artifacts', appId, 'public', 'data', 'itinerary', 'current');
        }

        async function initCloud() {
            const checkFirebase = setInterval(async () => {
                if (window.travelDB && window.travelDB.auth.currentUser) {
                    clearInterval(checkFirebase);
                    const { getDoc } = window.travelDB;
                    
                    try {
                        const docSnap = await getDoc(getDocRef());
                        if (docSnap.exists()) {
                            appState = docSnap.data();
                        }
                    } catch (e) {
                        console.error("讀取失敗:", e);
                    } finally {
                        hideLoading();
                        initializeAppUI();
                    }
                }
            }, 500);
        }

        function hideLoading() {
            const overlay = document.getElementById('loading-overlay');
            if (overlay) {
                overlay.style.opacity = '0';
                setTimeout(() => overlay.remove(), 500);
            }
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
                const cards = document.querySelectorAll(`#day-view-${day} .draggable-item`);
                appState.itineraries[day] = Array.from(cards).map(card => getCardData(card));
            });
            const toolboxCards = document.querySelectorAll(`#toolbox-list .draggable-item`);
            appState.toolbox = Array.from(toolboxCards).map(card => getCardData(card));

            try {
                const { setDoc } = window.travelDB;
                await setDoc(getDocRef(), appState);
                showToast("雲端同步成功");
            } catch (e) {
                console.error("儲存失敗:", e);
                showToast("同步失敗，請檢查網路");
            } finally {
                isSaving = false;
                saveBtn.innerHTML = '<i class="fa-solid fa-cloud-arrow-up"></i>';
            }
        }

        function getCardData(card) {
            const catEl = card.querySelector('.cat-tag');
            const data = {
                cat: catEl.innerText,
                time: card.querySelector('.time-text')?.innerText || '--:--',
                title: card.querySelector('h3').innerText,
                note: card.querySelector('p').innerText,
                color: catEl.style.backgroundColor,
                type: card.dataset.type || 'none',
                files: Array.from(card.querySelectorAll('.file-tag span')).map(s => s.innerText)
            };
            if (data.type === 'budget') {
                data.budgetItems = Array.from(card.querySelectorAll('.budget-item')).map(row => ({
                    name: row.querySelector('.b-name').innerText,
                    amount: parseFloat(row.querySelector('.b-amount').innerText) || 0
                }));
            }
            return data;
        }

        function createCard(data) {
            const div = document.createElement('div');
            div.className = `bg-white rounded-[2.2rem] shadow-sm border border-slate-50 p-6 space-y-4 relative draggable-item`;
            div.draggable = true;
            div.dataset.type = data.type || 'none';
            
            const isBudget = data.type === 'budget';
            const showNav = (data.type === 'hotel' || data.type === 'activity');

            div.innerHTML = `
                <div class="absolute top-6 right-6 text-slate-100 cursor-grab active:cursor-grabbing"><i class="fa-solid fa-grip-vertical"></i></div>
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
                    <button onclick="addBudgetItem(this)" class="w-full py-2 text-[10px] font-bold text-slate-400 border border-dashed border-slate-200 rounded-lg">+ 新增支出細項</button>
                    <div class="pt-2 border-t border-slate-200 flex justify-between font-bold text-slate-800 text-xs">
                        <span>總計</span><span class="total-amount">0</span>
                    </div>
                </div>` : ''}
                <div class="flex gap-2 pt-1">
                    <button onclick="triggerUpload(this)" class="flex-1 py-2 bg-slate-50 text-slate-400 rounded-xl text-[10px] font-bold"><i class="fa-solid fa-paperclip mr-1"></i>上傳</button>
                    ${showNav ? `<button onclick="openMap(this)" class="flex-1 py-2 bg-blue-50 text-blue-600 rounded-xl text-[10px] font-bold"><i class="fa-solid fa-location-dot mr-1"></i>導航</button>` : ''}
                    <button onclick="this.closest('.draggable-item').remove()" class="w-8 h-8 bg-rose-50 text-rose-400 rounded-xl text-xs flex items-center justify-center"><i class="fa-solid fa-trash-can"></i></button>
                </div>
                <div class="file-container flex flex-wrap gap-2"></div>
            `;

            if (isBudget) {
                const list = div.querySelector('.budget-list');
                (data.budgetItems || []).forEach(bi => list.appendChild(createBudgetItemRow(bi.name, bi.amount)));
                updateBudgetTotal(div);
            }
            if (data.files) {
                const fc = div.querySelector('.file-container');
                data.files.forEach(name => fc.appendChild(createFileTag(name)));
            }
            addDragEvents(div);
            return div;
        }

        function openCatEditor(el) {
            editingCatElement = el;
            selectedHex = el.style.backgroundColor || "#94a3b8";
            document.getElementById('cat-name-input').value = el.innerText;
            document.getElementById('cat-editor-modal').classList.remove('hidden');
        }

        function selectCatColor(el) {
            selectedHex = el.dataset.color;
            document.querySelectorAll('#cat-editor-modal [data-color]').forEach(circle => circle.style.boxShadow = 'none');
            el.style.boxShadow = '0 0 0 4px rgba(0,0,0,0.1)';
        }

        function closeCatEditor() {
            document.getElementById('cat-editor-modal').classList.add('hidden');
            editingCatElement = null;
        }

        function applyCatChanges() {
            if (editingCatElement) {
                editingCatElement.innerText = document.getElementById('cat-name-input').value;
                editingCatElement.style.backgroundColor = selectedHex;
            }
            closeCatEditor();
        }

        function createBudgetItemRow(name = "新項目", amount = 0) {
            const div = document.createElement('div');
            div.className = "budget-item flex justify-between text-[11px] py-1";
            div.innerHTML = `<span class="b-name editable flex-1" contenteditable="true">${name}</span><div class="flex items-center gap-2"><span class="b-amount editable text-right font-mono font-bold" contenteditable="true" oninput="updateBudgetTotal(this.closest('.draggable-item'))">${amount}</span><i class="fa-solid fa-circle-xmark text-slate-200 cursor-pointer" onclick="const p=this.closest('.draggable-item'); this.parentElement.parentElement.remove(); updateBudgetTotal(p);"></i></div>`;
            return div;
        }

        function addBudgetItem(btn) {
            const card = btn.closest('.draggable-item');
            card.querySelector('.budget-list').appendChild(createBudgetItemRow());
            updateBudgetTotal(card);
        }

        function updateBudgetTotal(card) {
            const amounts = card.querySelectorAll('.b-amount');
            let total = 0;
            amounts.forEach(a => total += parseFloat(a.innerText) || 0);
            card.querySelector('.total-amount').innerText = total.toLocaleString();
        }

        function renderSidebar() {
            const list = document.getElementById('side-nav-list');
            list.innerHTML = '';
            tripDates.forEach((date, i) => {
                const dayNum = i + 1;
                const btn = document.createElement('button');
                btn.id = `side-btn-${dayNum}`;
                btn.className = `menu-item flex items-center gap-4 w-full p-4 rounded-2xl text-sm font-bold ${dayNum === 1 ? 'active' : 'text-slate-600'}`;
                btn.innerHTML = `<span class="w-10 h-10 rounded-full flex items-center justify-center text-xs bg-slate-100 text-slate-400">D${dayNum}</span><span>${date} 行程</span>`;
                btn.onclick = () => { showDay(dayNum); toggleSidebar(); };
                list.appendChild(btn);
            });
        }

        function renderAllItineraries() {
            const wrapper = document.getElementById('itinerary-wrapper');
            wrapper.innerHTML = '';
            tripDates.forEach((_, i) => {
                const dayNum = i + 1;
                const section = document.createElement('div');
                section.id = `day-view-${dayNum}`;
                section.className = `day-view space-y-4 ${dayNum === 1 ? 'block' : 'hidden'}`;
                section.innerHTML = `<div class="itinerary-list space-y-4" ondragover="handleDragOver(event)"></div><button onclick="addCard(${dayNum})" class="w-full py-5 border-2 border-dashed border-slate-100 rounded-[2.5rem] text-slate-300 text-xs font-bold interactive-btn hover:border-wa-gold hover:text-wa-gold transition-all">+ 新增 D${dayNum} 行程</button>`;
                (appState.itineraries[dayNum] || []).forEach(data => section.querySelector('.itinerary-list').appendChild(createCard(data)));
                wrapper.appendChild(section);
            });
        }

        function renderToolbox() {
            const list = document.getElementById('toolbox-list');
            list.innerHTML = '';
            (appState.toolbox || []).forEach(data => list.appendChild(createCard(data)));
        }

        function showDay(dayNum) {
            currentDay = dayNum;
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

        function filterToolbox(type) {
            document.querySelectorAll('#toolbox-list .draggable-item').forEach(card => card.style.display = (type === 'all' || card.dataset.type === type) ? 'block' : 'none');
        }

        function addToolboxItem(type) {
            const list = document.getElementById('toolbox-list');
            list.insertBefore(createCard({ cat: "新增", type, title: "點擊編輯", note: "細節...", color: "#94a3b8" }), list.firstChild);
        }

        function addCard(dayNum) {
            document.querySelector(`#day-view-${dayNum} .itinerary-list`).appendChild(createCard({ cat: "新行程", time: "--:--", title: "行程標題", note: "描述...", color: "#94a3b8" }));
        }

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
            const afterElement = getDragAfterElement(list, e.clientY);
            if (afterElement == null) list.appendChild(dragElement); else list.insertBefore(dragElement, afterElement);
        }
        function getDragAfterElement(container, y) {
            const elements = [...container.querySelectorAll('.draggable-item:not(.dragging)')];
            return elements.reduce((closest, child) => {
                const box = child.getBoundingClientRect();
                const offset = y - box.top - box.height / 2;
                if (offset < 0 && offset > closest.offset) return { offset: offset, element: child };
                else return closest;
            }, { offset: Number.NEGATIVE_INFINITY }).element;
        }

        function triggerUpload(btn) { 
            window.currentFileTarget = btn.closest('.draggable-item').querySelector('.file-container'); 
            document.getElementById('global-uploader').click(); 
        }
        function handleFileSelect(e) { 
            if (e.target.files[0] && window.currentFileTarget) {
                window.currentFileTarget.appendChild(createFileTag(e.target.files[0].name));
            }
        }
        function createFileTag(name) {
            const tag = document.createElement('div');
            tag.className = 'file-tag mt-2 text-[10px] bg-slate-50 p-2 rounded-lg border border-slate-100 flex items-center gap-2';
            tag.innerHTML = `<i class="fa-solid fa-file-invoice"></i> <span>${name}</span> <i class="fa-solid fa-xmark ml-auto opacity-30 cursor-pointer" onclick="this.parentElement.remove()"></i>`;
            return tag;
        }
        function showToast(msg) {
            const t = document.getElementById('save-toast'); t.querySelector('span').innerText = msg;
            t.classList.remove('opacity-0', 'pointer-events-none'); t.classList.add('opacity-100');
            setTimeout(() => { t.classList.add('opacity-0', 'pointer-events-none'); t.classList.remove('opacity-100'); }, 2000);
        }
        function openMap(btn) {
            const card = btn.closest('.draggable-item');
            window.open(`https://www.google.com/maps/search/${encodeURIComponent(card.querySelector('h3').innerText + ' ' + card.querySelector('p').innerText)}`);
        }

        window.onload = initCloud;
    </script>
</body>
</html>
