# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Y2K Money Oracle | Cortis 金主模式</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Noto+Sans+TC:wght@300;700&display=swap" rel="stylesheet">
    <style>
        :root { --y2k-pink: #ff00ff; --y2k-green: #00ff9f; --y2k-blue: #00d1ff; }
        body { 
            background: #0a0a0a; color: #fff; font-family: 'Noto Sans TC', sans-serif;
            background-image: radial-gradient(circle at 50% 50%, #1a1a2e 0%, #0a0a0a 100%);
        }
        .y2k-border { border: 2px solid var(--y2k-green); box-shadow: 0 0 15px var(--y2k-green), inset 0 0 5px var(--y2k-green); }
        .y2k-card { background: rgba(0, 0, 0, 0.7); backdrop-filter: blur(10px); border-radius: 0; position: relative; }
        .y2k-btn { background: var(--y2k-green); color: #000; font-weight: bold; text-transform: uppercase; clip-path: polygon(10% 0, 100% 0, 90% 100%, 0% 100%); }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        input { background: rgba(255,255,255,0.1) !important; color: var(--y2k-blue) !important; border: 1px solid var(--y2k-blue) !important; padding: 5px; }
        
        /* Cortis 閃卡動畫 */
        @keyframes glow { 0% { filter: hue-rotate(0deg) brightness(1); } 50% { filter: hue-rotate(180deg) brightness(1.5); } 100% { filter: hue-rotate(360deg) brightness(1); } }
        .cortis-frame { border: 4px double var(--y2k-pink); animation: glow 3s infinite linear; background: #000; }
        .glitch-text { text-shadow: 2px 0 var(--y2k-pink), -2px 0 var(--y2k-blue); font-family: 'Orbitron', sans-serif; }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 border-b-2 border-pink-500 text-center bg-black">
        <h1 class="glitch-text text-2xl tracking-tighter">Y2K FINANCE ORACLE</h1>
    </header>

    <main class="p-5 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="y2k-card y2k-border p-6 mb-6">
                <p class="text-[10px] text-cyan-400 tracking-widest uppercase">Total Net Worth</p>
                <h2 id="display-total" class="text-4xl font-bold text-white mt-2">$ 0</h2>
                <div class="mt-4 grid grid-cols-3 gap-2">
                    <input type="number" id="in-cash" oninput="liveUpdate()" placeholder="現金" class="text-xs">
                    <input type="number" id="in-stock" oninput="liveUpdate()" placeholder="股票" class="text-xs">
                    <input type="number" id="in-save" oninput="liveUpdate()" placeholder="儲蓄" class="text-xs">
                </div>
            </div>

            <div class="cortis-frame p-4 mb-6 relative overflow-hidden">
                <div class="flex items-center gap-4">
                    <div class="w-20 h-20 bg-gray-800 border-2 border-pink-500 flex-shrink-0">
                         <img src="https://api.dicebear.com/7.x/avataaars/svg?seed=Cortis" alt="Cortis" class="w-full h-full">
                    </div>
                    <div>
                        <h4 class="text-pink-500 font-bold text-sm">CORTIS 金主訓</h4>
                        <p class="text-[11px] leading-tight italic text-gray-300">「不要讓貧窮限制你的想像，要在自己身上投資，直到你變成那個發錢的人。」</p>
                    </div>
                </div>
            </div>
        </div>

        <div id="tab-analysis" class="tab-content">
            <div class="y2k-card p-6 mb-6 border-l-4 border-cyan-400">
                <h3 class="text-cyan-400 font-bold mb-4">收支矩陣</h3>
                <div class="space-y-4">
                    <div class="flex justify-between"><span>目前薪資</span><input type="number" id="salary" value="32000" class="w-24 text-right"></div>
                    <div id="expense-container" class="space-y-2">
                        <div class="flex gap-2">
                            <input type="text" placeholder="品項" class="w-2/3 exp-name text-xs">
                            <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right">
                        </div>
                    </div>
                    <button onclick="addExp()" class="text-[10px] text-cyan-400">+ ADD DATA ROW</button>
                    <button onclick="genChart()" class="y2k-btn w-full py-2 mt-4 text-xs">GENERATE ANALYSIS 圖表分析</button>
                </div>
            </div>

            <div id="chart-box" class="y2k-card p-4 mb-6 hidden border border-pink-500">
                <canvas id="expenseChart"></canvas>
            </div>

            <div class="y2k-card p-6 border-2 border-green-500 bg-green-950/20">
                <h3 class="text-green-400 font-bold mb-2 uppercase">Exam Boost (5-Year Plan)</h3>
                <div class="text-[11px] space-y-2 text-gray-300">
                    <div class="flex justify-between"><span>目標月薪</span><input type="number" id="target-salary" value="55000" class="w-20"></div>
                    <div class="flex justify-between"><span>補習費(5年總計)</span><input type="number" id="exam-cost" value="55000" class="w-20"></div>
                </div>
                <button onclick="calcROI()" class="w-full bg-white text-black text-[10px] font-bold mt-4 py-1">RUN ROI CALCULATION</button>
                <div id="roi-result" class="mt-4 text-[11px] text-green-300 leading-relaxed"></div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-black border-t-2 border-cyan-500 flex justify-around p-4 shadow-[0_-10px_20px_rgba(0,209,255,0.2)]">
        <button onclick="switchTab('assets')" class="text-cyan-400 flex flex-col items-center">
            <i class="fas fa-crosshairs"></i><span class="text-[9px] mt-1 uppercase">Assets</span>
        </button>
        <button onclick="switchTab('analysis')" class="text-pink-500 flex flex-col items-center">
            <i class="fas fa-bolt"></i><span class="text-[9px] mt-1 uppercase">Deep Analysis</span>
        </button>
    </nav>

    <script>
        let myChart;
        function switchTab(t) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.getElementById('tab-' + t).classList.add('active');
        }

        function addExp() {
            const div = document.createElement('div');
            div.className = "flex gap-2";
            div.innerHTML = `<input type="text" placeholder="品項" class="w-2/3 exp-name text-xs">
                             <input type="number" placeholder="金額" class="w-1/3 exp-val text-xs text-right">`;
            document.getElementById('expense-container').appendChild(div);
        }

        function liveUpdate() {
            const total = (Number(document.getElementById('in-cash').value)||0) + 
                          (Number(document.getElementById('in-stock').value)||0) + 
                          (Number(document.getElementById('in-save').value)||0);
            document.getElementById('display-total').innerText = '$ ' + total.toLocaleString();
        }

        function genChart() {
            document.getElementById('chart-box').classList.remove('hidden');
            const names = Array.from(document.querySelectorAll('.exp-name')).map(i => i.value || '未命名');
            const vals = Array.from(document.querySelectorAll('.exp-val')).map(i => Number(i.value) || 0);
            const ctx = document.getElementById('expenseChart');
            if(myChart) myChart.destroy();
            myChart = new Chart(ctx, {
                type: 'pie',
                data: {
                    labels: names,
                    datasets: [{ data: vals, backgroundColor: ['#ff00ff', '#00ff9f', '#00d1ff', '#ffff00', '#ff8000'] }]
                },
                options: { plugins: { legend: { labels: { color: '#fff' } } } }
            });
        }

        function calcROI() {
            const currentS = Number(document.getElementById('salary').value);
            const targetS = Number(document.getElementById('target-salary').value);
            const cost = Number(document.getElementById('exam-cost').value);
            const diff = targetS - currentS;
            const payoff = (cost / diff).toFixed(1);
            
            document.getElementById('roi-result').innerHTML = `
                >> 系統分析中...<br>
                >> 加薪額度: $${diff.toLocaleString()} / 月<br>
                >> 回本速度: ${payoff} 個月<br>
                >> 結論: 補習費是資產而非開銷。這筆投資將在 ${payoff} 個月後轉為純利潤。啟動金主模式。
            `;
        }
    </script>
</body>
</html>
