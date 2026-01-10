# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MUJI Finance Pro | 深度理財分析</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root { --muji-bg: #F7F3F0; --muji-red: #7F2D2D; --muji-text: #444444; }
        body { background-color: var(--muji-bg); color: var(--muji-text); font-family: "PingFang TC", sans-serif; }
        .muji-card { background: white; border: 1px solid #E6E2DF; border-radius: 12px; margin-bottom: 1.5rem; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        input { border-bottom: 1px solid #BCB8B1 !important; background: transparent !important; }
        input:focus { border-bottom: 1px solid var(--muji-red) !important; outline: none; }
        .tag { font-size: 10px; padding: 2px 8px; border-radius: 4px; background: #E6E2DF; color: #7F2D2D; }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 bg-white border-b text-center sticky top-0 z-50">
        <h1 class="text-lg font-medium tracking-widest">深度財務分析</h1>
    </header>

    <main class="p-5 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-8 text-center">
                <p class="text-[10px] tracking-[0.2em] text-gray-400 mb-2">NET WORTH</p>
                <h2 id="display-total" class="text-3xl font-light">$ 0</h2>
            </div>
            
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-red-800 pl-3">目前資產餘額</h3>
                <div class="space-y-4 text-sm">
                    <div class="flex justify-between"><span>現金</span><input type="number" id="in-cash" oninput="liveUpdate()" class="text-right w-1/2"></div>
                    <div class="flex justify-between"><span>投資</span><input type="number" id="in-stock" oninput="liveUpdate()" class="text-right w-1/2"></div>
                    <div class="flex justify-between"><span>定存</span><input type="number" id="in-save" oninput="liveUpdate()" class="text-right w-1/2"></div>
                </div>
            </div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-4 border-l-4 border-red-800 pl-3">月收支自定義</h3>
                <div class="space-y-4">
                    <div class="flex justify-between items-center">
                        <span class="text-sm">月薪收入</span>
                        <input type="number" id="salary" class="text-right w-1/2 font-bold text-red-800" value="32000" oninput="deepAnalyze()">
                    </div>
                    <hr>
                    <div id="expense-list" class="space-y-3">
                        <div class="flex gap-2 items-center">
                            <input type="text" placeholder="品項(如:房租)" class="text-xs w-2/3 exp-name">
                            <input type="number" placeholder="金額" class="text-xs w-1/3 text-right exp-val" oninput="deepAnalyze()">
                        </div>
                    </div>
                    <button onclick="addExpenseRow()" class="text-[10px] text-gray-400 underline">+ 新增開銷品項</button>
                </div>
            </div>

            <div class="muji-card p-6 border-2 border-red-100 bg-red-50/30">
                <h3 class="text-sm font-bold mb-3 text-red-900">向上考試・進修試算</h3>
                <div class="space-y-3 text-xs">
                    <div class="flex justify-between"><span>目標職位月薪</span><input type="number" id="target-salary" class="text-right w-24" value="55000" oninput="deepAnalyze()"></div>
                    <div class="flex justify-between"><span>預計補習總費用</span><input type="number" id="exam-cost" class="text-right w-24" value="40000" oninput="deepAnalyze()"></div>
                </div>
                <div id="exam-roi" class="mt-4 p-3 bg-white rounded-lg text-[11px] leading-relaxed text-gray-600 shadow-sm">
                    </div>
            </div>

            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-3">存錢與投資建議</h3>
                <div id="plan-result" class="text-xs space-y-3 text-gray-500">
                    </div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t flex justify-around p-4 shadow-sm">
        <button onclick="switchTab('assets')" class="nav-btn flex flex-col items-center text-red-800">
            <i class="fas fa-wallet text-lg"></i><span class="text-[10px] mt-1">資產</span>
        </button>
        <button onclick="switchTab('analysis')" class="nav-btn flex flex-col items-center text-gray-400">
            <i class="fas fa-microscope text-lg"></i><span class="text-[10px] mt-1">深度分析</span>
        </button>
    </nav>

    <script>
        function addExpenseRow() {
            const div = document.createElement('div');
            div.className = "flex gap-2 items-center";
            div.innerHTML = `<input type="text" placeholder="品項名稱" class="text-xs w-2/3 exp-name">
                             <input type="number" placeholder="金額" class="text-xs w-1/3 text-right exp-val" oninput="deepAnalyze()">`;
            document.getElementById('expense-list').appendChild(div);
        }

        function deepAnalyze() {
            const salary = parseFloat(document.getElementById('salary').value) || 0;
            const targetSalary = parseFloat(document.getElementById('target-salary').value) || 0;
            const examCost = parseFloat(document.getElementById('exam-cost').value) || 0;
            
            // 計算總開銷
            let totalExp = 0;
            document.querySelectorAll('.exp-val').forEach(input => {
                totalExp += parseFloat(input.value) || 0;
            });

            const netCashflow = salary - totalExp;
            const targetNetCashflow = targetSalary - totalExp;

            // 1. 考試 ROI 分析
            const salaryDiff = targetSalary - salary;
            const payoffMonths = salaryDiff > 0 ? (examCost / salaryDiff).toFixed(1) : '∞';
            
            document.getElementById('exam-roi').innerHTML = `
                <p>✨ <b>回本分析：</b>考取後每月薪資增加 <b>$${salaryDiff.toLocaleString()}</b>。</p>
                <p>只需 <b>${payoffMonths} 個月</b> 即可賺回補習費。這是一筆投報率極高的自我投資。</p>
            `;

            // 2. 存錢與投資比例建議
            const investRatio = 0.6; // 淨現金流的 60% 投資
            const saveRatio = 0.4;   // 淨現金流的 40% 存款
            
            document.getElementById('plan-result').innerHTML = `
                <div class="flex justify-between"><span>每月結餘 (可運用金額)</span><span class="text-red-800 font-bold">$${netCashflow.toLocaleString()}</span></div>
                <hr>
                <p>📍 <b>資產加速策略：</b></p>
                <p>• <b>定期定額：</b>建議撥入 <b>$${(netCashflow * investRatio).toFixed(0)}</b> (佔結餘60%)</p>
                <p>• <b>緊急預備：</b>建議撥入 <b>$${(netCashflow * saveRatio).toFixed(0)}</b> (佔結餘40%)</p>
                <p class="mt-2 text-gray-400 italic">※ 若考取成功，每月結餘將跳升至 $${targetNetCashflow.toLocaleString()}，累積 80 萬的速度將提升 <b>${(targetNetCashflow/netCashflow).toFixed(1)} 倍</b>。</p>
            `;
        }

        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.getElementById('tab-' + name).classList.add('active');
            const btns = document.querySelectorAll('.nav-btn');
            btns[0].style.color = name === 'assets' ? '#7F2D2D' : '#BCB8B1';
            btns[1].style.color = name === 'analysis' ? '#7F2D2D' : '#BCB8B1';
            if(name === 'analysis') deepAnalyze();
        }

        function liveUpdate() {
            const c = parseFloat(document.getElementById('in-cash').value) || 0;
            const s = parseFloat(document.getElementById('in-stock').value) || 0;
            const v = parseFloat(document.getElementById('in-save').value) || 0;
            document.getElementById('display-total').innerText = '$ ' + (c+s+v).toLocaleString();
        }
        
        // 頁面預設值
        addExpenseRow(); // 預留一行
        liveUpdate();
    </script>
</body>
</html>
