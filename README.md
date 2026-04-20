<html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>2026 踐踏日本國土之旅</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        window.travelDB = { db, auth, appId, doc, getDoc, setDoc };
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
        .dragging { opacity: 0.5; transform: scale(0.98); border: 2px dashed var(--wa-gold) !important; }
        .interactive-btn { transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .interactive-btn:hover { transform: scale(1.05); }
        .editable { outline: none; border-bottom: 1px dashed transparent; padding: 2px 4px; border-radius: 4px; }
        .editable:hover { background-color: rgba(197, 160, 89, 0.08); }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .file-tag { font-size: 10px; background: #f1f5f9; padding: 6px 10px; border-radius: 10px; display: flex; align-items: center; gap: 6px; border: 1px solid #e2e8f0; }
        .toolbox-nav-btn { transition: all 0.3s ease; border-bottom: 3px solid transparent; }
        .toolbox-nav-btn.active { color: var(--wa-blue); border-bottom-color: var(--wa-blue); }
        .budget-item { border-bottom: 1px solid #f1f5f9; }
        .budget-item:last-child { border-bottom: none; }
        
        /* Loading Overlay */
        #loading-overlay { position: fixed; inset: 0; background: white; z-index: 1000; display: flex; flex-direction: column; align-items: center; justify-content: center; transition: opacity 0.5s ease; }
    </style>
</head>
<body class="pb-32">

    <div id="loading-overlay">
        <div class="w-12 h-12 border-4 border-wa-blue border-t-transparent rounded-full animate-spin"></div>
        <p class="mt-4 text-xs font-bold text-slate-400 tracking-widest uppercase">同步旅程中...</p>
    </div>

    <!-- 側邊選單 -->
    <div id="sidebar-overlay" class="fixed inset-0 bg-black/40 hidden opacity-0 z-[190]" onclick="toggleSidebar()"></div>
    <aside id="sidebar" class="fixed top-0 left-0 h-full w-72 bg-white shadow-2xl flex flex-col closed">
        <div class="p-8 border-b border-slate-50">
            <h2 class="text-2xl font-bold text-wa-blue font-serif">旅程清單</h2>
            <p class="text-[10px] text-slate-400 mt-1 uppercase tracking-widest font-bold">Cloud Sync Active</p>
        </div>
        <div class="flex-1 overflow-y-auto p-4 no-scrollbar">
            <div id="side-nav-list" class="space-y-2"></div>
        </div>
    </aside>

    <!-- 頂部欄 -->
    <header class="bg-white/90 backdrop-blur-md border-b border-slate-100 sticky top-0 z-[100] h-30 flex items-center px-4 shadow-sm">
        <button onclick="toggleSidebar()" class="interactive-btn w-12 h-12 flex items-center justify-center text-slate-600 rounded-full hover:bg-slate-50">
            <i class="fa-solid fa-bars-staggered text-xl"></i>
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
        <!-- 行程頁面 -->
        <div id="page-itinerary" class="page-content active">
            <div id="itinerary-wrapper"></div>
        </div>

        <!-- 工具箱頁面 -->
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

    <!-- 色彩選擇器 -->
    <div id="color-picker" class="fixed inset-0 z-[300] hidden flex items-center justify-center bg-black/40 backdrop-blur-sm">
        <div class="bg-white p-6 rounded-[2.5rem] shadow-2xl space-y-4 w-64 text-center">
            <div class="grid grid-cols-4 gap-4 justify-items-center">
                <div class="w-8 h-8 rounded-full bg-wa-red cursor-pointer" onclick="applyColor('#d93535')"></div>
                <div class="w-8 h-8 rounded-full bg-wa-blue cursor-pointer" onclick="applyColor('#2a4073')"></div>
                <div class="w-8 h-8 rounded-full bg-emerald-500 cursor-pointer" onclick="applyColor('#10b981')"></div>
                <div class="w-8 h-8 rounded-full bg-amber-500 cursor-pointer" onclick="applyColor('#f59e0b')"></div>
            </div>
            <button onclick="closeColorPicker()" class="text-xs text-slate-400 font-bold uppercase pt-2">關閉</button>
        </div>
    </div>

    <input type="file" id="global-uploader" class="hidden" onchange="handleFileSelect(event)">
    <div id="save-toast" class="fixed top-8 left-1/2 -translate-x-1/2 bg-slate-900 text-white px-6 py-3 rounded-full shadow-2xl z-[200] opacity-0 pointer-events-none transition-all duration-500">
        <span class="text-xs font-bold tracking-widest uppercase">已儲存至雲端</span>
    </div>

    <script>
        const tripDates = ["5/21", "5/22", "5/23", "5/24", "5/25", "5/26", "5/27", "5/28", "5/29"];
        let appState = {
            mainTitle: "踐踏日本國土之旅",
            itineraries: {
                1: [{ cat: "交通", time: "--:--", title: "GK28 HKG -> NRT", note: "中午落地", color: "#2a4073", files: [] }],
                2: [{ cat: "購物", time: "全日", title: "淺草全日購物", note: "淺草寺、仲見世街", color: "#10b981", files: [] }],
                3: [{ cat: "景點", time: "早晨", title: "鎌倉古都之旅", note: "大佛、長谷寺", color: "#f59e0b", files: [] }],
                4: [{ cat: "重點", time: "20:20", title: "熱海海上花火大會", note: "20:20–20:40 觀賞", color: "#d93535", files: [] }],
                5: [{ cat: "遊樂", time: "全日", title: "富士急ハイランド", note: "雲霄飛車行程", color: "#6366f1", files: [] }],
                6: [{ cat: "交通", time: "下午", title: "前往名古屋", note: "自駕移動", color: "#2a4073", files: [] }],
                7: [{ cat: "重點", time: "下午", title: "立山黑部雪之大谷", note: "需預約票券", color: "#d93535", files: [] }],
                8: [{ cat: "重點", time: "中午", title: "吉卜力樂園", note: "龍貓森林區域", color: "#d93535", files: [] }],
                9: [{ cat: "交通", time: "--:--", title: "GK29 NRT -> HKG", note: "成田返港", color: "#2a4073", files: [] }]
            },
            toolbox: [
                { id: 't1', type: 'flight', cat: '航班', title: 'GK28 香港 -> 成田', note: '5/21 落地', color: '#2a4073', files: [] },
                { id: 't2', type: 'hotel', cat: '住宿', title: '登坂酒店', note: '富士河口湖町', color: '#6366f1', files: [] },
                { id: 't3', type: 'budget', cat: '預算', title: '旅行支出清單', note: '點擊清單管理支出', color: '#f59e0b', files: [], budgetItems: [{name: '機票', amount: 3000}] }
            ]
        };

        let currentDay = 1;
        let currentFileTarget = null;
        let currentColorTarget = null;
        let isSaving = false;

        async function initCloud() {
            try {
                const { db, appId, getDoc, doc, auth } = window.travelDB;
                
                // Auth first
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                // Fetch data
                const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'itinerary_v1');
                const docSnap = await getDoc(docRef);
                
                if (docSnap.exists()) {
                    appState = docSnap.data();
                } else {
                    // Save initial state if not exists
                    await setDoc(docRef, appState);
                }
            } catch (e) {
                console.error("Cloud Error", e);
            } finally {
                document.getElementById('loading-overlay').style.opacity = '0';
                setTimeout(() => document.getElementById('loading-overlay').remove(), 500);
                initializeAppUI();
            }
        }

        function initializeAppUI() {
            document.getElementById('main-title').innerText = appState.mainTitle;
            renderSidebar();
            renderAllItineraries();
            renderToolbox();
            updateHeader(1);
        }

        async function saveAllToCloud() {
            if (isSaving) return;
            isSaving = true;
            const saveBtn = document.getElementById('save-all-btn');
            saveBtn.innerHTML = '<i class="fa-solid fa-spinner animate-spin"></i>';

            // Collect all current data from DOM
            appState.mainTitle = document.getElementById('main-title').innerText;
            
            // Collect Itineraries
            tripDates.forEach((_, i) => {
                const day = i + 1;
                const cards = document.querySelectorAll(`#day-view-${day} .draggable-item`);
                appState.itineraries[day] = Array.from(cards).map(card => getCardData(card));
            });

            // Collect Toolbox
            const toolboxCards = document.querySelectorAll(`#toolbox-list .draggable-item`);
            appState.toolbox = Array.from(toolboxCards).map(card => getCardData(card));

            try {
                const { db, appId, setDoc, doc } = window.travelDB;
                const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'itinerary_v1');
                await setDoc(docRef, appState);
                showToast("雲端同步成功");
            } catch (e) {
                showToast("儲存失敗，請檢查網路");
            } finally {
                isSaving = false;
                saveBtn.innerHTML = '<i class="fa-solid fa-cloud-arrow-up"></i>';
            }
        }

        function getCardData(card) {
            const data = {
                cat: card.querySelector('span[contenteditable]').innerText,
                time: card.querySelector('.time-text')?.innerText || '--:--',
                title: card.querySelector('h3').innerText,
                note: card.querySelector('p').innerText,
                color: card.querySelector('span[contenteditable]').style.backgroundColor,
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

        function createCard(data, mode) {
            const div = document.createElement('div');
            div.className = `bg-white rounded-[2.2rem] shadow-sm border border-slate-50 p-6 space-y-4 relative draggable-item`;
            div.draggable = true;
            div.dataset.type = data.type || 'none';
            
            const showNav = (data.type === 'hotel' || data.type === 'activity');
            const isBudget = data.type === 'budget';
            
            div.innerHTML = `
                <div class="absolute top-6 right-6 text-slate-200 cursor-grab active:cursor-grabbing"><i class="fa-solid fa-grip-vertical"></i></div>
                <div class="flex justify-between items-center pr-8">
                    <span onclick="openColorPicker(this)" class="text-[9px] font-black text-white px-3 py-1 rounded-full cursor-pointer editable" style="background-color: ${data.color || '#94a3b8'}" contenteditable="true">${data.cat}</span>
                    <span class="time-text text-[10px] text-slate-300 font-black font-mono editable" contenteditable="true">${data.time || '--:--'}</span>
                </div>
                <div>
                    <h3 class="text-lg font-bold text-slate-800 editable" contenteditable="true">${data.title}</h3>
                    <p class="text-xs text-slate-400 mt-2 leading-relaxed editable" contenteditable="true">${data.note}</p>
                </div>

                ${isBudget ? `<div class="budget-section bg-slate-50 rounded-2xl p-4 space-y-2">
                    <div class="budget-list space-y-2"></div>
                    <button onclick="addBudgetItem(this)" class="w-full py-2 text-[10px] font-bold text-slate-400 border border-dashed border-slate-200 rounded-lg">+ 新增支出細項</button>
                    <div class="pt-2 border-t border-slate-200 flex justify-between font-bold text-slate-800 text-xs">
                        <span>總計</span><span class="total-amount">0</span>
                    </div>
                </div>` : ''}

                <div class="flex gap-2 pt-1">
                    <button onclick="triggerUpload(this)" class="flex-1 py-2 bg-slate-50 text-slate-400 rounded-xl text-[10px] interactive-btn font-bold"><i class="fa-solid fa-paperclip mr-1"></i>上傳</button>
                    ${showNav ? `<button onclick="openMap(this)" class="flex-1 py-2 bg-blue-50 text-blue-600 rounded-xl text-[10px] font-bold interactive-btn"><i class="fa-solid fa-location-dot mr-1"></i>導航</button>` : ''}
                    <button onclick="this.closest('.draggable-item').remove()" class="w-8 h-8 bg-rose-50 text-rose-400 rounded-xl text-xs interactive-btn flex items-center justify-center"><i class="fa-solid fa-trash-can"></i></button>
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

        function createBudgetItemRow(name = "新項目", amount = 0) {
            const div = document.createElement('div');
            div.className = "budget-item flex justify-between text-[11px] py-1";
            div.innerHTML = `
                <span class="b-name editable flex-1" contenteditable="true">${name}</span>
                <div class="flex items-center gap-2">
                    <span class="b-amount editable text-right font-mono font-bold" contenteditable="true" oninput="updateBudgetTotal(this.closest('.draggable-item'))">${amount}</span>
                    <i class="fa-solid fa-circle-xmark text-slate-200 cursor-pointer" onclick="const p=this.closest('.draggable-item'); this.parentElement.parentElement.remove(); updateBudgetTotal(p);"></i>
                </div>
            `;
            return div;
        }

        function addBudgetItem(btn) {
            const card = btn.closest('.draggable-item');
            const list = card.querySelector('.budget-list');
            list.appendChild(createBudgetItemRow());
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
                btn.className = `menu-item flex items-center gap-4 w-full p-4 rounded-2xl text-sm font-bold transition-all ${dayNum === 1 ? 'active' : 'text-slate-600'}`;
                btn.innerHTML = `<span class="w-10 h-10 rounded-full flex items-center justify-center text-xs ${dayNum === 1 ? 'bg-white/20' : 'bg-slate-100 text-slate-400'}">D${dayNum}</span><span>${date} 行程</span>`;
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
                section.innerHTML = `<div class="itinerary-list space-y-4" ondragover="handleDragOver(event)"></div><button onclick="addCard(${dayNum})" class="w-full py-5 border-2 border-dashed border-slate-100 rounded-[2.5rem] text-slate-300 text-xs font-bold hover:border-wa-gold hover:text-wa-gold transition-all">+ 新增 D${dayNum} 行程</button>`;
                const list = section.querySelector('.itinerary-list');
                (appState.itineraries[dayNum] || []).forEach(data => list.appendChild(createCard(data, 'itinerary')));
                wrapper.appendChild(section);
            });
        }

        function renderToolbox() {
            const list = document.getElementById('toolbox-list');
            list.innerHTML = '';
            appState.toolbox.forEach(data => list.appendChild(createCard(data, 'toolbox')));
        }

        function addDragEvents(el) {
            el.addEventListener('dragstart', (e) => { dragElement = el; setTimeout(() => el.classList.add('dragging'), 0); });
            el.addEventListener('dragend', () => { el.classList.remove('dragging'); dragElement = null; });
        }

        let dragElement = null;
        function handleDragOver(e) {
            e.preventDefault();
            const list = e.currentTarget;
            const afterElement = getDragAfterElement(list, e.clientY);
            if (afterElement == null) list.appendChild(dragElement);
            else list.insertBefore(dragElement, afterElement);
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

        function createFileTag(name) {
            const tag = document.createElement('div');
            tag.className = 'file-tag mt-2';
            tag.innerHTML = `<i class="fa-solid fa-file-invoice"></i> <span>${name}</span> <i class="fa-solid fa-xmark ml-auto opacity-30 cursor-pointer" onclick="this.parentElement.remove()"></i>`;
            return tag;
        }

        function showDay(dayNum) {
            currentDay = dayNum;
            document.querySelectorAll('#side-nav-list button').forEach(b => {
                b.classList.remove('active');
                b.querySelector('span:first-child').className = "w-10 h-10 rounded-full flex items-center justify-center text-xs bg-slate-100 text-slate-400";
            });
            const activeBtn = document.getElementById(`side-btn-${dayNum}`);
            activeBtn.classList.add('active');
            activeBtn.querySelector('span:first-child').className = "w-10 h-10 rounded-full flex items-center justify-center text-xs bg-white/20 text-white";
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
            if(page === 'toolbox') {
                document.getElementById('top-subtitle').innerText = '附件與支出管理';
            } else {
                updateHeader(currentDay);
            }
        }

        function filterToolbox(type) {
            document.querySelectorAll('.toolbox-nav-btn').forEach(btn => {
                btn.classList.remove('active', 'text-wa-blue');
                btn.classList.add('text-slate-400');
                if(btn.dataset.type === type) btn.classList.add('active', 'text-wa-blue');
            });
            document.querySelectorAll('#toolbox-list .draggable-item').forEach(card => {
                card.style.display = (type === 'all' || card.dataset.type === type) ? 'block' : 'none';
            });
        }

        function addToolboxItem(type) {
            const list = document.getElementById('toolbox-list');
            const item = createCard({ cat: "新增", type, title: "點擊編輯", note: "細節...", color: "#94a3b8", budgetItems: type === 'budget' ? [] : undefined }, 'toolbox');
            list.insertBefore(item, list.firstChild);
            filterToolbox('all');
        }

        function addCard(dayNum) {
            const list = document.querySelector(`#day-view-${dayNum} .itinerary-list`);
            list.appendChild(createCard({ cat: "新項目", time: "--:--", title: "行程標題", note: "描述...", color: "#94a3b8" }, 'itinerary'));
        }

        function triggerUpload(btn) { currentFileTarget = btn.closest('.draggable-item').querySelector('.file-container'); document.getElementById('global-uploader').click(); }
        function handleFileSelect(e) {
            if (e.target.files[0] && currentFileTarget) {
                currentFileTarget.appendChild(createFileTag(e.target.files[0].name));
            }
        }
        function openColorPicker(el) { currentColorTarget = el; document.getElementById('color-picker').classList.remove('hidden'); }
        function closeColorPicker() { document.getElementById('color-picker').classList.add('hidden'); }
        function applyColor(hex) { if(currentColorTarget) currentColorTarget.style.backgroundColor = hex; closeColorPicker(); }
        function toggleSidebar() { 
            const s = document.getElementById('sidebar'); 
            const o = document.getElementById('sidebar-overlay');
            if(s.classList.contains('closed')){ s.classList.remove('closed'); o.classList.remove('hidden'); setTimeout(()=>o.classList.add('opacity-100'),10); }
            else { s.classList.add('closed'); o.classList.remove('opacity-100'); setTimeout(()=>o.classList.add('hidden'),300); }
        }
        function showToast(msg) {
            const t = document.getElementById('save-toast');
            t.querySelector('span').innerText = msg;
            t.classList.remove('opacity-0', 'pointer-events-none');
            t.classList.add('opacity-100');
            setTimeout(() => { t.classList.add('opacity-0', 'pointer-events-none'); t.classList.remove('opacity-100'); }, 2500);
        }
        function openMap(btn) {
            const card = btn.closest('.draggable-item');
            window.open(`https://www.google.com/maps/search/${encodeURIComponent(card.querySelector('h3').innerText + ' ' + card.querySelector('p').innerText)}`);
        }

        // Run Cloud Sync
        window.onload = initCloud;
    </script>
</body>
</html>
