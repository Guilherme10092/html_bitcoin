<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Bitcoin - Preço e Histórico</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: Arial, sans-serif; padding: 20px; }
    .info { margin-bottom: 20px; font-size: 18px; }
    .up { color: green; }
    .down { color: red; }
    label { margin-right: 10px; }
  </style>
</head>
<body>
  <div class="info">
    <label>Moeda:
      <select id="currency">
        <option value="usd">USD</option>
        <option value="eur">EUR</option>
        <option value="brl">BRL</option>
        <option value="gbp">GBP</option>
        <option value="jpy">JPY</option>
      </select>
    </label>

    <label>Dias atrás:
      <select id="days">
        <option value="1">1 dia</option>
        <option value="7">7 dias</option>
        <option value="14">14 dias</option>
        <option value="30">30 dias</option>
        <option value="90">90 dias</option>
        <option value="180">180 dias</option>
        <option value="365">1 ano</option>
      </select>
    </label>

    <div>Preço atual: <span id="price">Carregando...</span></div>
    <div>Variação 24h: <span id="change">Carregando...</span></div>
  </div>

  <canvas id="btcChart" width="800" height="400"></canvas>

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
          y: {
            beginAtZero: false
          }
        }
      }
    });

    async function fetchCurrentPrice(currency) {
      const res = await fetch(`https://api.coingecko.com/api/v3/coins/bitcoin?localization=false&tickers=false&market_data=true`);
      const data = await res.json();
      const price = data.market_data.current_price[currency];
      const change = data.market_data.price_change_percentage_24h;

      document.getElementById('price').innerText = `${price.toLocaleString()} ${currency.toUpperCase()}`;
      const changeEl = document.getElementById('change');
      changeEl.innerText = `${change.toFixed(2)}%`;
      changeEl.className = change >= 0 ? 'up' : 'down';
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
