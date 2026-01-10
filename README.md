# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MUJI 財務管家</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root {
            --muji-bg: #F7F3F0;
            --muji-red: #7F2D2D;
            --muji-text: #444444;
            --muji-gray: #BCB8B1;
        }
        body { 
            background-color: var(--muji-bg); 
            color: var(--muji-text);
            font-family: "PingFang TC", "Heiti TC", sans-serif;
            -webkit-font-smoothing: antialiased;
        }
        .muji-card { background: white; border: 1px solid #E6E2DF; border-radius: 8px; margin-bottom: 1.5rem; box-shadow: 0 2px 5px rgba(0,0,0,0.02); }
        .muji-btn { background-color: var(--muji-red); color: white; border-radius: 4px; transition: opacity 0.2s; }
        .muji-btn:active { opacity: 0.8; }
        input { border-bottom: 1px solid var(--muji-gray) !important; background: transparent !important; border-radius: 0 !important; }
        input:focus { border-bottom: 1px solid var(--muji-red) !important; outline: none; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 bg-white border-b border-gray-200 sticky top-0 z-50 text-center">
        <h1 class="text-base font-medium tracking-[0.2em] text-gray-800">資產管理與分析</h1>
    </header>

    <main class="p-5 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-8 text-center">
                <p class="text-[10px] tracking-widest text-gray-400 mb-2 uppercase">Current Assets</p>
                <h2 id="display-total" class="text-3xl font-light text-gray-800">$ 0</h2>
            </div>

            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-5 border-l-4 border-red-800 pl-3">目前資產紀錄</h3>
                <div class="space-y-5 text-sm">
                    <div class="flex justify-between items-center">
                        <span>現金</span><input type="number" id="in-cash" oninput="liveUpdate()" class="text-right w-1/2 p-1" placeholder="0">
                    </div>
                    <div class="flex justify-between items-center">
                        <span>股票投資</span><input type="number" id="in-stock" oninput="liveUpdate()" class="text-right w-1/2 p-1" placeholder="0">
                    </div>
                    <div class="flex justify-between items-center">
                        <span>銀行儲蓄</span><input type="number" id="in-save" oninput="liveUpdate()" class="text-right w-1/2 p-1" placeholder="0">
                    </div>
                </div>
            </div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="muji-card p-6">
                <h3 class="text-sm font-bold mb-5 border-l-4 border-red-800 pl-3">月收支自定義</h3>
                <div class="space-y-4">
                    <div class="flex justify-between items-center border-b pb-2">
                        <span class="text-sm">目前月薪收入</span>
                        <input type="number" id="salary" value="32000" class="text-right w-1/2 font-bold p-1">
                    </div>
                    <div id="expense-container" class="space-y-3">
                        <div class="flex gap-2">
                            <input type="text" placeholder="支出項目" class="w-2/3 exp-name text-xs p-1">
                            <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right p-1">
                        </div>
                    </div>
                    <button onclick="addExp()" class="text-[10px] text-gray-400 underline">+ 新增支出品項</button>
                    <button onclick="renderPie()" class="muji-btn w-full py-3 mt-4 text-xs tracking-widest">生成收支比例分析圖</button>
                </div>
            </div>

            <div id="chart-area" class="muji-card p-4 hidden">
                <canvas id="expenseChart"></canvas>
            </div>

            <div class="muji-card p-6 border-dashed border-2 bg-stone-50/50">
                <h3 class="text-sm font-bold mb-4">職涯升遷・進修試算</h3>
                <div class="space-y-4 text-xs">
                    <div class="flex justify-between">
                        <span>目標職位月薪</span><input type="number" id="target-salary" value="55000" class="w-24 text-right">
                    </div>
                    <div class="flex justify-between">
                        <span>補習費 (5年總計)</span><input type="number" id="exam-cost" value="55000" class="w-24 text-right">
                    </div>
                    <button onclick="runROI()" class="w-full border border-gray-300 py-2 mt-2 hover:bg-white transition-colors">執行數據試算</button>
                </div>
                <div id="roi-box" class="mt-5 p-4 bg-white rounded border border-gray-100 text-[11px] leading-relaxed text-gray-500 min-h-[40px]">
                    請輸入數據並執行試算。
                </div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-100 flex justify-around p-4 shadow-sm">
        <button onclick="switchTab('assets')" class="nav-btn flex flex-col items-center text-red-800">
            <i class="fas fa-wallet text-lg"></i><span class="text-[10px] mt-1 font-medium">資產記帳</span>
        </button>
        <button onclick="switchTab('analysis')" class="nav-btn flex flex-col items-center text-gray-400">
            <i class="fas fa-chart-line text-lg"></i><span class="text-[10px] mt-1 font-medium">深度分析</span>
        </button>
    </nav>

    <script>
        let myPie = null;

        function switchTab(t) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById('tab-' + t).classList.add('active');
            const btns = document.querySelectorAll('.nav-btn');
            btns[0].style.color = t === 'assets' ? '#7F2D2D' : '#BCB8B1';
            btns[1].style.color = t === 'analysis' ? '#7F2D2D' : '#BCB8B1';
        }

        function addExp() {
            const div = document.createElement('div');
            div.className = "flex gap-2";
            div.innerHTML = `<input type="text" placeholder="品項" class="w-2/3 exp-name text-xs p-1">
                             <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right p-1">`;
            document.getElementById('expense-container').appendChild(div);
        }

        function liveUpdate() {
            const total = (Number(document.getElementById('in-cash').value)||0) + 
                          (Number(document.getElementById('in-stock').value)||0) + 
                          (Number(document.getElementById('in-save').value)||0);
            document.getElementById('display-total').innerText = '$ ' + total.toLocaleString();
        }

        function renderPie() {
            document.getElementById('chart-area').classList.remove('hidden');
            const names = Array.from(document.querySelectorAll('.exp-name')).map(i => i.value || '未分類');
            const vals = Array.from(document.querySelectorAll('.exp-val')).map(i => Number(i.value) || 0);
            
            const ctx = document.getElementById('expenseChart').getContext('2d');
            if(myPie) myPie.destroy();
            myPie = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: names,
                    datasets: [{
                        data: vals,
                        backgroundColor: ['#7F2D2D', '#BCB8B1', '#D6D2C4', '#A8A39D', '#E6E2DF'],
                        borderWidth: 2,
                        borderColor: '#ffffff'
                    }]
                },
                options: { cutout: '70%', plugins: { legend: { position: 'bottom', labels: { font: { size: 10 } } } } }
            });
        }

        function runROI() {
            const current = Number(document.getElementById('salary').value) || 0;
            const target = Number(document.getElementById('target-salary').value) || 0;
            const cost = Number(document.getElementById('exam-cost').value) || 0;
            const diff = target - current;
            const months = diff > 0 ? (cost / diff).toFixed(1) : '無法計算';

            document.getElementById('roi-box').innerHTML = `
                <b>分析結果：</b><br>
                1. 考取後每月薪資將淨增 <b>$${diff.toLocaleString()}</b>。<br>
                2. 補習總投入費用預計在 <b>${months} 個月</b>內完整回收。<br>
                3. 長期來看，這是一筆回報率極高的自我投資。
            `;
        }

        liveUpdate();
    </script>
</body>
</html>
