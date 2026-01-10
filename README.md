# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>莫蘭迪財務管家 Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --m-blue: #718899; --m-light-blue: #A2B5BB; --m-bg: #F4F5F7; --m-text: #4A5568; --m-gray: #CBD5E0; }
        body { background-color: var(--m-bg); color: var(--m-text); font-family: "PingFang TC", sans-serif; }
        .muji-card { background: white; border-radius: 16px; margin-bottom: 1rem; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05); }
        .nav-btn { color: var(--m-gray); transition: 0.3s; font-size: 10px; }
        .nav-btn.active { color: var(--m-blue); }
        .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 1px; background: #EDF2F7; }
        .calendar-day { background: white; min-height: 65px; padding: 4px; font-size: 11px; cursor: pointer; display: flex; flex-direction: column; }
        .day-total { color: var(--m-blue); font-weight: bold; font-size: 9px; margin-top: auto; }
        .tab-content { display: none; }
        .tab-content.active { display: block; animation: fadeIn 0.4s ease-out; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 100; align-items: flex-end; }
        .modal.active { display: flex; }
        .modal-content { background: white; width: 100%; border-radius: 24px 24px 0 0; padding: 24px; }
        input { border-bottom: 1px solid var(--m-gray) !important; background: transparent !important; }
        .roi-table td { padding: 8px 4px; border-bottom: 1px solid #f0f0f0; font-size: 11px; }
    </style>
</head>
<body class="pb-24">

    <header class="p-5 bg-white flex justify-between items-center sticky top-0 z-40">
        <h1 id="header-title" class="text-lg font-medium tracking-widest text-slate-700">資產狀況</h1>
        <span class="text-xs text-slate-400 font-mono">2026.01</span>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-8 text-center bg-gradient-to-br from-slate-50 to-white">
                <p class="text-[11px] text-slate-400 uppercase tracking-widest mb-2">Total Net Worth</p>
                <h2 id="total-net-display" class="text-4xl font-light text-slate-800 tracking-tight">$ 180,000</h2>
            </div>
            <div class="muji-card p-6">
                <h3 class="font-bold text-sm mb-6 border-l-4 border-blue-200 pl-3">資產明細</h3>
                <div class="space-y-6 text-sm">
                    <div class="flex justify-between"><span>🛡️ 緊急備用金</span><span class="font-bold text-blue-400">$ 180,000</span></div>
                    <div class="flex justify-between"><span>銀行其餘儲蓄</span><input type="number" id="other-save" value="0" oninput="calcNet()" class="text-right w-1/3"></div>
                    <div class="flex justify-between"><span>投資市值</span><input type="number" id="in-stock" value="0" oninput="calcNet()" class="text-right w-1/3"></div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card overflow-hidden mb-4">
                <div id="calendar-body" class="calendar-grid"></div>
            </div>

            <div class="muji-card p-5">
                <div class="flex justify-between items-end mb-4">
                    <h3 class="text-sm font-bold">本月分析</h3>
                    <div class="text-right"><span class="text-[10px] text-gray-400">設定月薪</span><br>
                        <input type="number" id="salary" value="32000" class="text-right font-bold w-20 text-sm" oninput="updateStrategy()">
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-3 mb-4">
                    <div class="bg-slate-50 p-3 rounded-xl"><p class="text-[10px] text-gray-400">已支出</p><p id="analysis-spent" class="font-bold text-sm">$ 0</p></div>
                    <div class="bg-slate-50 p-3 rounded-xl"><p class="text-[10px] text-gray-400">剩餘結餘</p><p id="surplus-display" class="font-bold text-sm text-blue-500">$ 32,000</p></div>
                </div>
                <div id="strategy-box" class="p-4 rounded-xl border">
                    <p id="strategy-label" class="text-xs font-bold mb-1">--</p>
                    <p id="strategy-advice" class="text-[10px] text-slate-500 leading-relaxed"></p>
                </div>
            </div>
        </div>

        <div id="tab-strategy" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-slate-400 pl-3">2年 80 萬達標戰略</h3>
                <div id="plan-detail" class="text-xs space-y-4">
                    </div>
            </div>

            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-blue-400 pl-3">職涯投資：回報矩陣 (ROI)</h3>
                <table class="w-full roi-table text-left">
                    <thead class="text-[10px] text-gray-400">
                        <tr><th>年度</th><th>累計加薪收益</th><th>投資報酬率</th></tr>
                    </thead>
                    <tbody id="roi-matrix-body">
                        </tbody>
                </table>
                <p class="text-[9px] text-gray-400 mt-4">註：以補習費 $55,000 及月薪成長 $23,000 計算</p>
            </div>
        </div>
    </main>

    <div id="record-modal" class="modal" onclick="if(event.target==this) closeModal()">
        <div class="modal-content">
            <div class="flex justify-between items-center mb-6">
                <h3 id="modal-date" class="font-bold">2026-01-01</h3>
                <button onclick="closeModal()" class="text-slate-300"><i class="fas fa-times"></i></button>
            </div>
            <div id="current-records" class="mb-6 space-y-2 max-h-40 overflow-y-auto"></div>
            <div class="space-y-4 border-t pt-4">
                <div class="flex gap-2">
                    <input type="text" id="temp-note" placeholder="品項" class="w-3/5 text-sm">
                    <input type="number" id="temp-amount" placeholder="金額" class="w-2/5 text-sm text-right">
                </div>
                <button onclick="saveOneRecord()" class="w-full bg-slate-600 text-white py-3 rounded-xl text-sm">+ 增加紀錄</button>
            </div>
        </div>
    </div>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/80 backdrop-blur-md border-t flex justify-around p-4 z-50">
        <button onclick="switchTab('assets')" id="nav-assets" class="nav-btn active flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" id="nav-calendar" class="nav-btn flex flex-col items-center"><i class="fas fa-edit text-lg mb-1"></i><span>記帳</span></button>
        <button onclick="switchTab('strategy')" id="nav-strategy" class="nav-btn flex flex-col items-center"><i class="fas fa-chess-knight text-lg mb-1"></i><span>戰略</span></button>
    </nav>

    <script>
        let db = JSON.parse(localStorage.getItem('my_finance_db')) || {};
        let currentOpenDate = '';

        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-' + name).classList.add('active');
            document.getElementById('nav-' + name).classList.add('active');
            const titles = {'assets':'資產狀況', 'calendar':'記帳與分析', 'strategy':'成長戰略'};
            document.getElementById('header-title').innerText = titles[name];
            if(name === 'strategy') { calculatePlan(); renderROIMatrix(); }
        }

        function initCalendar() {
            const body = document.getElementById('calendar-body');
            body.innerHTML = '';
            const firstDay = new Date(2026, 0, 1).getDay();
            for(let i=0; i<firstDay; i++) body.innerHTML += `<div class="calendar-day bg-slate-50/30"></div>`;
            for(let d=1; d<=31; d++) {
                const dateStr = `2026-01-${String(d).padStart(2,'0')}`;
                const dayTotal = (db[dateStr] || []).reduce((s, r) => s + r.amt, 0);
                body.innerHTML += `<div class="calendar-day" onclick="openModal('${dateStr}')"><span>${d}</span><span class="day-total">${dayTotal>0?'$'+dayTotal:''}</span></div>`;
            }
        }

        function openModal(date) {
            currentOpenDate = date;
            document.getElementById('modal-date').innerText = date;
            renderCurrentRecords();
            document.getElementById('record-modal').classList.add('active');
        }

        function saveOneRecord() {
            const note = document.getElementById('temp-note').value || '未分類';
            const amt = Number(document.getElementById('temp-amount').value);
            if(!amt) return;
            if(!db[currentOpenDate]) db[currentOpenDate] = [];
            db[currentOpenDate].push({note, amt});
            localStorage.setItem('my_finance_db', JSON.stringify(db));
            document.getElementById('temp-note').value = '';
            document.getElementById('temp-amount').value = '';
            renderCurrentRecords(); initCalendar(); updateStrategy();
        }

        function renderCurrentRecords() {
            const list = document.getElementById('current-records');
            list.innerHTML = (db[currentOpenDate] || []).map((r, i) => `
                <div class="flex justify-between bg-slate-50 p-2 rounded text-xs">
                    <span>${r.note}</span><span>$${r.amt}</span>
                </div>
            `).join('');
        }

        function closeModal() { document.getElementById('record-modal').classList.remove('active'); }

        function calcNet() {
            const total = 180000 + (Number(document.getElementById('other-save').value)||0) + (Number(document.getElementById('in-stock').value)||0);
            document.getElementById('total-net-display').innerText = '$ ' + total.toLocaleString();
        }

        function updateStrategy() {
            const salary = Number(document.getElementById('salary').value) || 0;
            let spent = 0; Object.values(db).forEach(day => day.forEach(r => spent += r.amt));
            const surplus = salary - spent;
            document.getElementById('analysis-spent').innerText = `$ ${spent.toLocaleString()}`;
            document.getElementById('surplus-display').innerText = `$ ${surplus.toLocaleString()}`;
            
            const ratio = spent / salary;
            const box = document.getElementById('strategy-box');
            if(ratio < 0.4) { box.className="p-4 rounded-xl border border-emerald-100 bg-emerald-50"; document.getElementById('strategy-label').innerText="進取模式"; }
            else if(ratio < 0.6) { box.className="p-4 rounded-xl border border-blue-100 bg-blue-50"; document.getElementById('strategy-label').innerText="平衡模式"; }
            else { box.className="p-4 rounded-xl border border-slate-100 bg-white"; document.getElementById('strategy-label').innerText="保守模式"; }
            document.getElementById('strategy-advice').innerText = `目前支出佔比 ${(ratio*100).toFixed(0)}%，預計本月可存下 $${surplus.toLocaleString()}。`;
        }

        function calculatePlan() {
            const currentTotal = 180000 + (Number(document.getElementById('other-save').value)||0) + (Number(document.getElementById('in-stock').value)||0);
            const gap = 800000 - currentTotal;
            const monthlyNeed = (gap / 24).toFixed(0);
            
            document.getElementById('plan-detail').innerHTML = `
                <div class="bg-slate-50 p-4 rounded-xl">
                    <p>目前總額：<b>$${currentTotal.toLocaleString()}</b></p>
                    <p>距離目標：<b>$${gap.toLocaleString()}</b></p>
                </div>
                <div class="p-2">
                    <p>💡 <b>2年達成攻略：</b></p>
                    <p class="mt-2">在不考慮加薪下，你每個月需淨存 <b>$${Number(monthlyNeed).toLocaleString()}</b>。</p>
                    <p class="mt-1 text-slate-400 text-[10px]">※ 當你考取 5.5 萬職位後，每月存款能力將大幅跳升，目標達成時間將縮短至約 10 個月。</p>
                </div>
            `;
        }

        function renderROIMatrix() {
            const cost = 55000;
            const monthlyGain = 55000 - 32000;
            const body = document.getElementById('roi-matrix-body');
            body.innerHTML = '';
            for(let year=1; year<=5; year++) {
                const totalGain = monthlyGain * 12 * year;
                const roi = ((totalGain / cost) * 100).toFixed(0);
                body.innerHTML += `<tr><td>第 ${year} 年</td><td>$${totalGain.toLocaleString()}</td><td class="text-blue-500 font-bold">${roi}%</td></tr>`;
            }
        }

        window.onload = () => { initCalendar(); calcNet(); updateStrategy(); };
    </script>
</body>
</html>
