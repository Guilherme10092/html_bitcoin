<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Bitcoin - Preço e Histórico</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-chart-financial"></script>
  <script src="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/lib/index.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/lib/index.css" rel="stylesheet">
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      background-color: #f3f4f6;
    }
    .info {
      margin-bottom: 20px;
      font-size: 18px;
    }
    .up { color: green; }
    .down { color: red; }
    label { margin-right: 10px; }
    .section { margin-bottom: 20px; }
  </style>
</head>
<body class="bg-gray-100 p-8">
  <div class="container mx-auto">
    <div class="flex justify-between mb-4">
      <div class="text-xl font-semibold">Bitcoin - Preço e Histórico</div>
    </div>

    <div class="section flex gap-6">
      <label class="flex items-center">
        Moeda:
        <select id="currency" class="ml-2 p-2 border rounded">
          <option value="usd">USD</option>
          <option value="eur">EUR</option>
          <option value="brl">BRL</option>
          <option value="gbp">GBP</option>
          <option value="jpy">JPY</option>
        </select>
      </label>

      <label class="flex items-center">
        Dias atrás:
        <select id="days" class="ml-2 p-2 border rounded">
          <option value="0.17">4 horas</option> <!-- Nova opção -->
          <option value="0.5">12 horas</option> <!-- Nova opção -->
          <option value="1">1 dia</option>
          <option value="7">7 dias</option>
          <option value="14">14 dias</option>
          <option value="30">30 dias</option>
          <option value="90">90 dias</option>
          <option value="180">180 dias</option>
          <option value="365">1 ano</option>
          <option value="0.17">4 horas</option> <!-- Nova opção -->
          <option value="0.5">12 horas</option> <!-- Nova opção -->
        </select>
      </label>
    </div>

    <div class="section">
      <div>Preço atual: <span id="price">Carregando...</span></div>
      <div>Variação 24h: <span id="change">Carregando...</span></div>
      <div id="trend">Indicador de Tendência: Carregando...</div>
      <div id="high">Máximo 24h: Carregando...</div>
      <div id="low">Mínimo 24h: Carregando...</div>
    </div>

    <canvas id="btcChart" width="800" height="400"></canvas>
  </div>

  <script>
    const ctx = document.getElementById('btcChart').getContext('2d');
    const currencySelect = document.getElementById('currency');
    const daysSelect = document.getElementById('days');

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

    updateAll();
  </script>
</body>
</html>

