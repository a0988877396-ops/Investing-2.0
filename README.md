# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MUJI 財務計畫 Pro+ (通膨應對版)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --muji-bg: #F7F3F0; --muji-red: #7F2D2D; --muji-text: #444444; }
        body { background-color: var(--muji-bg); color: var(--muji-text); font-family: "PingFang TC", sans-serif; }
        .muji-card { background: white; border: 1px solid #E6E2DF; border-radius: 12px; margin-bottom: 1.5rem; }
        input { border-bottom: 1px solid #BCB8B1 !important; background: transparent !important; }
        input:focus { border-bottom: 1px solid var(--muji-red) !important; outline: none; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .nav-btn.active { color: var(--muji-red); }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 bg-white border-b sticky top-0 z-50 text-center">
        <h1 id="header-title" class="text-base font-medium tracking-widest">個人財務戰略</h1>
    </header>

    <main class="p-5 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-6 border-l-8 border-blue-200">
                <p class="text-[10px] text-gray-400">🛡️ 緊急備用金 (已鎖定)</p>
                <h2 class="text-2xl font-bold text-blue-800">$ 180,000</h2>
                <p class="text-[10px] mt-1 text-blue-600">這筆錢是你的防禦牆，不列入支出分配。</p>
            </div>
            
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-5">其他流動資產</h3>
                <div class="space-y-4 text-sm">
                    <div class="flex justify-between"><span>目前其餘存款</span><input type="number" id="other-save" oninput="liveUpdate()" class="text-right w-1/2" value="0"></div>
                    <div class="flex justify-between"><span>投資總市值</span><input type="number" id="in-stock" oninput="liveUpdate()" class="text-right w-1/2" value="0"></div>
                </div>
            </div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-red-800 pl-3">月收支分配 (應對通膨)</h3>
                <div class="space-y-4">
                    <div class="flex justify-between items-center text-sm font-bold">
                        <span>目前月薪</span>
                        <input type="number" id="salary" value="32000" class="text-right w-1/2" oninput="autoAnalyze()">
                    </div>
                    <div id="expense-container" class="space-y-3">
                        <div class="flex gap-2">
                            <input type="text" placeholder="品項" class="w-2/3 text-xs exp-name" value="固定支出加總">
                            <input type="number" placeholder="金額" class="w-1/3 text-right exp-val text-xs" value="15500" oninput="autoAnalyze()">
                        </div>
                    </div>
                    <div class="p-3 bg-stone-50 rounded space-y-3">
                        <div class="flex justify-between text-xs">
                            <span>存款分配比例</span>
                            <select id="save-ratio" onchange="autoAnalyze()" class="bg-transparent border-b border-gray-400 text-red-800 font-bold">
                                <option value="0.4">保守 (存40%)</option>
                                <option value="0.5">平衡 (存50%)</option>
                                <option value="0.6" selected>進取 (存60%)</option>
                            </select>
                        </div>
                        <div id="allocation-detail" class="text-[11px] leading-relaxed text-gray-500">
                            </div>
                    </div>
                </div>
            </div>

            <div class="muji-card p-6 bg-red-50/50">
                <h3 class="text-sm font-bold mb-2">職涯增值分析</h3>
                <div id="roi-text" class="text-xs text-red-900 leading-relaxed"></div>
            </div>
        </div>

        <div id="tab-plan" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4">2年 80萬 達成率</h3>
                <div id="progress-ui" class="text-center py-4">
                    </div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t flex justify-around p-4">
        <button onclick="switchTab('assets')" class="nav-btn active flex flex-col items-center"><i class="fas fa-shield-alt text-lg"></i><span class="text-[10px] mt-1">資產</span></button>
        <button onclick="switchTab('analysis')" class="nav-btn flex flex-col items-center"><i class="fas fa-calculator text-lg"></i><span class="text-[10px] mt-1">分析</span></button>
        <button onclick="switchTab('plan')" class="nav-btn flex flex-col items-center"><i class="fas fa-tasks text-lg"></i><span class="text-[10px] mt-1">計畫</span></button>
    </nav>

    <script>
        const FIXED_EMERGENCY = 180000;

        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById('tab-' + name).classList.add('active');
            if(name === 'analysis') autoAnalyze();
            if(name === 'plan') calculatePlan();
        }

        function autoAnalyze() {
            const salary = Number(document.getElementById('salary').value) || 0;
            let totalExp = 0;
            document.querySelectorAll('.exp-val').forEach(i => totalExp += Number(i.value) || 0);
            const surplus = salary - totalExp;
            const ratio = Number(document.getElementById('save-ratio').value);
            
            const fixedSave = Math.floor(surplus * ratio);
            const lifeMoney = surplus - fixedSave;

            document.getElementById('allocation-detail').innerHTML = `
                結餘：$${surplus.toLocaleString()}<br>
                <b>固定存款：$${fixedSave.toLocaleString()}</b><br>
                <b>生活開銷：$${lifeMoney.toLocaleString()}</b> (含餐費、社交)<br>
                ${lifeMoney < 8000 ? '<span class="text-red-500">⚠️ 生活費低於 $8,000，通膨環境下壓力較大，建議改為「保守存款」比例。</span>' : '✅ 生活費尚屬合理範圍。'}
            `;

            const yearlyGain = (55000 - salary) * 12;
            const roi = ((yearlyGain / 55000) * 100).toFixed(0);
            document.getElementById('roi-text').innerHTML = `考上後月薪跳升至 $55,000，年收增加 $${yearlyGain.toLocaleString()}。這筆補習投資的第一年投報率為 <b>${roi}%</b>，達成 80 萬計畫的速度將提升 2.5 倍。`;
        }

        function calculatePlan() {
            const other = Number(document.getElementById('other-save').value) || 0;
            const stock = Number(document.getElementById('in-stock').value) || 0;
            const total = FIXED_EMERGENCY + other + stock;
            const target = 800000;
            const percent = ((total / target) * 100).toFixed(1);
            
            document.getElementById('progress-ui').innerHTML = `
                <div class="text-4xl font-bold text-red-800">${percent}%</div>
                <div class="w-full bg-gray-100 h-2 rounded-full mt-4 overflow-hidden"><div class="bg-red-800 h-full" style="width:${percent}%"></div></div>
                <p class="text-xs text-gray-500 mt-4">總資產：$${total.toLocaleString()}<br>距目標還差 $${(target - total).toLocaleString()}</p>
            `;
        }
        
        autoAnalyze();
    </script>
</body>
</html>
