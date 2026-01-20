<!DOCTYPE html>
<html>
<head>
    <title>Greenguard Dashboard</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { font-family: sans-serif; background-color: #f4f4f4; text-align: center; }
        .container { padding: 20px; }
        canvas { max-width: 100%; margin-top: 20px; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🌱 Greenguard Live</h1>
        <p>Current Temperature: <strong id="temp-display">--°C</strong></p>
        <canvas id="tempChart"></canvas>
    </div>

    <script>
        // Tell Telegram the app is ready
        const tele = window.Telegram.WebApp;
        tele.ready();

        // Sample Data for the Chart
        const ctx = document.getElementById('tempChart').getContext('2d');
        new Chart(ctx, {
            type: 'line',
            data: {
                labels: ['9am', '10am', '11am', '12pm', '1pm'],
                datasets: [{
                    label: 'Temperature (°C)',
                    data: [18, 19, 22, 25, 24],
                    borderColor: '#2ecc71',
                    fill: false
                }]
            }
        });
    </script>
</body>
</html>
