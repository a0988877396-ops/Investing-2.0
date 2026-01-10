# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MUJI 財務戰略管家</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --muji-bg: #F7F3F0; --muji-red: #7F2D2D; --muji-text: #444444; --muji-gray: #BCB8B1; }
        body { background-color: var(--muji-bg); color: var(--muji-text); font-family: "PingFang TC", sans-serif; }
        .muji-card { background: white; border: 1px solid #E6E2DF; border-radius: 12px; margin-bottom: 1rem; }
        .nav-btn { color: var(--muji-gray); transition: 0.3s; font-size: 10px; }
        .nav-btn.active { color: var(--muji-red); }
        .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 2px; background: #E6E2DF; border: 1px solid #E6E2DF; }
        .calendar-day { background: white; min-height: 60px; padding: 4px; font-size: 10px; position: relative; cursor: pointer; }
        .calendar-day:active { background: #F7F3F0; }
        .today { background: #FFF9F9; border: 1px solid var(--muji-red); }
        .expense-tag { color: var(--muji-red); font-weight: bold; margin-top: 2px; display: block; }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 100; align-items: center; justify-content: center; }
        .modal.active { display: flex; }
    </style>
</head>
<body class="pb-24">

    <header class="p-4 bg-white border-b sticky top-0 z-50 flex justify-between items-center">
        <h1 id="header-title" class="text-sm font-medium tracking-widest">基礎資產</h1>
        <div id="month-display" class="text-xs font-bold text-gray-400">2026 / 01</div>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-6 border-l-8 border-blue-200">
                <p class="text-[10px] text-gray-400">🛡️ 緊急備用金 (已鎖定)</p>
                <h2 class="text-2xl font-bold text-blue-800">$ 180,000</h2>
            </div>
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4">流動資產狀況</h3>
                <div class="space-y-4 text-sm">
                    <div class="flex justify-between"><span>目前其餘存款</span><input type="number" id="other-save" class="text-right w-1/2 border-b border-gray-200" value="0"></div>
                    <div class="flex justify-between"><span>投資總市值</span><input type="number" id="in-stock" class="text-right w-1/2 border-b border-gray-200" value="0"></div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card overflow-hidden">
                <div class="grid grid-cols-7 text-center text-[10px] py-2 bg-stone-100 text-gray-500">
                    <div>日</div><div>一</div><div>二</div><div>三</div><div>四</div><div>五</div><div>六</div>
                </div>
                <div id="calendar-body" class="calendar-grid">
                    </div>
            </div>
            <div class="p-2 text-[11px] text-gray-400">＊點擊日期記錄當日支出</div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-red-800 pl-3">本月收支數據分析</h3>
                <div class="space-y-5">
                    <div class="flex justify-between text-sm">
                        <span>目前月薪收入</span>
                        <input type="number" id="salary" value="32000" class="text-right font-bold text-red-800" oninput="updateStrategy()">
                    </div>
                    <div class="p-4 bg-stone-50 rounded-lg space-y-3">
                        <div class="flex justify-between text-xs text-gray-500">
                            <span>本月已記帳支出</span>
                            <span id="total-spent-display" class="font-bold text-gray-800">$ 0</span>
                        </div>
                        <div class="flex justify-between text-xs text-gray-500 border-t pt-2">
                            <span>預估結餘</span>
                            <span id="surplus-display" class="font-bold text-blue-600">$ 32,000</span>
                        </div>
                    </div>
                    <div id="strategy-card" class="p-4 rounded-xl border-2 transition-all duration-500">
                        <div class="flex justify-between items-center mb-2">
                            <span id="strategy-label" class="text-xs font-bold">分析中...</span>
                            <span id="strategy-status" class="text-[10px] px-2 py-1 rounded bg-white">等待數據</span>
                        </div>
                        <p id="strategy-advice" class="text-[11px] leading-relaxed"></p>
                    </div>
                </div>
            </div>
            
            <div class="muji-card p-6 bg-red-50/30">
                <h3 class="text-xs font-bold mb-2">2年 80 萬計畫進度</h3>
                <div id="plan-progress-mini"></div>
            </div>
        </div>
    </main>

    <div id="record-modal" class="modal">
        <div class="bg-white p-6 rounded-2xl w-80 shadow-2xl">
            <h3 id="modal-date" class="text-sm font-bold mb-4">2026-01-01</h3>
            <div class="space-y-4">
                <input type="number" id="temp-amount" placeholder="輸入金額" class="w-full text-2xl py-2 border-b-2 border-red-800 text-center">
                <input type="text" id="temp-note" placeholder="備註 (如:晚餐)" class="w-full text-xs text-center text-gray-400">
                <div class="flex gap-2">
                    <button onclick="closeModal()" class="w-1/2 py-2 text-xs bg-gray-100 rounded">取消</button>
                    <button onclick="saveRecord()" class="w-1/2 py-2 text-xs bg-red-800 text-white rounded">儲存紀錄</button>
                </div>
            </div>
        </div>
    </div>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t flex justify-around p-4 shadow-lg">
        <button onclick="switchTab('assets')" id="nav-assets" class="nav-btn active flex flex-col items-center"><i class="fas fa-shield-alt text-lg"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" id="nav-calendar" class="nav-btn flex flex-col items-center"><i class="fas fa-calendar-day text-lg"></i><span>日曆記帳</span></button>
        <button onclick="switchTab('analysis')" id="nav-analysis" class="nav-btn flex flex-col items-center"><i class="fas fa-chart-line text-lg"></i><span>戰略分析</span></button>
    </nav>

    <script>
        let records = {}; // 格式: {'2026-01-11': 150}
        let selectedDate = '';

        function initCalendar() {
            const body = document.getElementById('calendar-body');
            body.innerHTML = '';
            const now = new Date();
            const year = 2026, month = 0; // 鎖定在 2026/01
            const firstDay = new Date(year, month, 1).getDay();
            const daysInMonth = 31;

            for(let i=0; i<firstDay; i++) body.innerHTML += `<div class="calendar-day bg-stone-50"></div>`;
            
            for(let day=1; day<=daysInMonth; day++) {
                const dateStr = `${year}-${String(month+1).padStart(2,'0')}-${String(day).padStart(2,'0')}`;
                const isToday = day === 11;
                const amt = records[dateStr] ? `$${records[dateStr]}` : '';
                body.innerHTML += `
                    <div class="calendar-day ${isToday?'today':''}" onclick="openModal('${dateStr}')">
                        ${day}
                        <span class="expense-tag">${amt}</span>
                    </div>`;
            }
        }

        function openModal(date) {
            selectedDate = date;
            document.getElementById('modal-date').innerText = date;
            document.getElementById('temp-amount').value = records[date] || '';
            document.getElementById('record-modal').classList.add('active');
        }

        function closeModal() { document.getElementById('record-modal').classList.remove('active'); }

        function saveRecord() {
            const amt = Number(document.getElementById('temp-amount').value);
            if(amt > 0) records[selectedDate] = amt;
            else delete records[selectedDate];
            initCalendar();
            closeModal();
            updateStrategy();
        }

        function updateStrategy() {
            const salary = Number(document.getElementById('salary').value) || 0;
            const totalSpent = Object.values(records).reduce((a,b)=>a+b, 0);
            const surplus = salary - totalSpent;
            
            document.getElementById('total-spent-display').innerText = `$ ${totalSpent.toLocaleString()}`;
            document.getElementById('surplus-display').innerText = `$ ${surplus.toLocaleString()}`;

            const card = document.getElementById('strategy-card');
            const label = document.getElementById('strategy-label');
            const status = document.getElementById('strategy-status');
            const advice = document.getElementById('strategy-advice');

            const spendRatio = totalSpent / salary;

            if (spendRatio <= 0.4) { // 進取：花不到 40%
                card.className = "p-4 rounded-xl border-2 border-emerald-200 bg-emerald-50";
                label.innerText = "戰略等級：進取模式";
                status.innerText = "存款率 60%+";
                advice.innerText = "表現優異！依照此節奏，你的 80 萬計畫將縮短 4 個月達成。目前的物價完全沒對你造成威脅。";
            } else if (spendRatio <= 0.6) { // 平衡：花 40%~60%
                card.className = "p-4 rounded-xl border-2 border-blue-200 bg-blue-50";
                label.innerText = "戰略等級：平衡模式";
                status.innerText = "存款率 40%-50%";
                advice.innerText = "這是在通膨環境下最健康的狀態。在享受生活（乾麵加蛋）與存錢之間取得了完美平衡。";
            } else { // 保守：花超過 60%
                card.className = "p-4 rounded-xl border-2 border-orange-200 bg-orange-50";
                label.innerText = "戰略等級：保守模式";
                status.innerText = "存款率 < 40%";
                advice.innerText = "目前生活成本較高。考慮到你已有 18 萬備用金，不必過度焦慮，但要警惕不必要的社交開銷。";
            }
            
            // 更新迷你進度條
            const totalAssets = 180000 + (Number(document.getElementById('other-save').value)||0) + (Number(document.getElementById('in-stock').value)||0);
            const percent = ((totalAssets / 800000) * 100).toFixed(1);
            document.getElementById('plan-progress-mini').innerHTML = `
                <div class="flex justify-between text-[10px] mb-1"><span>目標 80 萬</span><span>已達成 ${percent}%</span></div>
                <div class="w-full bg-white h-1.5 rounded-full overflow-hidden"><div class="bg-red-800 h-full" style="width:${percent}%"></div></div>
            `;
        }

        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-' + name).classList.add('active');
            document.getElementById('nav-' + name).classList.add('active');
            const titles = {'assets':'基礎資產', 'calendar':'日曆記帳', 'analysis':'戰略分析'};
            document.getElementById('header-title').innerText = titles[name];
        }

        window.onload = () => { initCalendar(); updateStrategy(); };
    </script>
</body>
</html>
