export default {
  async fetch(request, env, ctx) {
    const apiKey = await env.WEATHER_API.get("api_key");
    const cities = ['Bangkok', 'Jakarta'];

    try {
      const results = await Promise.all(
        cities.map(async (city) => {
          const currentRes = await fetch(`https://api.weatherapi.com/v1/current.json?key=${apiKey}&q=${city}&aqi=yes`);
          const forecastRes = await fetch(`https://api.weatherapi.com/v1/forecast.json?key=${apiKey}&q=${city}&days=5&aqi=yes&alerts=no`);

          if (!currentRes.ok || !forecastRes.ok) {
            throw new Error(`Failed to fetch weather data for ${city}`);
          }

          const current = await currentRes.json();
          const forecast = await forecastRes.json();
          return { city, current, forecast };
        })
      );

      function getAqiColor(pm25) {
        if (pm25 <= 12) return 'green';
        if (pm25 <= 35.4) return 'orange';
        if (pm25 <= 55.4) return 'red';
        return 'darkred';
      }

      function getAqiEmoji(pm25) {
        if (pm25 <= 12) return '😊';
        if (pm25 <= 35.4) return '😐';
        if (pm25 <= 55.4) return '😷';
        return '☠️';
      }

      function renderCurrentCard(cityName, data) {
        const {
          last_updated,
          temp_c,
          feelslike_c,
          condition,
          humidity,
          wind_kph,
          air_quality,
        } = data.current;

        return `
          <div class="card">
            <h2>${cityName}</h2>
            <p><strong>Condition:</strong> ${condition.text} <img src="https:${condition.icon}" alt="icon" /></p>
            <p><strong>Temperature:</strong> ${temp_c}°C</p>
            <p><strong>Feels like:</strong> ${feelslike_c}°C</p>
            <p><strong>Humidity:</strong> ${humidity}%</p>
            <p><strong>Wind:</strong> ${wind_kph} km/h</p>
            <p><strong>Air Quality:</strong></p>
            <ul style="list-style: none; padding: 0;">
              <li>
                PM2.5: 
                <span style="font-weight: bold; color: ${getAqiColor(air_quality.pm2_5)};">
                  ${air_quality.pm2_5.toFixed(1)} µg/m³ ${getAqiEmoji(air_quality.pm2_5)}
                </span>
              </li>
              <li>PM10: ${air_quality.pm10.toFixed(1)} µg/m³</li>
            </ul>
            <p><small>Last Updated: ${last_updated}</small></p>
          </div>
        `;
      }

      function renderForecastCards(forecastData) {
        const days = forecastData.forecast.forecastday;
        return days.map(day => {
          const date = day.date;
          const condition = day.day.condition.text;
          const icon = day.day.condition.icon;
          const maxTemp = day.day.maxtemp_c;
          const minTemp = day.day.mintemp_c;

          return `
            <div class="forecast-card">
              <p><strong>${date}</strong></p>
              <img src="https:${icon}" alt="${condition}" />
              <p>${condition}</p>
              <p>🌡️ ${minTemp}–${maxTemp}°C</p>
            </div>
          `;
        }).join('');
      }

      const bangkok = results.find(r => r.city === 'Bangkok');
      const jakarta = results.find(r => r.city === 'Jakarta');

      const html = `
        <!DOCTYPE html>
        <html>
        <head>
          <meta charset="UTF-8" />
          <title>Weather Dashboard</title>
          <style>
            body {
              font-family: Arial, sans-serif;
              background: #f4f9ff;
              margin: 0;
              padding: 20px;
              text-align: center;
            }
            .toggle-button {
              font-size: 20px;
              margin-bottom: 20px;
              cursor: pointer;
              background: #007bff;
              color: white;
              border: none;
              border-radius: 8px;
              padding: 8px 16px;
            }

            .section {
              margin-bottom: 40px;
            }

            .card, .forecast {
              background: white;
              padding: 20px;
              border-radius: 12px;
              box-shadow: 0 4px 12px rgba(0,0,0,0.1);
              margin-bottom: 20px;
            }

            .forecast-row {
              display: flex;
              justify-content: center;
              gap: 10px;
              flex-wrap: wrap;
              margin-top: 10px;
            }

            .forecast-card {
              background: #e9f5ff;
              padding: 10px;
              border-radius: 8px;
              width: 120px;
            }

            img {
              vertical-align: middle;
              max-width: 64px;
            }

            .mobile .city-row {
              display: block;
            }

            .desktop .city-row {
              display: flex;
              justify-content: center;
              gap: 30px;
              align-items: flex-start;
            }

            .desktop .forecast {
              max-width: 640px;
            }
          </style>
        </head>
        <body class="mobile">
          <button class="toggle-button" onclick="toggleView()">📱 Mobile View</button>
          <h1>🌤️ Bangkok & Jakarta Weather</h1>

          <div class="section city-row">
            ${renderCurrentCard('Bangkok', bangkok.current)}
            <div class="forecast">
              <h3>5-Day Forecast (Bangkok)</h3>
              <div class="forecast-row">
                ${renderForecastCards(bangkok.forecast)}
              </div>
            </div>
          </div>

          <div class="section city-row">
            ${renderCurrentCard('Jakarta', jakarta.current)}
            <div class="forecast">
              <h3>5-Day Forecast (Jakarta)</h3>
              <div class="forecast-row">
                ${renderForecastCards(jakarta.forecast)}
              </div>
            </div>
          </div>

          <script>
            function toggleView() {
              const body = document.body;
              const button = document.querySelector('.toggle-button');
              if (body.classList.contains('mobile')) {
                body.classList.remove('mobile');
                body.classList.add('desktop');
                button.textContent = '💻 Desktop View';
              } else {
                body.classList.remove('desktop');
                body.classList.add('mobile');
                button.textContent = '📱 Mobile View';
              }
            }
          </script>
        </body>
        </html>
      `;

      return new Response(html, {
        headers: { 'Content-Type': 'text/html; charset=UTF-8' },
      });

    } catch (err) {
      return new Response(`Error: ${err.message}`, { status: 500 });
    }
  }
};
