# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>莫蘭迪財務管家</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { 
            --m-blue: #718899; 
            --m-light-blue: #A2B5BB;
            --m-bg: #F4F5F7; 
            --m-text: #4A5568; 
            --m-gray: #CBD5E0; 
        }
        body { background-color: var(--m-bg); color: var(--m-text); font-family: "PingFang TC", sans-serif; overflow-x: hidden; }
        .muji-card { background: white; border-radius: 16px; margin-bottom: 1.2rem; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05); }
        .nav-btn { color: var(--m-gray); transition: 0.3s; font-size: 10px; }
        .nav-btn.active { color: var(--m-blue); }
        
        /* 日曆樣式 */
        .calendar-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 1px; background: #EDF2F7; }
        .calendar-day { background: white; min-height: 75px; padding: 6px; font-size: 11px; cursor: pointer; display: flex; flex-direction: column; }
        .calendar-day:active { background: #f0f4f8; }
        .today-circle { border: 1.5px solid var(--m-blue); border-radius: 50%; width: 20px; height: 20px; display: flex; items-center; justify-content: center; }
        .day-total { color: var(--m-blue); font-weight: bold; font-size: 10px; margin-top: auto; }

        /* 分頁切換控制 */
        .tab-content { display: none; min-height: 80vh; }
        .tab-content.active { display: block; animation: fadeIn 0.4s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* Modal */
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.4); z-index: 100; align-items: flex-end; }
        .modal.active { display: flex; }
        .modal-content { background: white; width: 100%; border-radius: 24px 24px 0 0; padding: 24px; animation: slideUp 0.3s ease-out; }
        @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
        
        input { border-bottom: 1px solid var(--m-gray) !important; background: transparent !important; }
        input:focus { border-bottom: 2px solid var(--m-blue) !important; outline: none; }
        .m-btn { background: var(--m-blue); color: white; padding: 12px; border-radius: 12px; font-size: 14px; width: 100%; }
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
                <p class="text-[11px] text-slate-400 uppercase tracking-widest mb-2">Total Balance</p>
                <h2 id="total-net-display" class="text-4xl font-light text-slate-800 tracking-tight">$ 180,000</h2>
            </div>
            
            <div class="muji-card p-6">
                <div class="flex items-center gap-3 mb-6">
                    <div class="w-1 h-5 bg-blue-300"></div>
                    <h3 class="font-bold text-sm">資產明細</h3>
                </div>
                <div class="space-y-6">
                    <div class="flex justify-between items-center">
                        <span class="text-sm">🛡️ 緊急備用金</span>
                        <span class="font-mono font-bold text-blue-500">$ 180,000</span>
                    </div>
                    <div class="flex justify-between items-center">
                        <span class="text-sm">銀行儲蓄</span>
                        <input type="number" id="other-save" value="0" oninput="calcNet()" class="text-right w-1/3 p-1">
                    </div>
                    <div class="flex justify-between items-center">
                        <span class="text-sm">投資市值</span>
                        <input type="number" id="in-stock" value="0" oninput="calcNet()" class="text-right w-1/3 p-1">
                    </div>
                </div>
            </div>
        </div>

        <div id="tab-calendar" class="tab-content">
            <div class="muji-card overflow-hidden">
                <div class="grid grid-cols-7 text-center text-[10px] py-3 bg-slate-50 text-slate-400 border-b">
                    <div>SUN</div><div>MON</div><div>TUE</div><div>WED</div><div>THU</div><div>FRI</div><div>SAT</div>
                </div>
                <div id="calendar-body" class="calendar-grid"></div>
            </div>
            <div class="mt-4 space-y-2">
                <p class="text-[11px] text-slate-400"><i class="fas fa-info-circle mr-1"></i> 點擊日期可記錄多筆支出</p>
            </div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-6 flex items-center gap-2"><i class="fas fa-fingerprint text-blue-300"></i> 本月數據分析</h3>
                <div class="space-y-4">
                    <div class="flex justify-between text-sm"><span>預設月薪</span><input type="number" id="salary" value="32000" class="text-right font-bold w-1/2" oninput="updateStrategy()"></div>
                    <div class="p-4 bg-slate-50 rounded-2xl flex justify-between items-center">
                        <span class="text-xs text-slate-500">已記帳總額</span>
                        <span id="analysis-spent" class="font-bold text-slate-700">$ 0</span>
                    </div>
                    
                    <div id="strategy-box" class="p-5 rounded-2xl border transition-all duration-500">
                        <div class="flex justify-between mb-2">
                            <span id="strategy-label" class="text-xs font-bold">--</span>
                            <span id="strategy-status" class="text-[10px] bg-white/50 px-2 py-1 rounded">分析中</span>
                        </div>
                        <p id="strategy-advice" class="text-[11px] leading-relaxed text-slate-500"></p>
                    </div>
                </div>
            </div>

            <div class="muji-card p-6 border-2 border-dashed border-slate-200">
                <p class="text-[11px] text-slate-400 mb-2">2年 80 萬進度</p>
                <div id="mini-progress"></div>
            </div>
        </div>
    </main>

    <div id="record-modal" class="modal" onclick="if(event.target==this) closeModal()">
        <div class="modal-content">
            <div class="flex justify-between items-center mb-6">
                <h3 id="modal-date" class="font-bold text-slate-700">2026-01-01</h3>
                <button onclick="closeModal()" class="text-slate-300"><i class="fas fa-times"></i></button>
            </div>
            
            <div id="current-records" class="mb-6 space-y-2 max-height-[200px] overflow-y-auto">
                </div>

            <div class="space-y-4 border-t pt-4">
                <div class="flex gap-2">
                    <input type="text" id="temp-note" placeholder="項目" class="w-3/5 text-sm">
                    <input type="number" id="temp-amount" placeholder="金額" class="w-2/5 text-sm text-right">
                </div>
                <button onclick="saveOneRecord()" class="m-btn">+ 增加紀錄</button>
            </div>
        </div>
    </div>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/80 backdrop-blur-md border-t flex justify-around p-4 shadow-sm z-50">
        <button onclick="switchTab('assets')" id="nav-assets" class="nav-btn active flex flex-col items-center"><i class="fas fa-wallet text-lg mb-1"></i><span>資產</span></button>
        <button onclick="switchTab('calendar')" id="nav-calendar" class="nav-btn flex flex-col items-center"><i class="fas fa-calendar-alt text-lg mb-1"></i><span>記帳</span></button>
        <button onclick="switchTab('analysis')" id="nav-analysis" class="nav-btn flex flex-col items-center"><i class="fas fa-leaf text-lg mb-1"></i><span>戰略</span></button>
    </nav>

    <script>
        let db = JSON.parse(localStorage.getItem('my_finance_db')) || {}; 
        // 結構: { '2026-01-11': [{note:'餐', amt:100}, {note:'飲', amt:50}] }
        let currentOpenDate = '';

        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('tab-' + name).classList.add('active');
            document.getElementById('nav-' + name).classList.add('active');
            const titles = {'assets':'資產狀況', 'calendar':'每日記帳', 'analysis':'財務戰略'};
            document.getElementById('header-title').innerText = titles[name];
            if(name === 'analysis') updateStrategy();
        }

        function initCalendar() {
            const body = document.getElementById('calendar-body');
            body.innerHTML = '';
            const firstDay = new Date(2026, 0, 1).getDay();
            for(let i=0; i<firstDay; i++) body.innerHTML += `<div class="calendar-day bg-slate-50/50"></div>`;
            
            for(let d=1; d<=31; d++) {
                const dateStr = `2026-01-${String(d).padStart(2,'0')}`;
                const dayRecords = db[dateStr] || [];
                const dayTotal = dayRecords.reduce((sum, r) => sum + r.amt, 0);
                const isToday = d === 11;
                
                body.innerHTML += `
                    <div class="calendar-day" onclick="openModal('${dateStr}')">
                        <span class="${isToday?'today-circle':''}">${d}</span>
                        <span class="day-total">${dayTotal > 0 ? '$'+dayTotal : ''}</span>
                    </div>`;
            }
        }

        function openModal(date) {
            currentOpenDate = date;
            document.getElementById('modal-date').innerText = date;
            renderCurrentRecords();
            document.getElementById('record-modal').classList.add('active');
        }

        function renderCurrentRecords() {
            const list = document.getElementById('current-records');
            const dayRecords = db[currentOpenDate] || [];
            list.innerHTML = dayRecords.length ? '' : '<p class="text-xs text-slate-300 text-center py-4">本日無紀錄</p>';
            dayRecords.forEach((r, idx) => {
                list.innerHTML += `
                    <div class="flex justify-between items-center bg-slate-50 p-3 rounded-xl">
                        <span class="text-xs text-slate-500">${r.note}</span>
                        <div class="flex items-center gap-3">
                            <span class="text-sm font-bold text-slate-700">$${r.amt}</span>
                            <button onclick="deleteRecord(${idx})" class="text-slate-300 text-[10px]"><i class="fas fa-trash"></i></button>
                        </div>
                    </div>`;
            });
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
            renderCurrentRecords();
            initCalendar();
        }

        function deleteRecord(idx) {
            db[currentOpenDate].splice(idx, 1);
            localStorage.setItem('my_finance_db', JSON.stringify(db));
            renderCurrentRecords();
            initCalendar();
        }

        function closeModal() { document.getElementById('record-modal').classList.remove('active'); }

        function calcNet() {
            const other = Number(document.getElementById('other-save').value) || 0;
            const stock = Number(document.getElementById('in-stock').value) || 0;
            document.getElementById('total-net-display').innerText = '$ ' + (180000 + other + stock).toLocaleString();
        }

        function updateStrategy() {
            const salary = Number(document.getElementById('salary').value) || 0;
            let totalSpent = 0;
            Object.values(db).forEach(day => day.forEach(r => totalSpent += r.amt));
            
            document.getElementById('analysis-spent').innerText = `$ ${totalSpent.toLocaleString()}`;
            const ratio = totalSpent / salary;
            const box = document.getElementById('strategy-box');
            const label = document.getElementById('strategy-label');
            const advice = document.getElementById('strategy-advice');

            if(ratio < 0.4) {
                box.className = "p-5 rounded-2xl border border-emerald-100 bg-emerald-50";
                label.innerText = "進取：資金高速成長中";
                advice.innerText = "目前消費極低，你的存錢效率非常驚人。適合增加定期定額的額度。";
            } else if(ratio < 0.6) {
                box.className = "p-5 rounded-2xl border border-blue-100 bg-blue-50";
                label.innerText = "平衡：莫蘭迪式的優雅";
                advice.innerText = "你在生活品質與未來儲蓄間取得了平衡，這是在通膨時代最長久的理財方式。";
            } else {
                box.className = "p-5 rounded-2xl border border-slate-100 bg-white";
                label.innerText = "保守：應對高物價挑戰";
                advice.innerText = "目前的開銷稍大，但考慮到你有 18 萬備用金，建議先專注於考試加薪，不必過度苛待自己。";
            }
        }

        window.onload = () => { initCalendar(); calcNet(); };
    </script>
</body>
</html>
