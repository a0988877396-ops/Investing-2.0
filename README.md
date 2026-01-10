# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>莫蘭迪財務戰略 v4.0</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --m-blue: #718899; --m-light: #F4F5F7; --m-text: #4A5568; --m-warn: #D69E2E; }
        body { background-color: var(--m-light); color: var(--m-text); font-family: "PingFang TC", sans-serif; }
        .tab-content { display: none; padding-bottom: 100px; }
        .tab-content.active { display: block; animation: fadeIn 0.3s; }
        .muji-card { background: white; border-radius: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.03); margin-bottom: 1rem; padding: 1.5rem; }
        .calendar-day { background: white; min-height: 65px; border: 0.5px solid #EDF2F7; cursor: pointer; transition: 0.2s; }
        .day-over { background: #FFF5F5; } /* 超支警示顏色 */
        .m-btn { background: var(--m-blue); color: white; border-radius: 12px; padding: 12px; width: 100%; font-weight: 500; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body>

    <header class="p-6 bg-white sticky top-0 z-40 border-b border-slate-100 flex justify-between items-center">
        <div class="flex items-center gap-3">
            <button onclick="changeMonth(-1)"><i class="fas fa-angle-left"></i></button>
            <h1 id="month-label" class="text-lg font-medium tracking-tighter">2026 / 01</h1>
            <button onclick="changeMonth(1)"><i class="fas fa-angle-right"></i></button>
        </div>
        <button onclick="importFixed()" class="text-[10px] text-blue-400 border border-blue-100 px-2 py-1 rounded-full">匯入固定支出</button>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card bg-gradient-to-tr from-slate-700 to-slate-500 text-white">
                <p class="text-[10px] opacity-70 mb-1">CURRENT NET WORTH</p>
                <h2 id="total-net" class="text-3xl font-light tracking-tight">$ 0</h2>
                <div class="mt-4 pt-4 border-t border-white/10 text-[10px]">
                    🛡️ 備用金 18 萬：可支撐約 <span id="safety-months">--</span> 個月基本生活
                </div>
            </div>
            <div class="muji-card">
                <h3 class="text-sm font-bold mb-4">資產配置</h3>
                <div class="space-y-4 text-xs">
                    <div class="flex justify-between"><span>其餘現金/儲蓄</span><input type="number" id="in-save" value="0" oninput="saveAssets()" class="text-right border-b"></div>
                    <div class="flex justify-between"><span>投資帳戶</span><input type="number" id="in-stock" value="0" oninput="saveAssets()" class="text-right border-b"></div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card p-0 overflow-hidden">
                <div class="grid grid-cols-7 text-[9px] text-center py-2 bg-slate-50 text-slate-400">
                    <div>SUN</div><div>MON</div><div>TUE</div><div>WED</div><div>THU</div><div>FRI</div><div>SAT</div>
                </div>
                <div id="calendar-grid" class="grid grid-cols-7 border-t border-slate-50"></div>
            </div>
            <div class="muji-card">
                <div class="flex justify-between items-center mb-3">
                    <span class="text-xs font-bold">每日預算分析</span>
                    <span id="daily-avg" class="text-[10px] text-slate-400">平均每日花費: $0</span>
                </div>
                <div class="w-full bg-slate-100 h-1.5 rounded-full overflow-hidden">
                    <div id="budget-bar" class="h-full bg-blue-400 transition-all duration-500"></div>
                </div>
            </div>
        </div>

        <div id="tab-stats" class="tab-content">
            <div class="muji-card">
                <canvas id="statChart" class="mb-4"></canvas>
                <div id="compare-logic" class="text-[11px] text-slate-500 space-y-2"></div>
            </div>
            <div class="muji-card bg-blue-50/50">
                <h3 class="text-sm font-bold mb-4 text-slate-700">職涯 ROI 矩陣</h3>
                <div id="roi-container" class="space-y-2"></div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t flex justify-around p-4 z-50">
        <button onclick="switchTab('assets')" class="nav-btn active flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" class="nav-btn flex flex-col items-center"><i class="fas fa-edit text-lg mb-1"></i><span>記帳</span></button>
        <button onclick="switchTab('stats')" class="nav-btn flex flex-col items-center"><i class="fas fa-chart-line text-lg mb-1"></i><span>分析</span></button>
    </nav>

    <div id="modal" class="hidden fixed inset-0 bg-black/40 z-[100] flex items-end">
        <div class="bg-white w-full rounded-t-3xl p-6 animate-slide-up">
            <div class="flex justify-between mb-4"><h3 id="m-date" class="font-bold"></h3><button onclick="closeModal()">✕</button></div>
            <div id="m-list" class="space-y-2 mb-4 max-h-32 overflow-y-auto"></div>
            <div class="flex flex-wrap gap-2 mb-4" id="cat-tags"></div>
            <div class="flex gap-2 mb-4">
                <input type="text" id="m-note" placeholder="備註" class="flex-1 border-b text-sm">
                <input type="number" id="m-amt" placeholder="金額" class="w-20 border-b text-sm text-right">
            </div>
            <button onclick="addEntry()" class="m-btn">儲存項目</button>
        </div>
    </div>

    <script>
        let curY = 2026, curM = 0;
        let data = JSON.parse(localStorage.getItem('finance_v4')) || { assets: {}, records: {} };
        let cats = ["食物費", "交通費", "健身費", "生活用品"];
        let selectedCat = "食物費";
        let activeKey = "";
        let chart = null;

        function switchTab(t) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach((b,i) => b.classList.toggle('active', ['assets','calendar','stats'][i] === t));
            document.getElementById('tab-'+t).classList.add('active');
            if(t === 'stats') renderStats();
            if(t === 'assets') updateAssetsUI();
        }

        function changeMonth(s) { curM += s; if(curM>11){curM=0;curY++} if(curM<0){curM=11;curY--} refresh(); }
        
        function refresh() {
            document.getElementById('month-label').innerText = `${curY} / ${String(curM+1).padStart(2,'0')}`;
            const grid = document.getElementById('calendar-grid'); grid.innerHTML = '';
            const first = new Date(curY, curM, 1).getDay();
            const days = new Date(curY, curM+1, 0).getDate();
            const dailyBudget = 32000 / days;

            for(let i=0; i<first; i++) grid.innerHTML += `<div class="bg-slate-50/50"></div>`;
            for(let d=1; d<=days; d++) {
                const key = `${curY}-${curM}-${d}`;
                const total = (data.records[key] || []).reduce((s,r)=>s+r.amt, 0);
                const isOver = total > dailyBudget;
                grid.innerHTML += `
                    <div class="calendar-day p-1 ${isOver?'day-over':''}" onclick="openModal('${key}')">
                        <span class="text-[10px] opacity-50">${d}</span>
                        <div class="text-[9px] font-bold text-slate-400 mt-1">${total>0?'$'+total:''}</div>
                    </div>`;
            }
            updateSummary();
        }

        function openModal(k) {
            activeKey = k;
            document.getElementById('m-date').innerText = k;
            document.getElementById('modal').classList.remove('hidden');
            renderModalList();
            const tags = document.getElementById('cat-tags');
            tags.innerHTML = cats.map(c => `<span class="text-[10px] px-3 py-1 border rounded-full cursor-pointer ${selectedCat===c?'bg-slate-600 text-white':''}" onclick="selectedCat='${c}';openModal('${k}')">${c}</span>`).join('');
        }

        function addEntry() {
            const amt = Number(document.getElementById('m-amt').value);
            if(!amt) return;
            if(!data.records[activeKey]) data.records[activeKey] = [];
            data.records[activeKey].push({cat: selectedCat, note: document.getElementById('m-note').value, amt});
            localStorage.setItem('finance_v4', JSON.stringify(data));
            document.getElementById('m-amt').value=''; document.getElementById('m-note').value='';
            renderModalList(); refresh();
        }

        function renderModalList() {
            document.getElementById('m-list').innerHTML = (data.records[activeKey]||[]).map(r => `<div class="text-xs flex justify-between bg-slate-50 p-2 rounded"><span>${r.cat}-${r.note}</span><b>$${r.amt}</b></div>`).join('');
        }

        function updateSummary() {
            let mTotal = 0;
            for(let k in data.records) if(k.startsWith(`${curY}-${curM}-`)) data.records[k].forEach(r=>mTotal+=r.amt);
            const days = new Date(curY, curM+1, 0).getDate();
            const avg = (mTotal/days).toFixed(0);
            document.getElementById('daily-avg').innerText = `平均每日花費: $${avg}`;
            document.getElementById('budget-bar').style.width = Math.min((mTotal/32000)*100, 100) + '%';
        }

        function saveAssets() {
            data.assets.save = Number(document.getElementById('in-save').value);
            data.assets.stock = Number(document.getElementById('in-stock').value);
            localStorage.setItem('finance_v4', JSON.stringify(data));
            updateAssetsUI();
        }

        function updateAssetsUI() {
            const total = 180000 + (data.assets.save||0) + (data.assets.stock||0);
            document.getElementById('total-net').innerText = `$ ${total.toLocaleString()}`;
            document.getElementById('safety-months').innerText = (180000 / 16500).toFixed(1);
        }

        function renderStats() {
            const ctx = document.getElementById('statChart').getContext('2d');
            const catsData = {};
            for(let k in data.records) if(k.startsWith(`${curY}-${curM}-`)) data.records[k].forEach(r=>catsData[r.cat]=(catsData[r.cat]||0)+r.amt);
            if(chart) chart.destroy();
            chart = new Chart(ctx, {
                type: 'doughnut',
                data: { labels: Object.keys(catsData), datasets: [{ data: Object.values(catsData), backgroundColor: ['#718899','#A2B5BB','#CBD5E0','#4A5568'] }] },
                options: { cutout: '70%', plugins: { legend: { position: 'bottom', labels: { boxWidth: 10, font: { size: 10 } } } } }
            });
            renderROI();
        }

        function renderROI() {
            const gain = (55000-32000)*12;
            document.getElementById('roi-container').innerHTML = [1,3,5].map(y => `
                <div class="flex justify-between text-xs py-2 border-b border-blue-100">
                    <span>第 ${y} 年累積回報</span>
                    <span class="font-bold text-blue-600">$${(gain*y).toLocaleString()} (ROI: ${((gain*y/55000)*100).toFixed(0)}%)</span>
                </div>
            `).join('');
        }

        function importFixed() {
            const key = `${curY}-${curM}-1`;
            if(!data.records[key]) data.records[key] = [];
            data.records[key].push({cat: "生活用品", note: "固定支出匯入", amt: 15500});
            localStorage.setItem('finance_v4', JSON.stringify(data));
            refresh(); alert("已自動匯入固定支出至本月 1 號");
        }

        function closeModal() { document.getElementById('modal').classList.add('hidden'); }

        window.onload = () => { 
            document.getElementById('in-save').value = data.assets.save || 0;
            document.getElementById('in-stock').value = data.assets.stock || 0;
            refresh(); updateAssetsUI(); 
        };
    </script>
</body>
</html>
