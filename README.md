# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>MUJI Finance | 日系理財管家</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root {
            --muji-bg: #F7F3F0;
            --muji-white: #FFFFFF;
            --muji-red: #7F2D2D;
            --muji-text: #444444;
            --muji-gray: #BCB8B1;
        }
        body { 
            background-color: var(--muji-bg); 
            color: var(--muji-text);
            font-family: "Helvetica Neue", Arial, "Heiti TC", "Microsoft JhengHei", sans-serif;
        }
        .muji-card { background: var(--muji-white); border: 1px solid #E6E2DF; border-radius: 12px; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        .muji-btn { background-color: var(--muji-red); color: white; border-radius: 4px; transition: opacity 0.2s; }
        .muji-btn:active { opacity: 0.8; }
        input { border-bottom: 1px solid var(--muji-gray) !important; background: transparent !important; border-radius: 0 !important; }
        input:focus { border-bottom: 1px solid var(--muji-red) !important; outline: none; }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 bg-white border-b border-gray-200 sticky top-0 z-50 text-center">
        <h1 class="text-lg font-medium tracking-widest text-gray-800">資產管理概況</h1>
    </header>

    <main class="p-5 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="muji-card p-8 mb-6 text-center">
                <p class="text-xs tracking-widest text-gray-400 mb-2">CURRENT NET WORTH</p>
                <h2 id="display-total" class="text-3xl font-light text-gray-800">$ 0</h2>
            </div>

            <div class="muji-card p-6 mb-6">
                <h3 class="text-sm font-bold mb-5 border-l-4 border-red-800 pl-3">資產紀錄</h3>
                <div class="space-y-5">
                    <div class="flex justify-between items-center">
                        <label class="text-sm">現金</label>
                        <input type="number" id="in-cash" oninput="liveUpdate()" class="text-right w-1/2 p-1" placeholder="0">
                    </div>
                    <div class="flex justify-between items-center">
                        <label class="text-sm">投資市值</label>
                        <input type="number" id="in-stock" oninput="liveUpdate()" class="text-right w-1/2 p-1" placeholder="0">
                    </div>
                    <div class="flex justify-between items-center">
                        <label class="text-sm">定期儲蓄</label>
                        <input type="number" id="in-save" oninput="liveUpdate()" class="text-right w-1/2 p-1" placeholder="0">
                    </div>
                    <button onclick="saveData()" class="muji-btn w-full py-3 mt-4 text-sm tracking-widest font-medium">儲存數據</button>
                </div>
            </div>

            <div class="muji-card p-6">
                <canvas id="assetChart"></canvas>
            </div>
        </div>

        <div id="tab-salary" class="tab-content">
            <div class="muji-card p-6 mb-6">
                <h3 class="text-sm font-bold mb-5 border-l-4 border-red-800 pl-3">每月薪資運用試算</h3>
                <div class="space-y-4">
                    <div>
                        <label class="text-xs text-gray-400">預計月薪輸入</label>
                        <input type="number" id="monthly-salary" oninput="analyzeSalary()" class="w-full p-2 text-lg" placeholder="請輸入金額">
                    </div>
                    <div class="grid grid-cols-2 gap-4 mt-4">
                        <div class="p-3 bg-stone-50 rounded">
                            <p class="text-[10px] text-gray-400">生活開銷 (50%)</p>
                            <p id="calc-living" class="text-sm font-medium">$ 0</p>
                        </div>
                        <div class="p-3 bg-stone-50 rounded">
                            <p class="text-[10px] text-gray-400">投資配置 (30%)</p>
                            <p id="calc-invest" class="text-sm font-medium">$ 0</p>
                        </div>
                    </div>
                </div>
            </div>

            <div class="muji-card p-6 mb-6">
                <h3 class="text-sm font-bold mb-3">理財分析建議</h3>
                <div id="salary-analysis" class="text-xs leading-relaxed text-gray-500 space-y-3">
                    請輸入薪資以開啟分析。
                </div>
            </div>

            <div class="muji-card p-6 border-dashed border-2">
                <h3 class="text-sm font-bold mb-2">目標：2年累積 80萬</h3>
                <div id="goal-status" class="text-xs text-gray-500">
                    </div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 flex justify-around p-4 shadow-sm">
        <button onclick="switchTab('assets')" class="nav-btn flex flex-col items-center text-red-800">
            <i class="fas fa- minimalist fa-leaf text-lg"></i>
            <span class="text-[10px] mt-1">資產</span>
        </button>
        <button onclick="switchTab('salary')" class="nav-btn flex flex-col items-center text-gray-400">
            <i class="fas fa-sliders-h text-lg"></i>
            <span class="text-[10px] mt-1">規劃</span>
        </button>
    </nav>

    <script>
        let myData = JSON.parse(localStorage.getItem('muji_finance')) || { cash: 0, stock: 0, save: 0 };
        let chart;

        function liveUpdate() {
            const cash = parseFloat(document.getElementById('in-cash').value) || 0;
            const stock = parseFloat(document.getElementById('in-stock').value) || 0;
            const save = parseFloat(document.getElementById('in-save').value) || 0;
            const total = cash + stock + save;
            document.getElementById('display-total').innerText = '$ ' + total.toLocaleString();
            renderChart(cash, stock, save);
        }

        function switchTab(name) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.getElementById('tab-' + name).classList.add('active');
            const btns = document.querySelectorAll('.nav-btn');
            btns[0].style.color = name === 'assets' ? '#7F2D2D' : '#BCB8B1';
            btns[1].style.color = name === 'salary' ? '#7F2D2D' : '#BCB8B1';
            if(name === 'salary') analyzeSalary();
        }

        function analyzeSalary() {
            const salary = parseFloat(document.getElementById('monthly-salary').value) || 0;
            const totalAssets = (parseFloat(document.getElementById('in-cash').value) || 0) + 
                                (parseFloat(document.getElementById('in-stock').value) || 0) + 
                                (parseFloat(document.getElementById('in-save').value) || 0);
            
            const living = salary * 0.5;
            const invest = salary * 0.3;
            const saving = salary * 0.2;

            document.getElementById('calc-living').innerText = '$ ' + living.toLocaleString();
            document.getElementById('calc-invest').innerText = '$ ' + invest.toLocaleString();

            const remainGoal = 800000 - totalAssets;
            const monthsNeeded = remainGoal > 0 ? (remainGoal / (invest + saving)).toFixed(1) : 0;

            document.getElementById('salary-analysis').innerHTML = `
                <p>1. 依照<b> 5:3:2 法則</b>，您每月可投入 <b>$${invest.toLocaleString()}</b> 進行定期定額投資。</p>
                <p>2. 保持此進度，配合複利效應，您的資產成長曲線將優於 85% 的同薪資族群。</p>
                <p>3. 建議將定期定額標的分散在低成本指數型 ETF，以抗衡每年約 3% 的通膨。</p>
            `;

            document.getElementById('goal-status').innerHTML = `
                目前資產：$${totalAssets.toLocaleString()}<br>
                距離目標：$${Math.max(remainGoal, 0).toLocaleString()}<br>
                預計還需：<b>${monthsNeeded} 個月</b> (以目前薪資配置計算)
            `;
        }

        function saveData() {
            myData.cash = parseFloat(document.getElementById('in-cash').value) || 0;
            myData.stock = parseFloat(document.getElementById('in-stock').value) || 0;
            myData.save = parseFloat(document.getElementById('in-save').value) || 0;
            localStorage.setItem('muji_finance', JSON.stringify(myData));
            alert('數據已儲存');
        }

        function renderChart(c, s, v) {
            const ctx = document.getElementById('assetChart').getContext('2d');
            if(chart) chart.destroy();
            chart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: ['現金', '投資', '儲蓄'],
                    datasets: [{
                        data: [c, s, v],
                        backgroundColor: ['#BCB8B1', '#7F2D2D', '#E6E2DF'],
                        borderWidth: 0
                    }]
                },
                options: { 
                    cutout: '85%', 
                    plugins: { legend: { position: 'bottom', labels: { font: { size: 10 } } } } 
                }
            });
        }

        document.getElementById('in-cash').value = myData.cash;
        document.getElementById('in-stock').value = myData.stock;
        document.getElementById('in-save').value = myData.save;
        liveUpdate();
    </script>
</body>
</html>
