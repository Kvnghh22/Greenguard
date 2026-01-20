<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Greenguard Live</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f0f2f5; color: #333; margin: 0; padding: 20px; text-align: center; }
        .card { background: white; border-radius: 15px; padding: 20px; margin-bottom: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .temp-val { font-size: 3rem; font-weight: bold; color: #2ecc71; }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .stat { background: #e8f5e9; padding: 10px; border-radius: 10px; }
        .weather-info { color: #555; font-style: italic; }
    </style>
</head>
<body>
    <div class="card">
        <h1>🌱 Greenguard Live</h1>
        <p class="weather-info" id="location-time">Preston, UK</p>
    </div>

    <div class="grid">
        <div class="card">
            <p>Farm Temp</p>
            <div class="temp-val" id="farm-temp">--°C</div>
        </div>
        <div class="card">
            <p>Battery</p>
            <div class="temp-val" id="battery-val">--%</div>
        </div>
    </div>

    <div class="card">
        <h3>Preston Weather Forecast</h3>
        <p id="weather-desc">Fetching forecast...</p>
        <div class="grid">
            <div class="stat">Outside: <span id="outside-temp">--°C</span></div>
            <div class="stat">Rain Prob: <span id="rain-prob">--%</span></div>
        </div>
    </div>

    <div class="card">
        <canvas id="tempChart"></canvas>
    </div>

    <script>
        const tele = window.Telegram.WebApp;
        tele.ready();
        tele.expand(); // Make the app full screen

        // YOUR API KEY
        const apiKey = "528c0f0810576f7f5fc7ccb5deef5b65";
        const city = "Preston,GB";

        async function fetchWeather() {
            try {
                const response = await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${city}&units=metric&appid=${apiKey}`);
                const data = await response.json();
                document.getElementById('outside-temp').innerText = Math.round(data.main.temp) + "°C";
                document.getElementById('weather-desc').innerText = data.weather[0].description;
                
                const forecastRes = await fetch(`https://api.openweathermap.org/data/2.5/forecast?q=${city}&units=metric&appid=${apiKey}`);
                const forecastData = await forecastRes.json();
                document.getElementById('rain-prob').innerText = Math.round(forecastData.list[0].pop * 100) + "%";
            } catch (e) {
                console.error("Weather Error", e);
            }
        }

        // Initialize Chart
        const ctx = document.getElementById('tempChart').getContext('2d');
        const tempChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: ['Start', 'Now'],
                datasets: [{ label: 'Farm Temp', data: [20, 20], borderColor: '#2ecc71', tension: 0.1 }]
            }
        });

        fetchWeather();
    </script>
</body>
</html>
