# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Y2K 財務預言機</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --y2k-blue: #00d1ff;
            --y2k-purple: #7000ff;
            --y2k-dark: #050510;
        }
        body {
            background: var(--y2k-dark);
            background-image: linear-gradient(180deg, #050510 0%, #0a1a3a 100%);
            color: #fff;
            font-family: 'PingFang TC', sans-serif;
            min-height: 100vh;
        }
        .y2k-panel {
            background: rgba(255, 255, 255, 0.05);
            border: 2px solid var(--y2k-blue);
            box-shadow: 0 0 20px rgba(0, 209, 255, 0.3);
            border-radius: 15px;
            backdrop-filter: blur(10px);
        }
        .y2k-btn {
            background: linear-gradient(90deg, var(--y2k-blue), var(--y2k-purple));
            color: white;
            font-weight: bold;
            text-shadow: 0 0 5px rgba(0,0,0,0.5);
            transition: all 0.3s;
        }
        .y2k-btn:active { transform: scale(0.95); opacity: 0.8; }
        input {
            background: rgba(0, 209, 255, 0.1) !important;
            border: 1px solid var(--y2k-blue) !important;
            color: var(--y2k-blue) !important;
            border-radius: 5px;
            padding: 8px;
        }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .nav-active { color: var(--y2k-blue); text-shadow: 0 0 10px var(--y2k-blue); }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 text-center border-b border-cyan-900 bg-black/50">
        <h1 style="font-family: 'Orbitron';" class="text-2xl font-bold tracking-widest text-cyan-400">Y2K FINANCE ORACLE</h1>
        <p class="text-[10px] text-cyan-700 tracking-[0.5em]">數位財富預言機</p>
    </header>

    <main class="p-5 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="y2k-panel p-6 mb-6">
                <p class="text-xs text-cyan-500 mb-1">目前總資產淨值</p>
                <h2 id="display-total" class="text-4xl font-bold tracking-tighter text-white">$ 0</h2>
                <div class="grid grid-cols-1 gap-3 mt-6">
                    <div class="flex justify-between items-center text-sm">
                        <span>現金餘額</span>
                        <input type="number" id="in-cash" oninput="liveUpdate()" class="w-32 text-right">
                    </div>
                    <div class="flex justify-between items-center text-sm">
                        <span>股票/投資</span>
                        <input type="number" id="in-stock" oninput="liveUpdate()" class="w-32 text-right">
                    </div>
                    <div class="flex justify-between items-center text-sm">
                        <span>銀行儲蓄</span>
                        <input type="number" id="in-save" oninput="liveUpdate()" class="w-32 text-right">
                    </div>
                </div>
            </div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="y2k-panel p-6 mb-6">
                <h3 class="text-cyan-400 font-bold mb-4 flex items-center">
                    <span class="mr-2">■</span> 月收支自定義
                </h3>
                <div class="space-y-4">
                    <div class="flex justify-between items-center border-b border-cyan-900 pb-2">
                        <span class="text-sm">目前月薪</span>
                        <input type="number" id="salary" value="32000" class="w-24 text-right">
                    </div>
                    <div id="expense-list" class="space-y-3">
                        <div class="flex gap-2">
                            <input type="text" placeholder="項目" class="w-2/3 exp-name text-xs">
                            <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right">
                        </div>
                    </div>
                    <button onclick="addRow()" class="text-[10px] text-cyan-500 underline">+ 新增支出品項</button>
                    <button onclick="drawChart()" class="y2k-btn w-full py-3 mt-2 rounded-lg">生成收支圓餅圖分析</button>
                </div>
            </div>

            <div id="chart-panel" class="y2k-panel p-4 mb-6 hidden">
                <canvas id="expenseChart"></canvas>
            </div>

            <div class="y2k-panel p-6 bg-blue-900/20 border-blue-500">
                <h3 class="text-blue-400 font-bold mb-4 flex items-center">
                    <span class="mr-2">▲</span> 職涯升遷試算
                </h3>
                <div class="space-y-3 text-sm">
                    <div class="flex justify-between">
                        <span>目標職位月薪</span>
                        <input type="number" id="target-salary" value="55000" class="w-24 text-right">
                    </div>
                    <div class="flex justify-between">
                        <span>補習總費用 (5年)</span>
                        <input type="number" id="exam-cost" value="55000" class="w-24 text-right">
                    </div>
                </div>
                <button onclick="calculateExam()" class="w-full bg-white/10 border border-white/20 mt-4 py-2 text-xs hover:bg-white/20">執行 ROI 數據回報</button>
                <div id="exam-result" class="mt-4 p-3 bg-cyan-950/50 rounded text-xs leading-relaxed text-cyan-200 min-h-[50px]">
                    等待數據輸入...
                </div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-black/80 backdrop-blur-md border-t border-cyan-900 p-4 flex justify-around">
        <button onclick="switchTab('assets')" id="btn-assets" class="nav-active flex flex-col items-center">
            <i class="fas fa-wallet text-lg"></i><span class="text-[10px] mt-1">資產中心</span>
        </button>
        <button onclick="switchTab('analysis')" id="btn-analysis" class="text-gray-500 flex flex-col items-center">
            <i class="fas fa-chart-pie text-lg"></i><span class="text-[10px] mt-1">深度分析</span>
        </button>
    </nav>

    <script>
        let chartInstance = null;

        function switchTab(tab) {
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            document.getElementById('tab-' + tab).classList.add('active');
            
            document.getElementById('btn-assets').className = tab === 'assets' ? 'nav-active flex flex-col items-center' : 'text-gray-500 flex flex-col items-center';
            document.getElementById('btn-analysis').className = tab === 'analysis' ? 'nav-active flex flex-col items-center' : 'text-gray-500 flex flex-col items-center';
        }

        function addRow() {
            const div = document.createElement('div');
            div.className = "flex gap-2";
            div.innerHTML = `<input type="text" placeholder="項目" class="w-2/3 exp-name text-xs">
                             <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right">`;
            document.getElementById('expense-list').appendChild(div);
        }

        function liveUpdate() {
            const total = (Number(document.getElementById('in-cash').value)||0) + 
                          (Number(document.getElementById('in-stock').value)||0) + 
                          (Number(document.getElementById('in-save').value)||0);
            document.getElementById('display-total').innerText = '$ ' + total.toLocaleString();
        }

        function drawChart() {
            const panel = document.getElementById('chart-panel');
            panel.classList.remove('hidden');
            
            const names = Array.from(document.querySelectorAll('.exp-name')).map(i => i.value || '未命名');
            const vals = Array.from(document.querySelectorAll('.exp-val')).map(i => Number(i.value) || 0);
            
            const ctx = document.getElementById('expenseChart').getContext('2d');
            if(chartInstance) chartInstance.destroy();
            
            chartInstance = new Chart(ctx, {
                type: 'pie',
                data: {
                    labels: names,
                    datasets: [{
                        data: vals,
                        backgroundColor: ['#00d1ff', '#7000ff', '#ff00ff', '#00ff9f', '#ff8000'],
                        borderWidth: 0
                    }]
                },
                options: {
                    plugins: { legend: { labels: { color: '#00d1ff', font: { size: 10 } } } }
                }
            });
        }

        function calculateExam() {
            const curr = Number(document.getElementById('salary').value) || 0;
            const target = Number(document.getElementById('target-salary').value) || 0;
            const cost = Number(document.getElementById('exam-cost').value) || 0;
            const diff = target - curr;
            const months = diff > 0 ? (cost / diff).toFixed(1) : '無法計算';

            document.getElementById('exam-result').innerHTML = `
                > 數據分析完畢<br>
                > 考取後月薪增長: $${diff.toLocaleString()}<br>
                > 補習成本回收期: ${months} 個月<br>
                > 提示: 這是高回報投資，建議立即啟動學習計畫。
            `;
        }

        // 預設跑一次
        liveUpdate();
    </script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
</body>
</html>
