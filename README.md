# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>莫蘭迪財務管家 v5.0</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --m-blue: #718899; --m-light: #F4F5F7; --m-text: #4A5568; --m-gray: #CBD5E0; }
        body { background-color: var(--m-light); color: var(--m-text); font-family: "PingFang TC", sans-serif; }
        .tab-content { display: none; padding-bottom: 100px; }
        .tab-content.active { display: block; animation: fadeIn 0.3s; }
        .muji-card { background: white; border-radius: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.03); margin-bottom: 1rem; padding: 1.5rem; }
        .calendar-day { background: white; min-height: 70px; border: 0.5px solid #EDF2F7; cursor: pointer; display: flex; flex-direction: column; transition: 0.2s; }
        .calendar-day:active { background: #f0f4f8; }
        .day-total { color: var(--m-blue); font-weight: bold; font-size: 10px; margin-top: auto; white-space: nowrap; overflow: hidden; }
        .nav-btn { color: var(--m-gray); transition: 0.3s; font-size: 10px; }
        .nav-btn.active { color: var(--m-blue); }
        .cat-chip { padding: 4px 12px; border-radius: 20px; border: 1px solid var(--m-gray); font-size: 11px; cursor: pointer; transition: 0.2s; }
        .cat-chip.active { background: var(--m-blue); color: white; border-color: var(--m-blue); }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 100; align-items: flex-end; }
        .modal.active { display: flex; }
    </style>
</head>
<body>

    <header class="p-5 bg-white sticky top-0 z-40 border-b border-slate-100 flex justify-between items-center">
        <div class="flex items-center gap-3">
            <button onclick="changeMonth(-1)" class="p-1"><i class="fas fa-angle-left"></i></button>
            <h1 id="month-label" class="text-lg font-medium tracking-tighter">2026 / 01</h1>
            <button onclick="changeMonth(1)" class="p-1"><i class="fas fa-angle-right"></i></button>
        </div>
        <button onclick="importFixed()" class="text-[10px] text-blue-400 border border-blue-100 px-3 py-1 rounded-full">匯入固定支出</button>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card bg-gradient-to-tr from-slate-700 to-slate-500 text-white border-none">
                <p class="text-[10px] opacity-70 mb-1">TOTAL NET WORTH</p>
                <h2 id="total-net" class="text-3xl font-light tracking-tight">$ 180,000</h2>
                <div class="mt-4 pt-4 border-t border-white/10 text-[10px]">
                    🛡️ 備用金 18 萬：可支撐約 <span id="safety-months">--</span> 個月生活
                </div>
            </div>
            <div class="muji-card">
                <h3 class="text-sm font-bold mb-6 flex items-center gap-2"><i class="fas fa-wallet text-blue-300"></i> 資產配置</h3>
                <div class="space-y-6 text-sm">
                    <div class="flex justify-between items-center">
                        <span>🛡️ 緊急備用金</span>
                        <span class="font-bold text-blue-400">$ 180,000</span>
                    </div>
                    <div class="flex justify-between items-center">
                        <span>其他儲蓄 / 現金</span>
                        <input type="number" id="in-save" value="0" oninput="saveAssets()" class="text-right w-1/3 border-b border-slate-100 focus:border-blue-300 outline-none">
                    </div>
                    <div class="flex justify-between items-center">
                        <span>投資市值</span>
                        <input type="number" id="in-stock" value="0" oninput="saveAssets()" class="text-right w-1/3 border-b border-slate-100 focus:border-blue-300 outline-none">
                    </div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card p-0 overflow-hidden mb-4">
                <div class="grid grid-cols-7 text-[9px] text-center py-2 bg-slate-50 text-slate-400">
                    <div>SUN</div><div>MON</div><div>TUE</div><div>WED</div><div>THU</div><div>FRI</div><div>SAT</div>
                </div>
                <div id="calendar-grid" class="grid grid-cols-7"></div>
            </div>
            <div class="muji-card">
                <div class="flex justify-between items-center mb-2">
                    <span class="text-xs font-bold">本月分析</span>
                    <span id="month-total-text" class="text-xs text-slate-500">$ 0</span>
                </div>
                <div id="quick-advice" class="text-[10px] text-slate-400 leading-relaxed">紀錄一個月後將為您分析支出比例。</div>
            </div>
        </div>

        <div id="tab-stats" class="tab-content">
            <div class="muji-card">
                <h3 class="text-sm font-bold mb-4">支出比例圖</h3>
                <canvas id="statChart" class="mb-6"></canvas>
                <div id="compare-logic" class="text-[11px] text-slate-500 space-y-2 pt-4 border-t border-slate-50"></div>
            </div>
            <div class="muji-card bg-blue-50/30">
                <h3 class="text-sm font-bold mb-4 text-slate-700">職涯投資回報 (ROI)</h3>
                <div id="roi-container" class="space-y-3"></div>
                <p class="text-[9px] text-gray-400 mt-4 italic">以 5.5 萬學費及月薪成長 2.3 萬計算</p>
            </div>
        </div>
    </main>

    <div id="record-modal" class="modal" onclick="if(event.target==this) closeModal()">
        <div class="bg-white w-full rounded-t-3xl p-6 shadow-2xl transition-all">
            <div class="flex justify-between items-center mb-4">
                <h3 id="modal-date" class="font-bold text-slate-700"></h3>
                <button onclick="closeModal()" class="text-slate-300"><i class="fas fa-times"></i></button>
            </div>
            
            <div id="modal-records-list" class="space-y-2 mb-6 max-h-40 overflow-y-auto"></div>
            
            <div class="border-t pt-4 space-y-4">
                <div class="flex flex-wrap gap-2" id="category-list"></div>
                <div class="flex gap-2">
                    <input type="text" id="new-cat" placeholder="新增類別" class="w-1/4 text-[10px] border-b outline-none">
                    <input type="text" id="temp-note" placeholder="項目備註" class="flex-1 text-sm border-b outline-none">
                    <input type="number" id="temp-amt" placeholder="金額" class="w-20 text-sm text-right border-b outline-none">
                </div>
                <button onclick="saveEntry()" class="w-full bg-slate-600 text-white py-3 rounded-xl text-sm font-bold shadow-lg active:scale-95 transition-transform">確認新增</button>
            </div>
        </div>
    </div>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/90 backdrop-blur-md border-t flex justify-around p-4 z-50">
        <button onclick="switchTab('assets')" id="nav-assets" class="nav-btn active flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" id="nav-calendar" class="nav-btn flex flex-col items-center"><i class="fas fa-edit text-lg mb-1"></i><span>記帳</span></button>
        <button onclick="switchTab('stats')" id="nav-stats" class="nav-btn flex flex-col items-center"><i class="fas fa-chart-pie text-lg mb-1"></i><span>分析</span></button>
    </nav>

    <script>
        let curY = 2026, curM = 0;
        let data = JSON.parse(localStorage.getItem('finance_v5')) || { assets: {save:0, stock:0}, records: {} };
        let categories = ["食物費", "交通費", "健身費", "生活用品"];
        let activeCat = "食物費";
        let activeDateKey = "";
        let chartInstance = null;

        function switchTab(t) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-'+t).classList.add('active');
            document.getElementById('nav-'+t).classList.add('active');
            if(t === 'stats') renderStats();
            if(t === 'assets') updateAssetsUI();
        }

        function changeMonth(s) {
            curM += s;
            if(curM > 11) { curM = 0; curY++; }
            if(curM < 0) { curM = 11; curY--; }
            refresh();
        }

        function refresh() {
            document.getElementById('month-label').innerText = `${curY} / ${String(curM+1).padStart(2,'0')}`;
            const grid = document.getElementById('calendar-grid'); grid.innerHTML = '';
            const first = new Date(curY, curM, 1).getDay();
            const days = new Date(curY, curM+1, 0).getDate();

            for(let i=0; i<first; i++) grid.innerHTML += `<div class="bg-slate-50/30"></div>`;
            for(let d=1; d<=days; d++) {
                const key = `${curY}-${curM}-${d}`;
                const total = (data.records[key] || []).reduce((s,r)=>s+r.amt, 0);
                grid.innerHTML += `
                    <div class="calendar-day p-1" onclick="openModal('${key}')">
                        <span class="text-[10px] opacity-40">${d}</span>
                        <div class="day-total text-center">${total>0?'$'+total:''}</div>
                    </div>`;
            }
            updateSummary();
        }

        function openModal(key) {
            activeDateKey = key;
            document.getElementById('modal-date').innerText = key.replace(/-/g, ' / ');
            renderModalRecords();
            renderCategoryChips();
            document.getElementById('record-modal').classList.add('active');
        }

        function renderCategoryChips() {
            const list = document.getElementById('category-list');
            list.innerHTML = categories.map(c => `<span class="cat-chip ${activeCat===c?'active':''}" onclick="setCat('${c}')">${c}</span>`).join('');
        }
        function setCat(c) { activeCat = c; renderCategoryChips(); }

        function saveEntry() {
            const newCat = document.getElementById('new-cat').value;
            const finalCat = newCat || activeCat;
            const amt = Number(document.getElementById('temp-amt').value);
            const note = document.getElementById('temp-note').value;
            if(!amt) return;
            if(newCat && !categories.includes(newCat)) categories.push(newCat);
            if(!data.records[activeDateKey]) data.records[activeDateKey] = [];
            data.records[activeDateKey].push({cat: finalCat, note, amt});
            saveToStorage();
            document.getElementById('temp-amt').value = ''; 
            document.getElementById('temp-note').value = '';
            document.getElementById('new-cat').value = '';
            renderModalRecords(); refresh();
        }

        function renderModalRecords() {
            const list = document.getElementById('modal-records-list');
            const items = data.records[activeDateKey] || [];
            list.innerHTML = items.length ? items.map((r, i) => `
                <div class="flex justify-between items-center bg-slate-50 p-3 rounded-xl animate-fade-in">
                    <div class="flex flex-col">
                        <span class="text-[11px] font-bold text-slate-600">[${r.cat}]</span>
                        <span class="text-[10px] text-slate-400">${r.note || '無備註'}</span>
                    </div>
                    <div class="flex items-center gap-3">
                        <span class="text-sm font-mono">$${r.amt}</span>
                        <button onclick="deleteEntry(${i})" class="text-slate-300 hover:text-red-400"><i class="fas fa-trash-alt"></i></button>
                    </div>
                </div>`).join('') : '<p class="text-center text-slate-300 py-4 text-xs">本日無紀錄</p>';
        }

        function deleteEntry(i) {
            data.records[activeDateKey].splice(i, 1);
            if(!data.records[activeDateKey].length) delete data.records[activeDateKey];
            saveToStorage(); renderModalRecords(); refresh();
        }

        function updateSummary() {
            let mTotal = 0;
            for(let k in data.records) if(k.startsWith(`${curY}-${curM}-`)) data.records[k].forEach(r=>mTotal+=r.amt);
            document.getElementById('month-total-text').innerText = `$ ${mTotal.toLocaleString()}`;
        }

        function saveAssets() {
            data.assets.save = Number(document.getElementById('in-save').value);
            data.assets.stock = Number(document.getElementById('in-stock').value);
            saveToStorage(); updateAssetsUI();
        }

        function updateAssetsUI() {
            const total = 180000 + (data.assets.save||0) + (data.assets.stock||0);
            document.getElementById('total-net').innerText = `$ ${total.toLocaleString()}`;
            document.getElementById('safety-months').innerText = (180000 / 16500).toFixed(1);
        }

        function renderStats() {
            const catsData = {};
            for(let k in data.records) if(k.startsWith(`${curY}-${curM}-`)) data.records[k].forEach(r=>catsData[r.cat]=(catsData[r.cat]||0)+r.amt);
            
            const ctx = document.getElementById('statChart').getContext('2d');
            if(chartInstance) chartInstance.destroy();
            chartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: { labels: Object.keys(catsData), datasets: [{ data: Object.values(catsData), backgroundColor: ['#718899','#A2B5BB','#CBD5E0','#4A5568','#E2E8F0'] }] },
                options: { cutout: '75%', plugins: { legend: { position: 'bottom', labels: { boxWidth: 12, font: { size: 10 } } } } }
            });

            // 交叉分析
            const lastM = curM === 0 ? 11 : curM - 1;
            const lastY = curM === 0 ? curY - 1 : curY;
            let lastTotal = 0;
            for(let k in data.records) if(k.startsWith(`${lastY}-${lastM}-`)) data.records[k].forEach(r=>lastTotal+=r.amt);
            let thisTotal = 0;
            for(let k in catsData) thisTotal += catsData[k];
            
            const diff = thisTotal - lastTotal;
            document.getElementById('compare-logic').innerHTML = `
                <div class="flex justify-between"><span>上月支出</span><span>$${lastTotal.toLocaleString()}</span></div>
                <div class="flex justify-between font-bold"><span>本月支出</span><span>$${thisTotal.toLocaleString()}</span></div>
                <p class="mt-2 text-blue-500">${diff <= 0 ? '✨ 表現優異！本月支出較上月下降。' : '💡 本月開銷增加，建議月底檢視大額項目。'}</p>
            `;
            renderROIMatrix();
        }

        function renderROIMatrix() {
            const gain = (55000-32000)*12;
            document.getElementById('roi-container').innerHTML = [1,3,5].map(y => `
                <div class="flex justify-between text-xs py-2 border-b border-slate-100">
                    <span class="text-slate-500">考上第 ${y} 年累積收益</span>
                    <span class="font-bold text-slate-700">$${(gain*y).toLocaleString()} (ROI: ${((gain*y/55000)*100).toFixed(0)}%)</span>
                </div>
            `).join('');
        }

        function importFixed() {
            const amt = prompt("請輸入固定支出預估值（房租、雜費等）", "15500");
            if(!amt) return;
            const key = `${curY}-${curM}-1`;
            if(!data.records[key]) data.records[key] = [];
            data.records[key].push({cat: "固定支出", note: "預估固定開銷", amt: Number(amt)});
            saveToStorage(); refresh();
        }

        function saveToStorage() { localStorage.setItem('finance_v5', JSON.stringify(data)); }
        function closeModal() { document.getElementById('record-modal').classList.remove('active'); }

        window.onload = () => { 
            document.getElementById('in-save').value = data.assets.save;
            document.getElementById('in-stock').value = data.assets.stock;
            refresh(); updateAssetsUI();
        };
    </script>
</body>
</html>
