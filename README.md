# Investing-2.0
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FinanceFlow | 個人財富管理</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        body { font-family: 'PingFang TC', 'Microsoft JhengHei', sans-serif; background-color: #f3f4f6; }
        .glass-card { background: rgba(255, 255, 255, 0.9); backdrop-filter: blur(10px); border-radius: 20px; }
        .nav-active { color: #3b82f6; }
        .tab-content { display: none; }
        .tab-content.active { display: block; }
    </style>
</head>
<body class="pb-24">

    <header class="p-6 bg-white shadow-sm sticky top-0 z-50">
        <div class="flex justify-between items-center">
            <div>
                <h1 class="text-2xl font-bold text-gray-800">Hi, 投資者</h1>
                <p class="text-sm text-gray-500">2026年1月11日 財務概況</p>
            </div>
            <div class="bg-blue-100 p-3 rounded-full text-blue-600">
                <i class="fas fa-user"></i>
            </div>
        </div>
    </header>

    <main class="p-4 max-w-md mx-auto">

        <div class="bg-gradient-to-br from-blue-600 to-indigo-700 p-6 rounded-3xl text-white shadow-lg mb-6">
            <p class="opacity-80 text-sm">總淨資產 (Net Worth)</p>
            <h2 class="text-3xl font-bold mt-1">$1,245,000</h2>
            <div class="flex mt-4 space-x-4 text-xs">
                <div class="bg-white/20 p-2 rounded-lg italic">
                    <i class="fas fa-arrow-up mr-1"></i> 月成長 4.2%
                </div>
                <div class="bg-white/20 p-2 rounded-lg">
                    通膨壓力：高
                </div>
            </div>
        </div>

        <div id="dashboard" class="tab-content active">
            <div class="glass-card p-5 shadow-sm mb-6 border border-gray-100">
                <h3 class="font-bold mb-4 flex items-center">
                    <i class="fas fa-chart-pie mr-2 text-blue-500"></i> 目前資產分配
                </h3>
                <canvas id="assetChart" width="400" height="250"></canvas>
                <div class="mt-4 grid grid-cols-2 gap-2 text-sm">
                    <div class="p-2 border rounded-lg">現金: $500k</div>
                    <div class="p-2 border rounded-lg">股票: $600k</div>
                    <div class="p-2 border rounded-lg">儲蓄: $145k</div>
                    <div class="p-2 border rounded-lg">虛擬貨幣: $0</div>
                </div>
            </div>

            <div class="bg-yellow-50 p-5 rounded-2xl border border-yellow-100 mb-6">
                <h3 class="font-bold text-yellow-800 flex items-center text-sm mb-2">
                    <i class="fas fa-lightbulb mr-2"></i> 智慧財務分析
                </h3>
                <p class="text-sm text-yellow-700 leading-relaxed">
                    目前的通膨率預計為 3%。若不進行投資，您的現金購買力將在 10 年後下降約 25%。建議增加 15% 的比例配置於抗通膨資產。
                </p>
            </div>
        </div>

        <div id="invest" class="tab-content">
            <div class="glass-card p-5 shadow-sm mb-6">
                <h3 class="font-bold mb-4">定期定額狀況</h3>
                <div class="flex justify-between items-center mb-4">
                    <span>每月定期定額總額</span>
                    <span class="font-bold text-blue-600">$15,000</span>
                </div>
                <div class="h-2 w-full bg-gray-100 rounded-full overflow-hidden">
                    <div class="h-full bg-blue-500" style="width: 65%"></div>
                </div>
                <p class="text-xs text-gray-400 mt-2">目前投資佔月收入 35%，表現優於平均。</p>
            </div>

            <div class="glass-card p-5 shadow-sm">
                <h3 class="font-bold mb-4">通膨損益估算 (未來20年)</h3>
                <canvas id="inflationChart" width="400" height="250"></canvas>
            </div>
        </div>

        <div id="goal" class="tab-content">
            <div class="glass-card p-5 shadow-sm mb-6">
                <h3 class="font-bold mb-2">2年累積 80萬 計劃</h3>
                <div class="flex justify-between mb-1">
                    <span class="text-xs">進度 45%</span>
                    <span class="text-xs">$360,000 / $800,000</span>
                </div>
                <div class="h-3 w-full bg-gray-100 rounded-full overflow-hidden mb-4">
                    <div class="h-full bg-green-500" style="width: 45%"></div>
                </div>
                <div class="bg-green-50 p-4 rounded-xl">
                    <h4 class="text-sm font-bold text-green-800 mb-2">執行策略建議：</h4>
                    <ul class="text-xs text-green-700 space-y-2">
                        <li>• 每月需存入：<strong>$18,333</strong></li>
                        <li>• 建議配置：60% 定存 / 40% 低風險 ETF</li>
                        <li>• 額外建議：若每年有 5% 投報率，每月僅需存 $16,500。</li>
                    </ul>
                </div>
            </div>
        </div>

    </main>

    <nav class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-100 flex justify-around p-4 z-50">
        <button onclick="showTab('dashboard')" class="nav-item nav-active flex flex-col items-center">
            <i class="fas fa-wallet text-xl"></i>
            <span class="text-[10px] mt-1">資產</span>
        </button>
        <button onclick="showTab('invest')" class="nav-item text-gray-400 flex flex-col items-center">
            <i class="fas fa-chart-line text-xl"></i>
            <span class="text-[10px] mt-1">分析</span>
        </button>
        <button onclick="showTab('goal')" class="nav-item text-gray-400 flex flex-col items-center">
            <i class="fas fa-bullseye text-xl"></i>
            <span class="text-[10px] mt-1">目標</span>
        </button>
        <button class="text-gray-400 flex flex-col items-center">
            <i class="fas fa-cog text-xl"></i>
            <span class="text-[10px] mt-1">設定</span>
        </button>
    </nav>

    <script>
        // 頁籤切換邏輯
        function showTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
            document.getElementById(tabId).classList.add('active');
            
            document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('nav-active', 'text-blue-500'));
            event.currentTarget.classList.add('nav-active');
        }

        // 資產分配圖表
        const ctx = document.getElementById('assetChart').getContext('2d');
        new Chart(ctx, {
            type: 'doughnut',
            data: {
                labels: ['現金', '股票', '儲蓄'],
                datasets: [{
                    data: [500000, 600000, 145000],
                    backgroundColor: ['#3b82f6', '#8b5cf6', '#ec4899'],
                    borderWidth: 0
                }]
            },
            options: {
                plugins: { legend: { position: 'bottom', labels: { boxWidth: 12 } } },
                cutout: '70%'
            }
        });

        // 通膨模擬圖表
        const ctx2 = document.getElementById('inflationChart').getContext('2d');
        new Chart(ctx2, {
            type: 'line',
            data: {
                labels: ['現在', '5年', '10年', '15年', '20年'],
                datasets: [
                    {
                        label: '名目金額',
                        data: [100, 100, 100, 100, 100],
                        borderColor: '#9ca3af',
                        fill: false
                    },
                    {
                        label: '實際購買力 (3%通膨)',
                        data: [100, 86, 74, 64, 55],
                        borderColor: '#ef4444',
                        backgroundColor: 'rgba(239, 68, 68, 0.1)',
                        fill: true
                    }
                ]
            }
        });
    </script>
</body>
</html>
