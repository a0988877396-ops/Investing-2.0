# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>莫蘭迪財務戰略 v6.0</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --m-blue: #718899; --m-light: #F4F5F7; --m-text: #4A5568; --m-gray: #CBD5E0; }
        body { background-color: var(--m-light); color: var(--m-text); font-family: "PingFang TC", sans-serif; }
        .tab-content { display: none; padding-bottom: 120px; }
        .tab-content.active { display: block; animation: fadeIn 0.3s; }
        .muji-card { background: white; border-radius: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.03); margin-bottom: 1rem; padding: 1.5rem; }
        .calendar-day { background: white; min-height: 65px; border: 0.5px solid #EDF2F7; cursor: pointer; display: flex; flex-direction: column; }
        .day-total { color: var(--m-blue); font-weight: bold; font-size: 10px; margin-top: auto; text-align: center; }
        .nav-btn { color: var(--m-gray); transition: 0.3s; font-size: 10px; }
        .nav-btn.active { color: var(--m-blue); }
        input[type=range] { accent-color: var(--m-blue); }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 100; align-items: flex-end; }
        .modal.active { display: flex; }
    </style>
</head>
<body>

    <header class="p-5 bg-white sticky top-0 z-40 border-b border-slate-100 flex justify-between items-center">
        <div class="flex items-center gap-3">
            <button onclick="changeMonth(-1)"><i class="fas fa-angle-left"></i></button>
            <h1 id="month-label" class="text-lg font-medium tracking-tighter">2026 / 01</h1>
            <button onclick="changeMonth(1)"><i class="fas fa-angle-right"></i></button>
        </div>
        <span class="text-[10px] text-slate-300">v6.0 FIRE 版</span>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card bg-gradient-to-tr from-slate-700 to-slate-500 text-white border-none">
                <p class="text-[10px] opacity-70 mb-1">TOTAL ASSETS</p>
                <h2 id="total-net" class="text-3xl font-light tracking-tight">$ 0</h2>
                <div class="mt-4 pt-4 border-t border-white/10 text-[10px]">
                    🛡️ 備用金 18 萬：可支撐約 <span id="safety-months">--</span> 個月生活
                </div>
            </div>
            <div class="muji-card">
                <h3 class="text-sm font-bold mb-4">基礎資產明細</h3>
                <div class="space-y-4 text-sm">
                    <div class="flex justify-between"><span>🛡️ 緊急備用金</span><span class="font-bold text-blue-400">$ 180,000</span></div>
                    <div class="flex justify-between"><span>其餘儲蓄</span><input type="number" id="in-save" value="0" oninput="saveAssets()" class="text-right border-b w-1/3 outline-none"></div>
                    <div class="flex justify-between"><span>投資市值</span><input type="number" id="in-stock" value="0" oninput="saveAssets()" class="text-right border-b w-1/3 outline-none"></div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card p-0 overflow-hidden mb-4 border border-slate-50">
                <div class="grid grid-cols-7 text-[9px] text-center py-2 bg-slate-50 text-slate-400">
                    <div>SUN</div><div>MON</div><div>TUE</div><div>WED</div><div>THU</div><div>FRI</div><div>SAT</div>
                </div>
                <div id="calendar-grid" class="grid grid-cols-7"></div>
            </div>
            <div class="muji-card">
                <div class="flex justify-between text-xs mb-2"><span>本月總支出</span><span id="month-total-display" class="font-bold">$ 0</span></div>
                <p class="text-[10px] text-slate-400 italic">點擊日期記錄多筆開銷，打錯可隨時刪除。</p>
            </div>
        </div>

        <div id="tab-stats" class="tab-content">
            <div class="muji-card border-2 border-blue-50">
                <h3 class="text-sm font-bold mb-4 flex items-center gap-2">💰 複利與提領模擬 (FIRE)</h3>
                
                <div class="space-y-6">
                    <div>
                        <div class="flex justify-between text-[11px] mb-2"><span>預期年化報酬率</span><span class="font-bold text-blue-500" id="val-roi">7.0 %</span></div>
                        <input type="range" min="1" max="15" step="0.5" value="7" class="w-full" id="range-roi" oninput="runFireSim()">
                    </div>
                    
                    <div>
                        <div class="flex justify-between text-[11px] mb-2"><span>每月額外投入金額</span><span class="font-bold text-blue-500" id="val-monthly">$ 10,000</span></div>
                        <input type="range" min="0" max="50000" step="1000" value="10000" class="w-full" id="range-monthly" oninput="runFireSim()">
                    </div>

                    <div class="p-4 bg-slate-50 rounded-2xl space-y-3">
                        <div class="flex justify-between text-xs"><span>10 年後資產預估</span><span id="res-10y" class="font-bold">--</span></div>
                        <div class="flex justify-between text-xs"><span>20 年後資產預估</span><span id="res-20y" class="font-bold">--</span></div>
                        <div class="flex justify-between text-xs border-t pt-2 border-slate-200">
                            <span class="text-blue-600 font-bold">每年可提領(4%法則)</span>
                            <span id="res-withdraw" class="font-bold text-blue-600">--</span>
                        </div>
                    </div>
                </div>
            </div>

            <div class="muji-card">
                <h3 class="text-sm font-bold mb-4">本月類別比例</h3>
                <canvas id="statChart"></canvas>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t flex justify-around p-4 z-50">
        <button onclick="switchTab('assets')" id="nav-assets" class="nav-btn active flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" id="nav-calendar" class="nav-btn flex flex-col items-center"><i class="fas fa-edit text-lg mb-1"></i><span>記帳</span></button>
        <button onclick="switchTab('stats')" id="nav-stats" class="nav-btn flex flex-col items-center"><i class="fas fa-chess-knight text-lg mb-1"></i><span>戰略</span></button>
    </nav>

    <div id="record-modal" class="modal" onclick="if(event.target==this) closeModal()">
        <div class="bg-white w-full rounded-t-3xl p-6 shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <h3 id="modal-date" class="font-bold text-slate-700"></h3>
                <button onclick="closeModal()" class="text-slate-300">✕</button>
            </div>
            <div id="modal-records-list" class="space-y-2 mb-6 max-h-40 overflow-y-auto"></div>
            <div class="border-t pt-4 space-y-4">
                <div class="flex flex-wrap gap-2" id="cat-list"></div>
                <div class="flex gap-2">
                    <input type="text" id="new-cat" placeholder="新類別" class="w-1/4 text-[10px] border-b outline-none">
                    <input type="text" id="temp-note" placeholder="項目" class="flex-1 text-sm border-b outline-none">
                    <input type="number" id="temp-amt" placeholder="金額" class="w-20 text-sm text-right border-b outline-none">
                </div>
                <button onclick="saveEntry()" class="w-full bg-slate-600 text-white py-3 rounded-xl text-sm font-bold shadow-lg">+ 儲存</button>
            </div>
        </div>
    </div>

    <script>
        let curY = 2026, curM = 0;
        let data = JSON.parse(localStorage.getItem('finance_v6')) || { assets: {save:0, stock:0}, records: {} };
        let categories = ["食物費", "交通費", "健身費", "生活用品"];
        let activeCat = "食物費";
        let activeDateKey = "";
        let chartInstance = null;

        function switchTab(t) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-'+t).classList.add('active');
            document.getElementById('nav-'+t).classList.add('active');
            if(t === 'stats') { renderStats(); runFireSim(); }
            if(t === 'assets') updateAssetsUI();
        }

        function changeMonth(s) { curM += s; if(curM>11){curM=0;curY++} if(curM<0){curM=11;curY--} refresh(); }

        function refresh() {
            document.getElementById('month-label').innerText = `${curY} / ${String(curM+1).padStart(2,'0')}`;
            const grid = document.getElementById('calendar-grid'); grid.innerHTML = '';
            const first = new Date(curY, curM, 1).getDay();
            const days = new Date(curY, curM+1, 0).getDate();
            for(let i=0; i<first; i++) grid.innerHTML += `<div class="bg-slate-50/50"></div>`;
            for(let d=1; d<=days; d++) {
                const key = `${curY}-${curM}-${d}`;
                const total = (data.records[key] || []).reduce((s,r)=>s+r.amt, 0);
                grid.innerHTML += `<div class="calendar-day p-1" onclick="openModal('${key}')"><span class="text-[10px] opacity-40">${d}</span><div class="day-total">${total>0?'$'+total:''}</div></div>`;
            }
            let mTotal = 0; for(let k in data.records) if(k.startsWith(`${curY}-${curM}-`)) data.records[k].forEach(r=>mTotal+=r.amt);
            document.getElementById('month-total-display').innerText = `$ ${mTotal.toLocaleString()}`;
        }

        function openModal(key) {
            activeDateKey = key; document.getElementById('modal-date').innerText = key;
            renderModalRecords(); renderCatChips();
            document.getElementById('record-modal').classList.add('active');
        }

        function renderCatChips() {
            document.getElementById('cat-list').innerHTML = categories.map(c => `<span class="text-[11px] px-3 py-1 border rounded-full cursor-pointer ${activeCat===c?'bg-slate-600 text-white':''}" onclick="activeCat='${c}';renderCatChips()">${c}</span>`).join('');
        }

        function saveEntry() {
            const amt = Number(document.getElementById('temp-amt').value); if(!amt) return;
            const newCat = document.getElementById('new-cat').value;
            const cat = newCat || activeCat;
            if(newCat && !categories.includes(newCat)) categories.push(newCat);
            if(!data.records[activeDateKey]) data.records[activeDateKey] = [];
            data.records[activeDateKey].push({cat, note: document.getElementById('temp-note').value, amt});
            localStorage.setItem('finance_v6', JSON.stringify(data));
            document.getElementById('temp-amt').value = ''; document.getElementById('temp-note').value = '';
            renderModalRecords(); refresh();
        }

        function renderModalRecords() {
            const items = data.records[activeDateKey] || [];
            document.getElementById('modal-records-list').innerHTML = items.map((r, i) => `
                <div class="flex justify-between items-center bg-slate-50 p-2 rounded-xl text-xs">
                    <span>[${r.cat}] ${r.note}</span>
                    <div class="flex items-center gap-2 font-bold">$${r.amt} <button onclick="deleteItem(${i})" class="text-slate-300">✕</button></div>
                </div>`).join('') || '<p class="text-center text-slate-300 py-4">無紀錄</p>';
        }

        function deleteItem(i) {
            data.records[activeDateKey].splice(i, 1);
            if(!data.records[activeDateKey].length) delete data.records[activeDateKey];
            localStorage.setItem('finance_v6', JSON.stringify(data)); renderModalRecords(); refresh();
        }

        function saveAssets() {
            data.assets.save = Number(document.getElementById('in-save').value);
            data.assets.stock = Number(document.getElementById('in-stock').value);
            localStorage.setItem('finance_v6', JSON.stringify(data)); updateAssetsUI();
        }

        function updateAssetsUI() {
            const total = 180000 + (data.assets.save||0) + (data.assets.stock||0);
            document.getElementById('total-net').innerText = `$ ${total.toLocaleString()}`;
            document.getElementById('safety-months').innerText = (180000 / 16500).toFixed(1);
        }

        // --- FIRE 模擬器邏輯 ---
        function runFireSim() {
            const roi = parseFloat(document.getElementById('range-roi').value) / 100;
            const monthly = parseInt(document.getElementById('range-monthly').value);
            const currentStock = data.assets.stock || 0;

            document.getElementById('val-roi').innerText = (roi * 100).toFixed(1) + ' %';
            document.getElementById('val-monthly').innerText = '$ ' + monthly.toLocaleString();

            const calcCompund = (years) => {
                let total = currentStock;
                for (let i = 0; i < years * 12; i++) {
                    total = total * (1 + roi / 12) + monthly;
                }
                return total;
            };

            const res10 = calcCompund(10);
            const res20 = calcCompund(20);

            document.getElementById('res-10y').innerText = '$ ' + Math.round(res10).toLocaleString();
            document.getElementById('res-20y').innerText = '$ ' + Math.round(res20).toLocaleString();
            document.getElementById('res-withdraw').innerText = '$ ' + Math.round(res20 * 0.04).toLocaleString() + ' / 年';
        }

        function renderStats() {
            const catsData = {};
            for(let k in data.records) if(k.startsWith(`${curY}-${curM}-`)) data.records[k].forEach(r=>catsData[r.cat]=(catsData[r.cat]||0)+r.amt);
            const ctx = document.getElementById('statChart').getContext('2d');
            if(chartInstance) chartInstance.destroy();
            chartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: { labels: Object.keys(catsData), datasets: [{ data: Object.values(catsData), backgroundColor: ['#718899','#A2B5BB','#CBD5E0','#4A5568'] }] },
                options: { plugins: { legend: { position: 'bottom' } } }
            });
        }

        function closeModal() { document.getElementById('record-modal').classList.remove('active'); }

        window.onload = () => {
            document.getElementById('in-save').value = data.assets.save;
            document.getElementById('in-stock').value = data.assets.stock;
            refresh(); updateAssetsUI(); runFireSim();
        };
    </script>
</body>
</html>
