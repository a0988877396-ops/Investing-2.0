# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>莫蘭迪財務管家 v3.0</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --m-blue: #718899; --m-light-blue: #A2B5BB; --m-bg: #F4F5F7; --m-text: #4A5568; --m-gray: #CBD5E0; }
        body { background-color: var(--m-bg); color: var(--m-text); font-family: "PingFang TC", sans-serif; }
        .muji-card { background: white; border-radius: 16px; margin-bottom: 1rem; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05); }
        .nav-btn { color: var(--m-gray); transition: 0.3s; font-size: 10px; }
        .nav-btn.active { color: var(--m-blue); }
        .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 1px; background: #EDF2F7; }
        .calendar-day { background: white; min-height: 60px; padding: 4px; font-size: 11px; cursor: pointer; display: flex; flex-direction: column; }
        .day-total { color: var(--m-blue); font-weight: bold; font-size: 9px; margin-top: auto; overflow: hidden; white-space: nowrap; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 100; align-items: flex-end; }
        .modal.active { display: flex; }
        .cat-chip { padding: 4px 12px; border-radius: 20px; border: 1px solid var(--m-gray); font-size: 11px; cursor: pointer; }
        .cat-chip.active { background: var(--m-blue); color: white; border-color: var(--m-blue); }
    </style>
</head>
<body class="pb-24">

    <header class="p-5 bg-white flex justify-between items-center sticky top-0 z-40">
        <div class="flex items-center gap-2">
            <button onclick="changeMonth(-1)" class="text-slate-400"><i class="fas fa-chevron-left"></i></button>
            <h1 id="month-label" class="text-lg font-medium tracking-widest text-slate-700">2026.01</h1>
            <button onclick="changeMonth(1)" class="text-slate-400"><i class="fas fa-chevron-right"></i></button>
        </div>
        <span id="header-subtitle" class="text-xs text-slate-400">記帳本</span>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-8 text-center bg-gradient-to-br from-slate-50 to-white">
                <p class="text-[11px] text-slate-400 uppercase tracking-widest mb-2">Total Assets</p>
                <h2 id="total-net-display" class="text-4xl font-light text-slate-800">$ 180,000</h2>
            </div>
            <div class="muji-card p-6">
                <h3 class="font-bold text-sm mb-4">基礎資產明細</h3>
                <div class="space-y-4 text-sm">
                    <div class="flex justify-between"><span>🛡️ 緊急備用金</span><span class="font-bold text-blue-400">$ 180,000</span></div>
                    <div class="flex justify-between"><span>其他儲蓄/現金</span><input type="number" id="other-save" value="0" oninput="calcNet()" class="text-right w-1/3 border-b"></div>
                    <div class="flex justify-between"><span>投資總市值</span><input type="number" id="in-stock" value="0" oninput="calcNet()" class="text-right w-1/3 border-b"></div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card overflow-hidden mb-4">
                <div class="grid grid-cols-7 text-center text-[10px] py-2 bg-slate-50 text-slate-400">
                    <div>SUN</div><div>MON</div><div>TUE</div><div>WED</div><div>THU</div><div>FRI</div><div>SAT</div>
                </div>
                <div id="calendar-body" class="calendar-grid"></div>
            </div>
            <div class="muji-card p-5">
                <div class="flex justify-between text-xs mb-3">
                    <span>本月總支出：<b id="month-spent-amt">$ 0</b></span>
                    <span>預算剩餘：<b id="month-budget-left">$ 32,000</b></span>
                </div>
                <div id="quick-strategy" class="text-[10px] p-3 bg-slate-50 rounded-lg text-slate-500">正在分析本月開銷...</div>
            </div>
        </div>

        <div id="tab-stats" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4">類別支出比例</h3>
                <canvas id="categoryChart" style="max-height: 250px;"></canvas>
            </div>
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4">月度交叉對比</h3>
                <div id="compare-text" class="text-xs text-slate-500 leading-relaxed"></div>
            </div>
        </div>

        <div id="tab-strategy" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-blue-400 pl-3">職涯投資 ROI 矩陣</h3>
                <table class="w-full text-xs text-left">
                    <thead class="text-gray-400"><tr><th class="py-2">年度</th><th>收益</th><th>回報率</th></tr></thead>
                    <tbody id="roi-matrix"></tbody>
                </table>
            </div>
        </div>
    </main>

    <div id="record-modal" class="modal" onclick="if(event.target==this) closeModal()">
        <div class="bg-white w-full rounded-t-3xl p-6 shadow-2xl animate-slide-up">
            <h3 id="modal-date" class="font-bold text-slate-700 mb-4"></h3>
            <div id="modal-records-list" class="space-y-2 mb-4 max-h-32 overflow-y-auto"></div>
            
            <div class="border-t pt-4 space-y-4">
                <div class="flex flex-wrap gap-2" id="category-list">
                    </div>
                <div class="flex gap-2">
                    <input type="text" id="new-cat" placeholder="新增類別" class="w-1/4 text-xs">
                    <input type="text" id="temp-note" placeholder="備註" class="w-2/4 text-sm">
                    <input type="number" id="temp-amt" placeholder="金額" class="w-1/4 text-sm text-right">
                </div>
                <button onclick="saveEntry()" class="w-full bg-slate-600 text-white py-3 rounded-xl text-sm font-bold">確認新增</button>
            </div>
        </div>
    </div>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/90 backdrop-blur-md border-t flex justify-around p-4 z-50">
        <button onclick="switchTab('assets')" class="nav-btn active flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" class="nav-btn flex flex-col items-center"><i class="fas fa-calendar-alt text-lg mb-1"></i><span>記帳</span></button>
        <button onclick="switchTab('stats')" class="nav-btn flex flex-col items-center"><i class="fas fa-chart-pie text-lg mb-1"></i><span>統計分析</span></button>
        <button onclick="switchTab('strategy')" class="nav-btn flex flex-col items-center"><i class="fas fa-chess-knight text-lg mb-1"></i><span>戰略</span></button>
    </nav>

    <script>
        let currentYear = 2026, currentMonth = 0; // 0 = 1月
        let db = JSON.parse(localStorage.getItem('muji_v3_db')) || {};
        let categories = ["食物費", "交通費", "健身費", "生活用品"];
        let activeCat = "食物費";
        let selectedDate = "";
        let chartInstance = null;

        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach((b, i) => b.classList.toggle('active', ['assets','calendar','stats','strategy'][i] === name));
            document.getElementById('tab-' + name).classList.add('active');
            if(name === 'stats') renderStats();
            if(name === 'strategy') renderROI();
        }

        function changeMonth(step) {
            currentMonth += step;
            if(currentMonth > 11) { currentMonth = 0; currentYear++; }
            if(currentMonth < 0) { currentMonth = 11; currentYear--; }
            document.getElementById('month-label').innerText = `${currentYear}.${String(currentMonth+1).padStart(2,'0')}`;
            initCalendar();
            updateMonthSummary();
        }

        function initCalendar() {
            const body = document.getElementById('calendar-body'); body.innerHTML = '';
            const firstDay = new Date(currentYear, currentMonth, 1).getDay();
            const days = new Date(currentYear, currentMonth + 1, 0).getDate();
            for(let i=0; i<firstDay; i++) body.innerHTML += `<div class="calendar-day bg-slate-50/50"></div>`;
            for(let d=1; d<=days; d++) {
                const dateKey = `${currentYear}-${currentMonth}-${d}`;
                const total = (db[dateKey] || []).reduce((s, r) => s + r.amt, 0);
                body.innerHTML += `<div class="calendar-day" onclick="openModal('${dateKey}')"><span>${d}</span><span class="day-total">${total>0?'$'+total:''}</span></div>`;
            }
        }

        function openModal(key) {
            selectedDate = key;
            document.getElementById('modal-date').innerText = key.replace(/-/g, '/');
            renderModalRecords();
            renderCategories();
            document.getElementById('record-modal').classList.add('active');
        }

        function renderCategories() {
            const list = document.getElementById('category-list');
            list.innerHTML = categories.map(c => `<span class="cat-chip ${activeCat===c?'active':''}" onclick="setCat('${c}')">${c}</span>`).join('');
        }
        function setCat(c) { activeCat = c; renderCategories(); }

        function saveEntry() {
            const newCat = document.getElementById('new-cat').value;
            const finalCat = newCat || activeCat;
            const amt = Number(document.getElementById('temp-amt').value);
            const note = document.getElementById('temp-note').value;
            if(!amt) return;
            if(newCat && !categories.includes(newCat)) categories.push(newCat);
            if(!db[selectedDate]) db[selectedDate] = [];
            db[selectedDate].push({cat: finalCat, note, amt});
            localStorage.setItem('muji_v3_db', JSON.stringify(db));
            document.getElementById('temp-amt').value = ''; document.getElementById('new-cat').value = '';
            renderModalRecords(); initCalendar(); updateMonthSummary();
        }

        function renderModalRecords() {
            const list = document.getElementById('modal-records-list');
            list.innerHTML = (db[selectedDate] || []).map((r, i) => `<div class="flex justify-between text-xs p-2 bg-slate-50 rounded"><span>[${r.cat}] ${r.note}</span><b>$${r.amt}</b></div>`).join('');
        }

        function updateMonthSummary() {
            let total = 0;
            for(let key in db) if(key.startsWith(`${currentYear}-${currentMonth}-`)) db[key].forEach(r => total += r.amt);
            document.getElementById('month-spent-amt').innerText = `$ ${total.toLocaleString()}`;
            document.getElementById('month-budget-left').innerText = `$ ${Math.max(32000-total, 0).toLocaleString()}`;
        }

        function renderStats() {
            const data = {};
            for(let key in db) if(key.startsWith(`${currentYear}-${currentMonth}-`)) {
                db[key].forEach(r => data[r.cat] = (data[r.cat] || 0) + r.amt);
            }
            const ctx = document.getElementById('categoryChart').getContext('2d');
            if(chartInstance) chartInstance.destroy();
            chartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: Object.keys(data),
                    datasets: [{ data: Object.values(data), backgroundColor: ['#718899', '#A2B5BB', '#CBD5E0', '#4A5568', '#E2E8F0'] }]
                }
            });
            // 交叉對比建議
            const lastMonthTotal = getMonthTotal(currentYear, currentMonth - 1);
            const thisMonthTotal = Object.values(data).reduce((a,b)=>a+b, 0);
            const diff = thisMonthTotal - lastMonthTotal;
            document.getElementById('compare-text').innerHTML = `
                上月總支出：$${lastMonthTotal.toLocaleString()}<br>
                本月總支出：$${thisMonthTotal.toLocaleString()}<br>
                <b>分析建議：</b> ${diff > 0 ? `本月支出增加了 $${diff}，主要增量來自 ${Object.keys(data)[0] || '無'}。` : '表現優異，本月支出較上月下降！'}
            `;
        }

        function getMonthTotal(y, m) {
            let t = 0; for(let k in db) if(k.startsWith(`${y}-${m}-`)) db[k].forEach(r => t += r.amt); return t;
        }

        function renderROI() {
            const body = document.getElementById('roi-matrix'); body.innerHTML = '';
            const cost = 55000; const gain = (55000-32000)*12;
            for(let i=1; i<=5; i++) {
                body.innerHTML += `<tr><td class="py-2">第 ${i} 年</td><td>$${(gain*i).toLocaleString()}</td><td class="text-blue-500 font-bold">${((gain*i/cost)*100).toFixed(0)}%</td></tr>`;
            }
        }

        function closeModal() { document.getElementById('record-modal').classList.remove('active'); }
        function calcNet() { 
            const total = 180000 + (Number(document.getElementById('other-save').value)||0) + (Number(document.getElementById('in-stock').value)||0);
            document.getElementById('total-net-display').innerText = '$ ' + total.toLocaleString();
        }

        window.onload = () => { initCalendar(); updateMonthSummary(); calcNet(); };
    </script>
</body>
</html>
