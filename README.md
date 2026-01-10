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
        .tab-content { display: none; }
        .tab-content.active { display: block; }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
    </style>
</head>
<body class="bg-gray-50 pb-24">

    <header class="p-6 bg-white shadow-sm sticky top-0 z-50">
        <h1 class="text-xl font-bold text-blue-600">我的財富看板</h1>
    </header>

    <main class="p-4 max-w-md mx-auto">
        <div id="tab-assets" class="tab-content active">
            <div class="bg-gradient-to-br from-blue-600 to-indigo-700 p-8 rounded-[2rem] text-white mb-6 shadow-xl">
                <p class="text-sm opacity-80">目前估算總資產</p>
                <h2 id="display-total" class="text-4xl font-black mt-2">$ 0</h2>
            </div>

            <div class="bg-white p-6 rounded-3xl shadow-sm mb-6 border border-gray-100">
                <h3 class="font-bold mb-4 text-gray-700">快速記帳 / 更新數值</h3>
                <div class="space-y-4">
                    <div>
                        <label class="text-xs text-gray-400 ml-1">現金餘額</label>
                        <input type="number" id="in-cash" oninput="liveUpdate()" class="w-full bg-gray-50 rounded-xl p-3 outline-none focus:ring-2 focus:ring-blue-400 transition-all" placeholder="0">
                    </div>
                    <div>
                        <label class="text-xs text-gray-400 ml-1">投資市值 (股票/基金)</label>
                        <input type="number" id="in-stock" oninput="liveUpdate()" class="w-full bg-gray-50 rounded-xl p-3 outline-none focus:ring-2 focus:ring-blue-400 transition-all" placeholder="0">
                    </div>
                    <div>
                        <label class="text-xs text-gray-400 ml-1">定存/儲蓄</label>
                        <input type="number" id="in-save" oninput="liveUpdate()" class="w-full bg-gray-50 rounded-xl p-3 outline-none focus:ring-2 focus:ring-blue-400 transition-all" placeholder="0">
                    </div>
                    <button onclick="saveData()" class="w-full bg-blue-600 text-white font-bold py-4 rounded-2xl mt-2 shadow-lg active:scale-95 transition-all">儲存並鎖定數據</button>
                </div>
            </div>

            <div class="bg-white p-6 rounded-3xl shadow-sm">
                <canvas id="assetChart"></canvas>
            </div>
        </div>

        <div id="tab-goals" class="tab-content">
            <div class="bg-white p-6 rounded-3xl shadow-sm mb-6">
                <h3 class="font-bold text-xl mb-4 text-gray-800">2年 80萬 目標進度</h3>
                <div class="mb-6">
                    <div class="flex justify-between font-bold mb-2">
                        <span id="goal-percent" class="text-blue-600">進度 0%</span>
                        <span id="goal-remain" class="text-gray-500 text-sm">還差 $800,000</span>
                    </div>
                    <div class="w-full bg-gray-100 h-5 rounded-full overflow-hidden">
                        <div id="goal-bar" class="bg-blue-500 h-full w-0 transition-all duration-700"></div>
                    </div>
                </div>
                <div id="goal-advice" class="p-4 bg-blue-50 text-blue-800 rounded-2xl text-sm leading-loose">
                    </div>
            </div>

            <div class="bg-red-50 p-6 rounded-3xl border border-red-100">
                <h3 class="font-bold text-red-600 mb-2 italic">⚠️ 通膨試算警告</h3>
                <p id="inflation-text" class="text-sm text-red-700"></p>
            </div>
        </div>
    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white/90 backdrop-blur-lg border-t flex justify-around p-4 shadow-2xl">
        <button onclick="switchTab('assets')" class="flex flex-col items-center text-blue-600 transition-colors">
            <i class="fas fa-wallet text-xl"></i><span class="text-[10px] mt-1">資產記帳</span>
        </button>
        <button onclick="switchTab('goals')" class="flex flex-col items-center text-gray-400 transition-colors">
            <i class="fas fa-chart-line text-xl"></i><span class="text-[10px] mt-1">目標試算</span>
        </button>
    </nav>

    <script>
        let myData = JSON.parse(localStorage.getItem('user_finance')) || { cash: 0, stock: 0, save: 0 };
        let chart;

        // 即時試算功能：輸入時數字會跟著動
        function liveUpdate() {
            const cash = parseFloat(document.getElementById('in-cash').value) || 0;
            const stock = parseFloat(document.getElementById('in-stock').value) || 0;
            const save = parseFloat(document.getElementById('in-save').value) || 0;
            const total = cash + stock + save;
            document.getElementById('display-total').innerText = '$ ' + total.toLocaleString();
            renderChart(cash, stock, save);
        }

        function switchTab(tabName) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.getElementById('tab-' + tabName).classList.add('active');
            
            // 更新按鈕顏色
            const btns = document.querySelectorAll('nav button');
            btns[0].style.color = tabName === 'assets' ? '#2563eb' : '#9ca3af';
            btns[1].style.color = tabName === 'goals' ? '#2563eb' : '#9ca3af';
            
            if(tabName === 'goals') calculateGoals();
        }

        function saveData() {
            myData.cash = parseFloat(document.getElementById('in-cash').value) || 0;
            myData.stock = parseFloat(document.getElementById('in-stock').value) || 0;
            myData.save = parseFloat(document.getElementById('in-save').value) || 0;
            localStorage.setItem('user_finance', JSON.stringify(myData));
            alert('數據已安全儲存至本機');
            liveUpdate();
        }

        function calculateGoals() {
            const total = (parseFloat(document.getElementById('in-cash').value) || 0) + 
                          (parseFloat(document.getElementById('in-stock').value) || 0) + 
                          (parseFloat(document.getElementById('in-save').value) || 0);
            const target = 800000;
            const remain = Math.max(target - total, 0);
            const percent = Math.min((total / target) * 100, 100).toFixed(1);
            
            document.getElementById('goal-percent').innerText = `進度 ${percent}%`;
            document.getElementById('goal-remain').innerText = `還差 $${remain.toLocaleString()}`;
            document.getElementById('goal-bar').style.width = percent + '%';
            document.getElementById('goal-advice').innerHTML = `目標 80 萬，目前已達成 <b>$${total.toLocaleString()}</b>。要在兩年內達標，每月需平均存款 <b>$${(remain/24).toFixed(0)}</b> 元。`;
            document.getElementById('inflation-text').innerText = `若持續保持 3% 通膨，您目前的資產在一年後的實際購買力將縮水約 $${(total * 0.03).toFixed(0)} 元。`;
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
                        backgroundColor: ['#3b82f6', '#8b5cf6', '#f59e0b'],
                        borderWidth: 0
                    }]
                },
                options: { cutout: '80%', plugins: { legend: { position: 'bottom' } } }
            });
        }

        // 頁面載入時讀取舊資料
        document.getElementById('in-cash').value = myData.cash;
        document.getElementById('in-stock').value = myData.stock;
        document.getElementById('in-save').value = myData.save;
        liveUpdate();
    </script>
</body>
</html>
