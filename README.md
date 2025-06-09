<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Bitcoin - Preço e Histórico</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-chart-financial"></script>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <style>
    .up { color: green; }
    .down { color: red; }
  </style>
</head>
<body class="bg-gray-100 p-4 sm:p-8">
  <div class="max-w-5xl mx-auto">
    <!-- Cabeçalho -->
    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-6 gap-4">
      <h1 class="text-2xl font-bold">Bitcoin - Preço e Histórico</h1>
      <button id="refreshBtn" class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">Atualizar</button>
    </div>

    <!-- Seleções -->
    <div class="flex flex-col sm:flex-row gap-4 mb-6">
      <label class="flex flex-col sm:flex-row items-start sm:items-center">
        <span class="mb-1 sm:mb-0">Moeda:</span>
        <select id="currency" class="ml-0 sm:ml-2 p-2 border rounded w-full sm:w-auto">
          <option value="usd">USD</option>
          <option value="eur">EUR</option>
          <option value="brl">BRL</option>
          <option value="gbp">GBP</option>
          <option value="jpy">JPY</option>
        </select>
      </label>

      <label class="flex flex-col sm:flex-row items-start sm:items-center">
        <span class="mb-1 sm:mb-0">Dias atrás:</span>
        <select id="days" class="ml-0 sm:ml-2 p-2 border rounded w-full sm:w-auto">
          <option value="0.17">4 horas</option>
          <option value="0.5">12 horas</option>
          <option value="1">1 dia</option>
          <option value="7">7 dias</option>
          <option value="14">14 dias</option>
          <option value="30">30 dias</option>
          <option value="90">90 dias</option>
          <option value="180">180 dias</option>
          <option value="365">1 ano</option>
        </select>
      </label>
    </div>

    <!-- Informações -->
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-6 text-lg">
      <div>Preço atual: <span id="price" class="font-medium">Carregando...</span></div>
      <div>Variação 24h: <span id="change" class="font-medium">Carregando...</span></div>
      <div id="trend" class="font-medium">Indicador de Tendência: Carregando...</div>
      <div id="high" class="font-medium">Máximo 24h: Carregando...</div>
      <div id="low" class="font-medium">Mínimo 24h: Carregando...</div>
    </div>

    <!-- Gráfico -->
    <div class="w-full">
      <canvas id="btcChart" class="w-full h-64 sm:h-80 md:h-96"></canvas>
    </div>
  </div>

  <script>
    const ctx = document.getElementById('btcChart').getContext('2d');
    const currencySelect = document.getElementById('currency');
    const daysSelect = document.getElementById('days');
    const refreshBtn = document.getElementById('refreshBtn');

    const btcChart = new Chart(ctx, {
      type: 'line',
      data: {
        labels: [],
        datasets: [{
          label: 'Preço do Bitcoin',
          data: [],
          borderColor: 'rgba(54, 162, 235, 1)',
          fill: false
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          y: { beginAtZero: false }
        }
      }
    });

    async function fetchCurrentPrice(currency) {
      const res = await fetch(`https://api.coingecko.com/api/v3/coins/bitcoin?localization=false&tickers=false&market_data=true`);
      const data = await res.json();
      const price = data.market_data.current_price[currency];
      const change = data.market_data.price_change_percentage_24h;
      const high_24h = data.market_data.high_24h[currency];
      const low_24h = data.market_data.low_24h[currency];

      document.getElementById('price').innerText = `${price.toLocaleString()} ${currency.toUpperCase()}`;
      const changeEl = document.getElementById('change');
      changeEl.innerText = `${change.toFixed(2)}%`;
      changeEl.className = change >= 0 ? 'up' : 'down';

      const trendEl = document.getElementById('trend');
      trendEl.innerText = change >= 0 ? "Tendência de Alta" : "Tendência de Baixa";
      trendEl.style.color = change >= 0 ? 'green' : 'red';

      document.getElementById('high').innerText = `Máximo 24h: ${high_24h.toLocaleString()} ${currency.toUpperCase()}`;
      document.getElementById('low').innerText = `Mínimo 24h: ${low_24h.toLocaleString()} ${currency.toUpperCase()}`;
    }

    async function fetchHistoricalData(currency, days) {
      const res = await fetch(`https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=${currency}&days=${days}`);
      const data = await res.json();

      const prices = data.prices;
      const labels = prices.map(p => {
        const date = new Date(p[0]);
        return days <= 1 ? date.toLocaleTimeString() : date.toLocaleDateString();
      });

      const values = prices.map(p => p[1]);

      btcChart.data.labels = labels;
      btcChart.data.datasets[0].data = values;
      btcChart.data.datasets[0].label = `Preço do Bitcoin (${currency.toUpperCase()})`;
      btcChart.update();
    }

    async function updateAll() {
      const currency = currencySelect.value;
      const days = daysSelect.value;
      await fetchCurrentPrice(currency);
      await fetchHistoricalData(currency, days);
    }

    currencySelect.addEventListener('change', updateAll);
    daysSelect.addEventListener('change', updateAll);
    refreshBtn.addEventListener('click', updateAll);

    updateAll();
  </script>
</body>
</html>
