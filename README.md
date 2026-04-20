<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>2026 日本深度遊 | 智慧旅行管家</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@400;700&family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
        
        :root {
            --wa-red: #d93535; --wa-blue: #2a4073; --wa-gold: #c5a059; --wa-bg: #fdfaf5;
        }

        body { font-family: 'Noto Sans TC', sans-serif; background-color: var(--wa-bg); color: #2d2d2d; overflow-x: hidden; }
        h1, h2, .font-serif { font-family: 'Noto Serif TC', serif; }

        /* 側邊選單互動 */
        #sidebar { transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1); z-index: 200; }
        #sidebar.closed { transform: translateX(-100%); }
        .menu-item { transition: all 0.3s ease; }
        .menu-item:hover { transform: scale(1.05) translateX(10px); background-color: rgba(42, 64, 115, 0.05); }
        .menu-item.active { background-color: var(--wa-blue) !important; color: white !important; transform: scale(1.05) translateX(10px); }

        /* 頁面切換動畫 */
        .page-content { display: none; opacity: 0; transform: translateY(15px); transition: all 0.4s ease-out; }
        .page-content.active { display: block; opacity: 1; transform: translateY(0); }
        .dragging { opacity: 0.5; transform: scale(0.98); border: 2px dashed var(--wa-gold) !important; }

        /* 按鈕與卡片 */
        .interactive-btn { transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        .interactive-btn:hover { transform: scale(1.05); }
        .interactive-btn:active { transform: scale(0.95); }

        .editable { outline: none; border-bottom: 1px dashed transparent; padding: 2px 4px; border-radius: 4px; }
        .editable:hover { background-color: rgba(197, 160, 89, 0.08); }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        .file-tag { font-size: 10px; background: #f1f5f9; padding: 6px 10px; border-radius: 10px; display: flex; align-items: center; gap: 6px; border: 1px solid #e2e8f0; }

        /* 工具箱專屬樣式 */
        .toolbox-nav-btn { transition: all 0.3s ease; border-bottom: 3px solid transparent; }
        .toolbox-nav-btn.active { color: var(--wa-blue); border-bottom-color: var(--wa-blue); }
    </style>
</head>
<body class="pb-32">

    <!-- 側邊選單 -->
    <div id="sidebar-overlay" class="fixed inset-0 bg-black/40 hidden opacity-0 z-[190]" onclick="toggleSidebar()"></div>
    <aside id="sidebar" class="fixed top-0 left-0 h-full w-72 bg-white shadow-2xl flex flex-col closed">
        <div class="p-8 border-b border-slate-50">
            <h2 class="text-2xl font-bold text-wa-blue font-serif">旅程清單</h2>
            <p class="text-[10px] text-slate-400 mt-1 uppercase tracking-widest font-bold">2026 Japan Itinerary</p>
        </div>
        <div class="flex-1 overflow-y-auto p-4 no-scrollbar">
            <div id="side-nav-list" class="space-y-2">
                <!-- 由 JS 生成 Day 1 - Day 9 -->
            </div>
        </div>
    </aside>

    <!-- 頂部欄 -->
    <header class="bg-white/90 backdrop-blur-md border-b border-slate-100 sticky top-0 z-[100] h-16 flex items-center px-4 shadow-sm">
        <button onclick="toggleSidebar()" class="interactive-btn w-12 h-12 flex items-center justify-center text-slate-600 rounded-full hover:bg-slate-50">
            <i class="fa-solid fa-bars-staggered text-xl"></i>
        </button>
        <div class="flex-1 px-4 overflow-hidden text-center">
            <h1 id="top-title" class="text-base font-bold text-slate-800 editable truncate" contenteditable="true">2026 日本深度遊</h1>
            <p id="top-subtitle" class="text-[10px] text-wa-gold font-bold uppercase tracking-wider editable" contenteditable="true">Day 1 · 5/21</p>
        </div>
        <button onclick="saveFeedback()" class="interactive-btn w-10 h-10 flex items-center justify-center text-emerald-500 rounded-full bg-emerald-50">
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
            <!-- 工具箱分類列 -->
            <div class="flex items-center gap-4 overflow-x-auto no-scrollbar border-b border-slate-100 pb-2">
                <button onclick="filterToolbox('all')" class="toolbox-nav-btn active flex-shrink-0 px-2 py-2 text-sm font-bold" data-type="all">全部</button>
                <button onclick="filterToolbox('flight')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="flight">航班</button>
                <button onclick="filterToolbox('hotel')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="hotel">住宿</button>
                <button onclick="filterToolbox('activity')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="activity">活動</button>
                <button onclick="filterToolbox('budget')" class="toolbox-nav-btn flex-shrink-0 px-2 py-2 text-sm font-bold text-slate-400" data-type="budget">預算</button>
            </div>

            <div id="toolbox-list" class="space-y-4">
                <!-- 工具卡片由 JS 渲染 -->
            </div>

            <!-- 工具箱新增按鈕 -->
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
            <p class="text-xs font-black text-slate-400 uppercase tracking-widest">標籤顏色</p>
            <div class="grid grid-cols-4 gap-4 justify-items-center">
                <div class="w-8 h-8 rounded-full bg-wa-red cursor-pointer" onclick="applyColor('#d93535')"></div>
                <div class="w-8 h-8 rounded-full bg-wa-blue cursor-pointer" onclick="applyColor('#2a4073')"></div>
                <div class="w-8 h-8 rounded-full bg-emerald-500 cursor-pointer" onclick="applyColor('#10b981')"></div>
                <div class="w-8 h-8 rounded-full bg-amber-500 cursor-pointer" onclick="applyColor('#f59e0b')"></div>
                <div class="w-8 h-8 rounded-full bg-indigo-500 cursor-pointer" onclick="applyColor('#6366f1')"></div>
                <div class="w-8 h-8 rounded-full bg-rose-400 cursor-pointer" onclick="applyColor('#fb7185')"></div>
                <div class="w-8 h-8 rounded-full bg-slate-400 cursor-pointer" onclick="applyColor('#94a3b8')"></div>
                <div class="w-8 h-8 rounded-full bg-black cursor-pointer" onclick="applyColor('#000000')"></div>
            </div>
            <button onclick="closeColorPicker()" class="text-xs text-slate-400 font-bold uppercase pt-2">關閉</button>
        </div>
    </div>

    <input type="file" id="global-uploader" class="hidden" onchange="handleFileSelect(event)">
    <div id="save-toast" class="fixed top-8 left-1/2 -translate-x-1/2 bg-slate-900 text-white px-6 py-3 rounded-full shadow-2xl z-[200] opacity-0 pointer-events-none transition-all duration-500 -translate-y-10">
        <span class="text-xs font-bold tracking-widest uppercase">已更新至雲端</span>
    </div>

    <script>
        const tripDates = ["5/21", "5/22", "5/23", "5/24", "5/25", "5/26", "5/27", "5/28", "5/29"];
        
        // 核心行程資料
        const itineraryData = {
            1: [
                { cat: "交通", time: "--:--", title: "GK28 HKG -> NRT", note: "中午落地 -> 機場巴士/京成線去新宿 (1.5h)", color: "#2a4073" },
                { cat: "景點", time: "下午", title: "新宿區域探索", note: "東京都廳展望台、歌舞伎町、伊勢丹/高島屋購物", color: "#f59e0b" },
                { cat: "景點", time: "晚上", title: "澀谷重點", note: "澀谷十字路口、澀谷Sky、逛街吃飯", color: "#f59e0b" }
            ],
            2: [
                { cat: "購物", time: "全日", title: "淺草全日購物", note: "淺草寺、仲見世街、東京晴空塔、隅田川散步。下午加阿美橫丁/上野", color: "#10b981" },
                { cat: "購物", time: "晚上", title: "新宿購物", note: "回新宿繼續逛", color: "#10b981" }
            ],
            3: [
                { cat: "景點", time: "早晨", title: "鎌倉古都之旅", note: "鎌倉大佛、長谷寺、鶴岡八幡宮、小町通工藝品/甜點", color: "#f59e0b" },
                { cat: "交通", time: "晚上", title: "返回新宿", note: "住宿於新宿", color: "#2a4073" }
            ],
            4: [
                { cat: "租車", time: "早上", title: "東京租車啟程", note: "開車去伊豆 (1.5-2h)。大室山 + 城崎海岸", color: "#10b981" },
                { cat: "重點", time: "下午", title: "熱海佔位", note: "Sun Beach 沙灘區佔位，看大空中ナイアガラ＋海面倒影", color: "#d93535" },
                { cat: "重點", time: "20:20", title: "熱海海上花火大會", note: "20:20–20:40，熱海灣沙灘/親水公園免費觀賞", color: "#d93535" },
                { cat: "住宿", time: "晚上", title: "登坂酒店 入住", note: "THE NOBORISAKA HOTEL, Yamanashi 401-0301", color: "#2a4073" }
            ],
            5: [
                { cat: "遊樂", time: "早上", title: "富士急ハイランド", note: "雲霄飛車、Thomas Land、富士山景", color: "#6366f1" },
                { cat: "景點", time: "下午", title: "河口湖睇富士山", note: "續住登坂酒店", color: "#f59e0b" }
            ],
            6: [
                { cat: "交通", time: "--:--", title: "前往名古屋", note: "河口湖開車移動 -> 名古屋市區", color: "#2a4073" },
                { cat: "購物", time: "下午", title: "大須商店街", note: "名古屋當地生活風格購物", color: "#10b981" }
            ],
            7: [
                { cat: "景點", time: "上午", title: "白川鄉合掌村", note: "展望台拍照、村落散步、參觀合掌造。5月綠意美", color: "#f59e0b" },
                { cat: "重點", time: "下午", title: "立山黑部阿爾卑斯路線", note: "雪之大谷步行(10m+雪牆)、黑部水壩。需預約WEB票", color: "#d93535" },
                { cat: "交通", time: "傍晚", title: "返回名古屋", note: "從立山站開車回名古屋住宿", color: "#2a4073" }
            ],
            8: [
                { cat: "重點", time: "中午", title: "吉卜力樂園", note: "龍貓森林、魔女之谷。停愛・地球博記念公園停車場", color: "#d93535" },
                { cat: "景點", time: "晚上", title: "名古屋城/美食", note: "吃ひつまぶし (鰻魚飯三吃)", color: "#f59e0b" }
            ],
            9: [
                { cat: "景點", time: "早上", title: "市川市動物園", note: "回程前探訪，結束後還車前往成田", color: "#f59e0b" },
                { cat: "交通", time: "--:--", title: "GK29 NRT -> HKG", note: "登機返港", color: "#2a4073" }
            ]
        };

        // 工具箱初始資料
        const toolboxData = [
            { id: 't1', type: 'flight', cat: '航班', title: 'GK28 香港 -> 成田', note: '5/21 12:00 落地預定', color: '#2a4073' },
            { id: 't2', type: 'hotel', cat: '住宿', title: '登坂酒店 (Noborisaka)', note: '5/24-5/26, 富士河口湖町船津6832', color: '#6366f1' },
            { id: 't3', type: 'budget', cat: '預算', title: '旅費總額 (HKD)', note: '機票: 3000 / 住宿: 5000 / 零用: 8000', color: '#f59e0b' }
        ];

        let currentDay = 1;
        let currentFileTarget = null;
        let currentColorTarget = null;

        window.onload = function() {
            renderSidebar();
            renderAllItineraries();
            renderToolbox();
            updateHeader(1);
        };

        function toggleSidebar() {
            const sidebar = document.getElementById('sidebar');
            const overlay = document.getElementById('sidebar-overlay');
            const isClosed = sidebar.classList.contains('closed');
            if (isClosed) {
                sidebar.classList.remove('closed');
                overlay.classList.remove('hidden');
                setTimeout(() => overlay.classList.add('opacity-100'), 10);
            } else {
                sidebar.classList.add('closed');
                overlay.classList.remove('opacity-100');
                setTimeout(() => overlay.classList.add('hidden'), 300);
            }
        }

        function renderSidebar() {
            const list = document.getElementById('side-nav-list');
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
            tripDates.forEach((date, i) => {
                const dayNum = i + 1;
                const section = document.createElement('div');
                section.id = `day-view-${dayNum}`;
                section.className = `day-view space-y-4 ${dayNum === 1 ? 'block' : 'hidden'}`;
                section.innerHTML = `<div class="itinerary-list space-y-4" ondragover="handleDragOver(event)"></div><button onclick="addCard(${dayNum})" class="w-full py-5 border-2 border-dashed border-slate-100 rounded-[2.5rem] text-slate-300 text-xs font-bold hover:border-wa-gold hover:text-wa-gold transition-all">+ 新增 D${dayNum} 行程</button>`;
                const list = section.querySelector('.itinerary-list');
                (itineraryData[dayNum] || []).forEach(data => list.appendChild(createCard(data, 'itinerary')));
                wrapper.appendChild(section);
            });
        }

        function renderToolbox() {
            const list = document.getElementById('toolbox-list');
            list.innerHTML = '';
            toolboxData.forEach(data => list.appendChild(createCard(data, 'toolbox')));
        }

        function createCard(data, mode) {
            const div = document.createElement('div');
            div.className = `bg-white rounded-[2.2rem] shadow-sm border border-slate-50 p-6 space-y-4 relative fade-in-section draggable-item toolbox-item-${data.type || 'none'}`;
            div.draggable = true;
            div.dataset.type = data.type || 'none';
            
            // 根據類型決定按鈕顯示
            const showNav = (data.type === 'hotel' || data.type === 'activity');
            
            div.innerHTML = `
                <div class="absolute top-6 right-6 text-slate-200 cursor-grab active:cursor-grabbing"><i class="fa-solid fa-grip-vertical"></i></div>
                <div class="flex justify-between items-center pr-8">
                    <span onclick="openColorPicker(this)" class="text-[9px] font-black text-white px-3 py-1 rounded-full cursor-pointer editable" style="background-color: ${data.color || '#94a3b8'}" contenteditable="true">${data.cat}</span>
                    <span class="text-[10px] text-slate-300 font-black font-mono editable" contenteditable="true">${data.time || '--:--'}</span>
                </div>
                <div>
                    <h3 class="text-lg font-bold text-slate-800 editable" contenteditable="true">${data.title}</h3>
                    <p class="text-xs text-slate-400 mt-2 leading-relaxed editable" contenteditable="true">${data.note}</p>
                </div>
                <div class="flex gap-2 pt-1">
                    <button onclick="triggerUpload(this)" class="flex-1 py-2 bg-slate-50 text-slate-400 rounded-xl text-[10px] interactive-btn font-bold"><i class="fa-solid fa-paperclip mr-1"></i>上傳文件</button>
                    ${showNav ? `<button onclick="openMap(this)" class="flex-1 py-2 bg-blue-50 text-blue-600 rounded-xl text-[10px] font-bold interactive-btn"><i class="fa-solid fa-location-dot mr-1"></i>導航</button>` : ''}
                    <button onclick="this.closest('.draggable-item').remove()" class="w-10 h-10 bg-rose-50 text-rose-400 rounded-xl text-xs interactive-btn"><i class="fa-solid fa-trash-can"></i></button>
                </div>
                <div class="file-container flex flex-wrap gap-2"></div>
            `;
            addDragEvents(div);
            return div;
        }

        function filterToolbox(type) {
            document.querySelectorAll('.toolbox-nav-btn').forEach(btn => {
                btn.classList.remove('active', 'text-wa-blue');
                btn.classList.add('text-slate-400');
                if(btn.dataset.type === type) {
                    btn.classList.add('active', 'text-wa-blue');
                }
            });

            document.querySelectorAll('#toolbox-list .draggable-item').forEach(card => {
                if(type === 'all' || card.dataset.type === type) {
                    card.style.display = 'block';
                } else {
                    card.style.display = 'none';
                }
            });
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
            window.scrollTo({ top: 0, behavior: 'smooth' });
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
            document.getElementById('top-title').innerText = (page === 'itinerary' ? '2026 日本深度遊' : '旅行工具箱');
            if(page === 'toolbox') {
                document.getElementById('top-subtitle').innerText = '附件與重要資訊管理';
                filterToolbox('all');
            } else {
                updateHeader(currentDay);
            }
        }

        function addToolboxItem(type) {
            const list = document.getElementById('toolbox-list');
            const config = {
                flight: { cat: '航班', color: '#2a4073', title: '新增航班資訊' },
                hotel: { cat: '住宿', color: '#6366f1', title: '新增住宿地點' },
                activity: { cat: '活動', color: '#10b981', title: '預約活動項目' },
                budget: { cat: '預算', color: '#f59e0b', title: '新開支紀錄' }
            };
            const item = createCard({ ...config[type], type, note: '點擊編輯內容...' }, 'toolbox');
            list.insertBefore(item, list.firstChild);
            filterToolbox('all'); // 確保新加入的能看到
            item.scrollIntoView({ behavior: 'smooth' });
        }

        function openMap(btn) {
            const card = btn.closest('.bg-white');
            const title = card.querySelector('h3').innerText;
            const note = card.querySelector('p').innerText;
            window.open(`https://www.google.com/maps/search/${encodeURIComponent(title + ' ' + note)}`);
        }

        // 拖拽排序
        let dragElement = null;
        function addDragEvents(el) {
            el.addEventListener('dragstart', (e) => { dragElement = el; setTimeout(() => el.classList.add('dragging'), 0); });
            el.addEventListener('dragend', () => { el.classList.remove('dragging'); dragElement = null; });
        }
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

        function openColorPicker(el) { currentColorTarget = el; document.getElementById('color-picker').classList.remove('hidden'); }
        function closeColorPicker() { document.getElementById('color-picker').classList.add('hidden'); }
        function applyColor(hex) { if(currentColorTarget) currentColorTarget.style.backgroundColor = hex; closeColorPicker(); }
        function triggerUpload(btn) { currentFileTarget = btn.closest('.bg-white').querySelector('.file-container'); document.getElementById('global-uploader').click(); }
        function handleFileSelect(e) {
            const file = e.target.files[0];
            if (file && currentFileTarget) {
                const tag = document.createElement('div');
                tag.className = 'file-tag mt-2 animate-[fadeIn_0.3s_ease-out]';
                tag.innerHTML = `<i class="fa-solid fa-file-invoice"></i> <span>${file.name.length > 10 ? file.name.substring(0,8)+'...' : file.name}</span> <i class="fa-solid fa-xmark ml-auto opacity-30 cursor-pointer" onclick="this.parentElement.remove()"></i>`;
                currentFileTarget.appendChild(tag);
            }
        }
        function addCard(dayNum) {
            const list = document.querySelector(`#day-view-${dayNum} .itinerary-list`);
            list.appendChild(createCard({ cat: "新項目", time: "--:--", title: "行程標題", note: "細節描述...", color: "#94a3b8" }, 'itinerary'));
        }
        function saveFeedback() {
            const toast = document.getElementById('save-toast');
            toast.style.opacity = '1'; toast.style.transform = 'translateY(0) translateX(-50%)';
            setTimeout(() => { toast.style.opacity = '0'; toast.style.transform = 'translateY(-10px) translateX(-50%)'; }, 2000);
        }
    </script>
</body>
</html>
