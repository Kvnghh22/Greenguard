
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Greenguard Live</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background-color: #f0f2f5; margin: 0; padding: 20px; text-align: center; }
        .card { background: white; border-radius: 15px; padding: 20px; margin-bottom: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .temp-val { font-size: 3rem; font-weight: bold; color: #2ecc71; }
        .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .stat-box { background: #e8f5e9; padding: 10px; border-radius: 10px; font-weight: bold; }
    </style>
</head>
<body>
    <div class="card">
        <h1>🌱 Greenguard Live</h1>
        <p>Preston, UK</p>
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
        <h3>Outside Weather (Preston)</h3>
        <p id="weather-desc">Loading...</p>
        <div class="grid">
            <div class="stat-box">Outside: <span id="out-temp">--</span>°C</div>
            <div class="stat-box">Rain: <span id="rain">--</span>%</div>
        </div>
    </div>

    <script>
        const tele = window.Telegram.WebApp;
        tele.ready();
        tele.expand();

        // --- REPLACE THESE WITH YOUR THINGSPEAK INFO ---
        const channelID = "YOUR_CHANNEL_ID";
        const readKey = "YOUR_READ_API_KEY";
        const weatherKey = "528c0f0810576f7f5fc7ccb5deef5b65";

        async function updateDashboard() {
            // 1. Fetch from ThingSpeak (Your ESP32 Data)
            try {
                const tsRes = await fetch(`https://api.thingspeak.com/channels/${channelID}/feeds/last.json?api_key=${readKey}`);
                const tsData = await tsRes.json();
                document.getElementById('farm-temp').innerText = parseFloat(tsData.field1).toFixed(1) + "°C";
                document.getElementById('battery-val').innerText = tsData.field2 + "%";
            } catch(e) { console.log("ThingSpeak not ready yet"); }

            // 2. Fetch Outside Weather
            try {
                const wRes = await fetch(`https://api.openweathermap.org/data/2.5/forecast?q=Preston,GB&units=metric&appid=${weatherKey}`);
                const wData = await wRes.json();
                document.getElementById('out-temp').innerText = Math.round(wData.list[0].main.temp);
                document.getElementById('rain').innerText = Math.round(wData.list[0].pop * 100);
                document.getElementById('weather-desc').innerText = wData.list[0].weather[0].description;
            } catch(e) { console.log("Weather error"); }
        }

        setInterval(updateDashboard, 15000); // Update every 15 seconds
        updateDashboard();
    </script>
</body>
</html>
