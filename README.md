# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MUJI 財務計畫 Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --muji-bg: #F7F3F0; --muji-red: #7F2D2D; --muji-text: #444444; }
        body { background-color: var(--muji-bg); color: var(--muji-text); font-family: "PingFang TC", sans-serif; }
        .muji-card { background: white; border: 1px solid #E6E2DF; border-radius: 12px; margin-bottom: 1.5rem; }
        .muji-btn { background-color: var(--muji-red); color: white; border-radius: 4px; }
        input { border-bottom: 1px solid #BCB8B1 !important; background: transparent !important; border-radius: 0 !important; }
        input:focus { border-bottom: 1px solid var(--muji-red) !important; outline: none; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .nav-btn { color: #BCB8B1; transition: 0.3s; }
        .nav-btn.active { color: var(--muji-red); }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 bg-white border-b sticky top-0 z-50 text-center">
        <h1 id="header-title" class="text-base font-medium tracking-[0.2em]">資產中心</h1>
    </header>

    <main class="p-5 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-8 text-center">
                <p class="text-[10px] tracking-widest text-gray-400 mb-2 uppercase">Current Assets</p>
                <h2 id="display-total" class="text-3xl font-light">$ 0</h2>
            </div>
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-5 border-l-4 border-red-800 pl-3">目前資產紀錄</h3>
                <div class="space-y-4 text-sm">
                    <div class="flex justify-between items-center"><span>現金</span><input type="number" id="in-cash" oninput="liveUpdate()" class="text-right w-1/2 p-1" value="0"></div>
                    <div class="flex justify-between items-center"><span>投資</span><input type="number" id="in-stock" oninput="liveUpdate()" class="text-right w-1/2 p-1" value="0"></div>
                    <div class="flex justify-between items-center"><span>儲蓄</span><input type="number" id="in-save" oninput="liveUpdate()" class="text-right w-1/2 p-1" value="0"></div>
                </div>
            </div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-red-800 pl-3">月收支自定義</h3>
                <div class="space-y-4">
                    <div class="flex justify-between items-center">
                        <span class="text-sm font-bold">目前月薪</span>
                        <input type="number" id="salary" value="32000" class="text-right w-1/2 font-bold p-1">
                    </div>
                    <div id="expense-container" class="space-y-3">
                        <div class="flex gap-2">
                            <input type="text" placeholder="品項(如:房租)" class="w-2/3 exp-name text-xs p-1">
                            <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right p-1" oninput="autoAnalyze()">
                        </div>
                    </div>
                    <button onclick="addExp()" class="text-[10px] text-gray-400 underline">+ 新增支出</button>
                    
                    <div id="allocation-result" class="mt-4 p-4 bg-stone-50 rounded text-xs leading-relaxed space-y-2">
                        </div>
                </div>
            </div>

            <div class="muji-card p-6 border-dashed border-2">
                <h3 class="text-sm font-bold mb-4">職涯升遷・進修試算</h3>
                <div class="space-y-4 text-xs">
                    <div class="flex justify-between"><span>目標職位月薪</span><input type="number" id="target-salary" value="55000" class="w-24 text-right"></div>
                    <div class="flex justify-between"><span>補習費 (5年總計)</span><input type="number" id="exam-cost" value="55000" class="w-24 text-right"></div>
                    <div id="roi-result" class="p-3 bg-red-50 text-red-900 rounded">
                        </div>
                </div>
            </div>
        </div>

        <div id="tab-plan" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-red-800 pl-3">2年 80 萬存款計畫</h3>
                <div class="space-y-6">
                    <div class="text-center">
                        <p class="text-xs text-gray-400">目前達成率</p>
                        <h4 id="plan-percent" class="text-3xl font-bold text-red-800">0%</h4>
                    </div>
                    <div class="w-full bg-gray-100 h-2 rounded-full overflow-hidden">
                        <div id="plan-bar" class="bg-red-800 h-full w-0 transition-all duration-700"></div>
                    </div>
                    <div id="plan-advice" class="text-xs leading-loose text-gray-500 bg-stone-50 p-4 rounded-lg">
                        </div>
                </div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t flex justify-around p-4">
        <button onclick="switchTab('assets')" class="nav-btn active flex flex-col items-center">
            <i class="fas fa-wallet text-lg"></i><span class="text-[10px] mt-1">資產記帳</span>
        </button>
        <button onclick="switchTab('analysis')" class="nav-btn flex flex-col items-center">
            <i class="fas fa-search-dollar text-lg"></i><span class="text-[10px] mt-1">深度分析</span>
        </button>
        <button onclick="switchTab('plan')" class="nav-btn flex flex-col items-center">
            <i class="fas fa-flag-checkered text-lg"></i><span class="text-[10px] mt-1">達標計畫</span>
        </button>
    </nav>

    <script>
        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById('tab-' + name).classList.add('active');
            document.querySelectorAll('.nav-btn').forEach((btn, idx) => {
                btn.classList.toggle('active', ['assets','analysis','plan'][idx] === name);
            });
            const titles = {'assets':'資產中心', 'analysis':'深度財務分析', 'plan':'2年80萬計畫'};
            document.getElementById('header-title').innerText = titles[name];
            if(name === 'analysis') autoAnalyze();
            if(name === 'plan') calculatePlan();
        }

        function addExp() {
            const div = document.createElement('div');
            div.className = "flex gap-2";
            div.innerHTML = `<input type="text" placeholder="品項" class="w-2/3 exp-name text-xs p-1">
                             <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right p-1" oninput="autoAnalyze()">`;
            document.getElementById('expense-container').appendChild(div);
        }

        function liveUpdate() {
            const total = (Number(document.getElementById('in-cash').value)||0) + 
                          (Number(document.getElementById('in-stock').value)||0) + 
                          (Number(document.getElementById('in-save').value)||0);
            document.getElementById('display-total').innerText = '$ ' + total.toLocaleString();
        }

        function autoAnalyze() {
            const salary = Number(document.getElementById('salary').value) || 0;
            let totalExp = 0;
            document.querySelectorAll('.exp-val').forEach(i => totalExp += Number(i.value) || 0);
            
            const surplus = salary - totalExp;
            const livingSave = (surplus * 0.4).toFixed(0); // 剩餘的40%作生活額外預備
            const fixedSave = (surplus * 0.6).toFixed(0);  // 剩餘的60%作固定存款

            document.getElementById('allocation-result').innerHTML = `
                <b>📊 結餘分配建議：</b><br>
                扣除開銷後剩餘：$${surplus.toLocaleString()}<br>
                建議<b>固定存款：$${fixedSave.toLocaleString()}</b> (結餘60%)<br>
                建議<b>彈性生活：$${livingSave.toLocaleString()}</b> (結餘40%)
            `;

            // ROI 計算
            const targetS = Number(document.getElementById('target-salary').value) || 0;
            const cost = Number(document.getElementById('exam-cost').value) || 0;
            const yearlyGain = (targetS - salary) * 12;
            const roi = ((yearlyGain / cost) * 100).toFixed(1);

            document.getElementById('roi-result').innerHTML = `
                <b>📈 考試投報率分析：</b><br>
                考取後第一年預計增收：$${yearlyGain.toLocaleString()}<br>
                <b>第一年投資報酬率：${roi}%</b><br>
                <span class="text-[10px]">※ 每投入 $1 補習費，考取首年即賺回 $${(yearlyGain/cost).toFixed(1)}</span>
            `;
        }

        function calculatePlan() {
            const totalAssets = (Number(document.getElementById('in-cash').value)||0) + 
                                (Number(document.getElementById('in-stock').value)||0) + 
                                (Number(document.getElementById('in-save').value)||0);
            const target = 800000;
            const percent = Math.min((totalAssets / target) * 100, 100).toFixed(1);
            const remain = target - totalAssets;
            const monthlyNeed = (remain / 24).toFixed(0);

            document.getElementById('plan-percent').innerText = percent + '%';
            document.getElementById('plan-bar').style.width = percent + '%';
            document.getElementById('plan-advice').innerHTML = `
                目標：$800,000 | 尚差：$${Math.max(remain, 0).toLocaleString()}<br><br>
                <b>📅 2年達成路徑：</b><br>
                在不考慮調薪下，每個月需淨存 <b>$${monthlyNeed.toLocaleString()}</b>。<br>
                ${monthlyNeed > 15000 ? '⚠️ 目前每月需存金額較高，建議加速職涯升遷。' : '✅ 目前目標在可控範圍內，請保持紀律。'}
            `;
        }

        liveUpdate();
    </script>
</body>
</html>
