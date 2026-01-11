# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>莫蘭迪財務戰略 v7.0</title>
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
        <span class="text-[10px] text-slate-400">比例投資模式</span>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card bg-gradient-to-tr from-slate-700 to-slate-500 text-white border-none shadow-lg">
                <p class="text-[10px] opacity-70 mb-1">TOTAL ASSETS</p>
                <h2 id="total-net" class="text-3xl font-light tracking-tight">$ 0</h2>
                <div class="mt-4 pt-4 border-t border-white/10 text-[10px]">
                    🛡️ 備用金 18 萬：可支撐約 <span id="safety-months">--</span> 個月基本開銷
                </div>
            </div>
            <div class="muji-card">
                <h3 class="text-sm font-bold mb-4">資產紀錄</h3>
                <div class="space-y-4 text-sm">
                    <div class="flex justify-between items-center"><span>🛡️ 緊急備用金</span><span class="font-bold text-blue-400">$ 180,000</span></div>
                    <div class="flex justify-between items-center"><span>現金存款</span><input type="number" id="in-save" value="0" oninput="saveAssets()" class="text-right border-b w-1/3 outline-none focus:border-blue-300"></div>
                    <div class="flex justify-between items-center"><span>投資庫存市值</span><input type="number" id="in-stock" value="0" oninput="saveAssets()" class="text-right border-b w-1/3 outline-none focus:border-blue-300"></div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card p-0 overflow-hidden mb-4 border border-slate-50 shadow-sm">
                <div class="grid grid-cols-7 text-[9px] text-center py-2 bg-slate-50 text-slate-400">
                    <div>SUN</div><div>MON</div><div>TUE</div><div>WED</div><div>THU</div><div>FRI</div><div>SAT</div>
                </div>
                <div id="calendar-grid" class="grid grid-cols-7"></div>
            </div>
            <div class="muji-card text-center py-4">
                <p class="text-[10px] text-slate-400">本月總支出</p>
                <h3 id="month-total-display" class="text-2xl font-light text-slate-700">$ 0</h3>
            </div>
        </div>

        <div id="tab-strategy" class="tab-content">
            <div class="muji-card border-2 border-blue-50">
                <h3 class="text-sm font-bold mb-6 flex items-center gap-2">📈 0050 & 00878 投資模擬</h3>
                
                <div class="space-y-6">
                    <div>
                        <div class="flex justify-between text-[11px] mb-2">
                            <span class="text-slate-500">每月總投入資金</span>
                            <span class="font-bold text-blue-600" id="val-total-invest">$ 6,000</span>
                        </div>
                        <input type="range" min="3000" max="60000" step="1500" value="6000" class="w-full" id="range-invest" oninput="runTargetSim()">
                        <div class="flex justify-between text-[9px] text-slate-400 mt-2">
                            <span>0050 (2/3): <b id="val-0050-part">$4,000</b></span>
                            <span>00878 (1/3): <b id="val-00878-part">$2,000</b></span>
                        </div>
                    </div>

                    <div>
                        <div class="flex justify-between text-[11px] mb-2">
                            <span class="text-slate-500">模擬投資時間</span>
                            <span class="font-bold text-slate-700" id="val-years">10 年</span>
                        </div>
                        <input type="range" min="1" max="30" step="1" value="10" class="w-full" id="range-years" oninput="runTargetSim()">
                    </div>

                    <div class="p-5 bg-slate-50 rounded-2xl space-y-4">
                        <div class="flex justify-between text-xs">
                            <span class="text-slate-500">累計總投入本金</span>
                            <span id="res-principal" class="font-medium">--</span>
                        </div>
                        <div class="flex justify-between items-end">
                            <span class="text-slate-500 text-xs">預期本利和 (含息)</span>
                            <div class="text-right">
                                <span id="res-total-wealth" class="text-xl font-bold text-blue-600">--</span>
                                <p class="text-[9px] text-emerald-500 font-bold" id="res-profit-text"></p>
                            </div>
                        </div>
                        <div class="pt-3 border-t border-slate-200">
                            <div class="flex justify-between text-[10px] text-slate-400">
                                <span>每年可領息 (預估 5%)</span>
                                <span id="res-annual-dividend" class="font-bold text-slate-600">--</span>
                            </div>
                        </div>
                    </div>
                </div>
                <p class="text-[9px] text-slate-300 mt-4 leading-relaxed">※ 模擬邏輯：0050 預期年化 8% / 00878 預期年化 6%，每月複利計算。本金包含您第一頁輸入的當前投資市值。</p>
            </div>

            <div class="muji-card bg-slate-800 text-white">
                <h4 class="text-xs font-bold mb-2">💡 財務導航建議</h4>
                <p id="strategy-guide" class="text-[10px] opacity-80 leading-relaxed"></p>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/90 backdrop-blur-md border-t flex justify-around p-4 z-50">
        <button onclick="switchTab('assets')" id="nav-assets" class="nav-btn active flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" id="nav-calendar" class="nav-btn flex flex-col items-center"><i class="fas fa-edit text-lg mb-1"></i><span>記帳</span></button>
        <button onclick="switchTab('strategy')" id="nav-strategy" class="nav-btn flex flex-col items-center"><i class="fas fa-chess-knight text-lg mb-1"></i><span>戰略</span></button>
    </nav>

    <div id="record-modal" class="modal" onclick="if(event.target==this) closeModal()">
        <div class="bg-white w-full rounded-t-3xl p-6 shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <h3 id="modal-date" class="font-bold text-slate-700"></h3>
                <button onclick="closeModal()" class="text-slate-300"><i class="fas fa-times"></i></button>
            </div>
            <div id="modal-records-list" class="space-y-2 mb-6 max-h-40 overflow-y-auto"></div>
            <div class="border-t pt-4 space-y-4">
                <div class="flex flex-wrap gap-2" id="cat-list"></div>
                <div class="flex gap-2">
                    <input type="text" id="new-cat" placeholder="新類別" class="w-1/4 text-[10px] border-b outline-none">
                    <input type="text" id="temp-note" placeholder="備註" class="flex-1 text-sm border-b outline-none">
                    <input type="number" id="temp-amt" placeholder="金額" class="w-20 text-sm text-right border-b outline-none">
                </div>
                <button onclick="saveEntry()" class="w-full bg-slate-600 text-white py-3 rounded-xl text-sm font-bold shadow-lg">+ 儲存項目</button>
            </div>
        </div>
    </div>

    <script>
        let curY = 2026, curM = 0;
        let data = JSON.parse(localStorage.getItem('finance_v7')) || { assets: {save:0, stock:0}, records: {} };
        let categories = ["食物費", "交通費", "健身費", "生活用品"];
        let activeCat = "食物費";
        let activeDateKey = "";

        function switchTab(t) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-'+t).classList.add('active');
            document.getElementById('nav-'+t).classList.add('active');
            if(t === 'strategy') runTargetSim();
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
            const cat = document.getElementById('new-cat').value || activeCat;
            if(!data.records[activeDateKey]) data.records[activeDateKey] = [];
            data.records[activeDateKey].push({cat, note: document.getElementById('temp-note').value, amt});
            localStorage.setItem('finance_v7', JSON.stringify(data));
            document.getElementById('temp-amt').value = ''; document.getElementById('temp-note').value = '';
            renderModalRecords(); refresh();
        }

        function renderModalRecords() {
            const items = data.records[activeDateKey] || [];
            document.getElementById('modal-records-list').innerHTML = items.map((r, i) => `
                <div class="flex justify-between items-center bg-slate-50 p-2 rounded-xl text-xs mb-1">
                    <span>[${r.cat}] ${r.note}</span>
                    <div class="flex items-center gap-2 font-bold">$${r.amt} <button onclick="deleteItem(${i})" class="text-slate-300">✕</button></div>
                </div>`).join('') || '<p class="text-center text-slate-300 py-4 text-xs">尚無紀錄</p>';
        }

        function deleteItem(i) {
            data.records[activeDateKey].splice(i, 1);
            if(!data.records[activeDateKey].length) delete data.records[activeDateKey];
            localStorage.setItem('finance_v7', JSON.stringify(data)); renderModalRecords(); refresh();
        }

        function saveAssets() {
            data.assets.save = Number(document.getElementById('in-save').value);
            data.assets.stock = Number(document.getElementById('in-stock').value);
            localStorage.setItem('finance_v7', JSON.stringify(data)); updateAssetsUI();
        }

        function updateAssetsUI() {
            const total = 180000 + (data.assets.save||0) + (data.assets.stock||0);
            document.getElementById('total-net').innerText = `$ ${total.toLocaleString()}`;
            document.getElementById('safety-months').innerText = (180000 / 16500).toFixed(1);
        }

        // --- 定額比例投資模擬邏輯 ---
        function runTargetSim() {
            const totalInvest = parseInt(document.getElementById('range-invest').value);
            const years = parseInt(document.getElementById('range-years').value);
            const currentStock = data.assets.stock || 0;

            // 分配金額 (2:1)
            const amt0050 = Math.floor(totalInvest * (2/3));
            const amt00878 = totalInvest - amt0050;

            document.getElementById('val-total-invest').innerText = `$ ${totalInvest.toLocaleString()}`;
            document.getElementById('val-0050-part').innerText = `$ ${amt0050.toLocaleString()}`;
            document.getElementById('val-00878-part').innerText = `$ ${amt00878.toLocaleString()}`;
            document.getElementById('val-years').innerText = `${years} 年`;

            // 複利計算 (0050: 8% / 00878: 6%)
            const calc = (start, monthly, rate, yrs) => {
                let total = start;
                let r = rate / 12;
                for(let i=0; i < yrs * 12; i++) { total = total * (1 + r) + monthly; }
                return total;
            };

            // 假設現有庫存也按比例分配計算，或統一放入 0050 計算
            const final0050 = calc(currentStock * (2/3), amt0050, 0.08, years);
            const final00878 = calc(currentStock * (1/3), amt00878, 0.06, years);
            const totalWealth = final0050 + final00878;
            const totalPrincipal = currentStock + (totalInvest * 12 * years);

            document.getElementById('res-principal').innerText = `$ ${Math.round(totalPrincipal).toLocaleString()}`;
            document.getElementById('res-total-wealth').innerText = `$ ${Math.round(totalWealth).toLocaleString()}`;
            document.getElementById('res-profit-text').innerText = `獲利 +$ ${Math.round(totalWealth - totalPrincipal).toLocaleString()}`;
            document.getElementById('res-annual-dividend').innerText = `$ ${Math.round(totalWealth * 0.05).toLocaleString()}`;

            // 策略導航建議
            let guide = `若維持每月投資 $${totalInvest.toLocaleString()}，${years} 年後資產將成長至 $${Math.round(totalWealth).toLocaleString()}。`;
            if (totalWealth > 800000) guide += ` 恭喜！這已遠超您的 80 萬存錢目標。`;
            document.getElementById('strategy-guide').innerText = guide;
        }

        function closeModal() { document.getElementById('record-modal').classList.remove('active'); }

        window.onload = () => {
            document.getElementById('in-save').value = data.assets.save;
            document.getElementById('in-stock').value = data.assets.stock;
            refresh(); updateAssetsUI();
        };
    </script>
</body>
</html>
