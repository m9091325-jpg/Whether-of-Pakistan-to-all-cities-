# Wheather-of-Pakistan-to-all-cities-
<!DOCTYPE html>
<html lang="ur" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>پاکستانی شہروں کا موسم</title>
  <style>
    body {
      font-family: 'Noto Nastaliq Urdu', sans-serif;
      background-color: #f0f4f8;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      direction: rtl;
    }
    .container {
      background: white;
      padding: 20px 30px;
      border-radius: 12px;
      box-shadow: 0 8px 20px rgba(0,0,0,0.1);
      text-align: center;
      width: 90%;
      max-width: 400px;
    }
    h1 {
      margin-bottom: 20px;
    }
    input {
      padding: 10px;
      width: 100%;
      margin-bottom: 10px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 16px;
    }
    button {
      padding: 10px 20px;
      background-color: #0077ff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 16px;
    }
    button:hover {
      background-color: #005bb5;
    }
    #result {
      margin-top: 20px;
      text-align: right;
    }
    @media (max-width: 480px) {
      .container { padding: 15px; }
      button { width: 100%; }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>موسم معلوم کریں (پاکستان)</h1>
    <input type="text" id="cityInput" placeholder="شہر کا نام لکھیں جیسے 'Lahore'" />
    <button id="getWeatherBtn">موسم دکھائیں</button>
    <div id="result"></div>
  </div>

  <script>
    async function getWeather() {
      const city = document.getElementById('cityInput').value.trim();
      if (!city) return alert('براہ کرم شہر کا نام درج کریں۔');

      const apiKey = 'YOUR_API_KEY_HERE'; // ← یہاں اپنی API Key لگائیں
      const url = `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(city)},pk&units=metric&lang=ur&appid=${apiKey}`;

      try {
        const res = await fetch(url);
        const data = await res.json();

        if (data.cod === 200) {
          document.getElementById('result').innerHTML = `
            <h2>${data.name} (${data.sys.country})</h2>
            <p>🌤️ موسم: ${data.weather[0].description}</p>
            <p>🌡️ درجہ حرارت: ${data.main.temp}°C</p>
            <p>💧 نمی: ${data.main.humidity}%</p>
            <p>💨 ہوائیں: ${data.wind.speed} m/s</p>
          `;
        } else {
          document.getElementById('result').innerHTML = `<p>❌ شہر نہیں ملا۔ دوبارہ کوشش کریں۔</p>`;
        }
      } catch (error) {
        console.error(error);
        document.getElementById('result').innerHTML = `<p>⚠️ کچھ غلط ہو گیا ہے۔ بعد میں دوبارہ کوشش کریں۔</p>`;
      }
    }

    document.getElementById('getWeatherBtn').addEventListener('click', getWeather);
  </script>
</body>
</html>
