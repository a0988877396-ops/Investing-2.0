# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>理財管家 Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        .tab-content { display: none; }
        .active { display: block; }
        .glass-card { background: white; border-radius: 20px; box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1); }
    </style>
</head>
<body class="bg-gray-100 pb-24">

    <header class="p-6 bg-white flex justify-between items-center shadow-sm">
        <h1 class="text-xl font-bold text-blue-600 italic">FinanceFlow</h1>
        <div class="text-xs text-gray-400" id="last-update">尚未儲存</div>
    </header>

    <main class="p-4 max-w-md mx-auto">
        
        <div id="tab-assets" class="tab-content active">
            <div class="bg-blue-600 p-6 rounded-[2.5rem] text-white mb-6 shadow-lg">
                <p class="text-sm opacity-80">目前總淨資產</p>
                <h2 id="display-total" class="text-3xl font-bold mt-1">$ 0</h2>
                <div class="mt-4 flex justify-between text-xs bg-white/10 p-3 rounded-xl">
                    <span>現金: <span id="lbl-cash">$0</span></span>
                    <span>投資: <span id="lbl-stock">$0</span></span>
                    <span>儲蓄: <span id="lbl-save">$0</span></span>
                </div>
            </div>

            <div class="glass-card p-6 mb-6">
                <h3 class="font-bold text-gray-700 mb-4"><i class="fas fa-edit mr-2"></i>資產更新 (記帳)</h3>
                <div class="space-y-4">
                    <div>
                        <label class="block text-xs text-gray-500 mb-1">現金帳戶 / 錢包</label>
                        <input type="number" id="in-cash" class="w-full border-b-2 border-gray-100 p-2 focus:border-blue-500 outline-none transition-all" placeholder="輸入金額">
                    </div>
                    <div>
                        <label class="block text-xs text-gray-500 mb-1">股票 / 基金帳戶</label>
                        <input type="number" id="in-stock" class="w-full border-b-2 border-gray-100 p-2 focus:border-blue-500 outline-none transition-all" placeholder="輸入金額">
                    </div>
                    <div>
                        <label class="block text-xs text-gray-500 mb-1">定期儲蓄 / 定存</label>
                        <input type="number" id="in-save" class="w-full border-b-2 border-gray-100 p-2 focus:border-blue-500 outline-none transition-all" placeholder="輸入金額">
                    </div>
                    <button onclick="saveData()" class="w-full bg-blue-600 text-white font-bold py-3 rounded-xl shadow-md active:scale-95 transition-transform">
                        確認更新資產
                    </button>
                </div>
            </div>

            <div class="glass-card p-6">
                <canvas id="assetChart"></canvas>
            </div>
        </div>

        <div id="tab-goals" class="tab-content">
            <div class="glass-card p-6 mb-6">
                <h3 class="font-bold text-gray-700 mb-4">2年 80萬 儲蓄目標</h3>
                <div class="mb-4">
                    <div class="flex justify-between text-sm mb-2">
                        <span id="goal-percent">進度 0%</span>
                        <span id="goal-remain">還差 $800,000</span>
                    </div>
                    <div class="w-full bg-gray-100 h-4 rounded-full overflow-hidden">
                        <div id="goal-bar" class="bg-green-500 h-full transition-all duration-500" style="width: 0%"></div>
                    </div>
                </div>
                <div class="bg-green-50 p-4 rounded-2xl border border-green-100">
                    <p class="text-sm text-green-800 leading-relaxed" id="goal-advice">
                        載入中...
                    </p>
                </div>
            </div>

            <div class="glass-card p-6">
                <h3 class="font-bold text-gray-700 mb-2">投資分析與建議</h3>
                <p class="text-xs text-gray-400 mb-4">基於當前通膨率 3% 估算</p>
                <div id="analysis-text" class="text-sm text-gray-600 space-y-3">
                    </div>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/80 backdrop-blur-md border-t flex justify-around p-4">
        <button onclick="switchTab('assets')" id="btn-assets" class="flex flex-col items-center text-blue-600">
            <i class="fas fa-wallet text-xl"></i>
            <span class="text-[10px] mt-1 font-bold">資產記帳</span>
        </button>
        <button onclick="switchTab('goals')" id="btn-goals" class="flex flex-col items-center text-gray-400">
            <i class="fas fa-bullseye text-xl"></i>
            <span class="text-[10px] mt-1 font-bold">目標試算</span>
        </button>
    </nav>

    <script>
        // 1. 資料初始化 (從手機儲存空間讀取)
        let myData = JSON.parse(localStorage.getItem('user_finance')) || {
            cash: 0, stock: 0, save: 0, lastUpdate: '無紀錄'
        };

        // 2. 切換頁籤
        function switchTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.getElementById('tab-' + tabName).classList.add('active');
            
            document.getElementById('btn-assets').className = tabName === 'assets' ? 'flex flex-col items-center text-blue-600' : 'flex flex-col items-center text-gray-400';
            document.getElementById('btn-goals').className = tabName === 'goals' ? 'flex flex-col items-center text-blue-600' : 'flex flex-col items-center text-gray-400';
            
            if(tabName === 'goals') calculateGoals();
        }

        // 3. 儲存功能
        function saveData() {
            myData.cash = parseFloat(document.getElementById('in-cash').value) || 0;
            myData.stock = parseFloat(document.getElementById('in-stock').value) || 0;
            myData.save = parseFloat(document.getElementById('in-save').value) || 0;
            myData.lastUpdate = new Date().toLocaleString();
            
            localStorage.setItem('user_finance', JSON.stringify(myData));
            alert('資產資料已成功儲存並同步試算！');
            updateUI();
        }

        // 4. 更新畫面
        function updateUI() {
            const total = myData.cash + myData.stock + myData.save;
            document.getElementById('display-total').innerText = '$ ' + total.toLocaleString();
            document.getElementById('lbl-cash').innerText = '$' + myData.cash.toLocaleString();
            document.getElementById('lbl-stock').innerText = '$' + myData.stock.toLocaleString();
            document.getElementById('lbl-save').innerText = '$' + myData.save.toLocaleString();
            document.getElementById('last-update').innerText = '上次更新: ' + myData.lastUpdate;
            
            // 填入輸入框
            document.getElementById('in-cash').value = myData.cash;
            document.getElementById('in-stock').value = myData.stock;
            document.getElementById('in-save').value = myData.save;

            renderChart(myData);
        }

        // 5. 目標試算邏輯
        function calculateGoals() {
            const total = myData.cash + myData.stock + myData.save;
            const target = 800000;
            const percent = Math.min((total / target) * 100, 100).toFixed(1);
            const remain = Math.max(target - total, 0);
            
            document.getElementById('goal-percent').innerText = `進度 ${percent}%`;
            document.getElementById('goal-remain').innerText = `還差 $${remain.toLocaleString()}`;
            document.getElementById('goal-bar').style.width = percent + '%';

            // 試算建議
            const monthlySave = (remain / 24).toFixed(0);
            document.getElementById('goal-advice').innerHTML = 
                `距離 80 萬目標還需 <b>$${remain.toLocaleString()}</b>。<br>
                 若要在 2 年內達成，接下來每個月需平均存下 <b>$${monthlySave.toLocaleString()}</b>。`;
            
            // 投資建議
            const inflationImpact = (total * 0.03).toFixed(0);
            document.getElementById('analysis-text').innerHTML = `
                <div class="p-3 bg-red-50 rounded-lg text-red-700">⚠️ 通膨警示：按 3% 通膨計，您的資產明年購買力將縮水 約 $${inflationImpact.toLocaleString()}。</div>
                <div class="p-3 bg-blue-50 rounded-lg text-blue-700">💡 策略：目前投資佔比為 ${((myData.stock/total)*100 || 0).toFixed(1)}%，建議將閒置現金的 20% 轉入低風險標的。</div>
            `;
        }

        // 6. 圖表繪製
        let chart;
        function renderChart(d) {
            const ctx = document.getElementById('assetChart').getContext('2d');
            if(chart) chart.destroy();
            chart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: ['現金', '股票', '儲蓄'],
                    datasets: [{
                        data: [d.cash, d.stock, d.save],
                        backgroundColor: ['#3b82f6', '#8b5cf6', '#f59e0b'],
                        borderWidth: 0
                    }]
                },
                options:
