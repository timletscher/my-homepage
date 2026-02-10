# Tech Brief: Adding Open-Meteo Weather Widget to Vercel Website

## Overview
This brief outlines the integration of a weather widget using the Open-Meteo API into your existing Vercel-hosted website. Open-Meteo provides free weather data without requiring API keys, making it an excellent choice for simple weather displays.

## Technical Requirements

### Prerequisites
- Existing website hosted on Vercel
- Basic HTML/CSS/JavaScript knowledge
- Modern web browser support (ES6+)

### Dependencies
- Open-Meteo API (free, no authentication required)
- Fetch API or XMLHttpRequest for API calls
- Optional: CSS framework for styling

### API Specifications
- **Base URL**: `https://api.open-meteo.com/v1/forecast`
- **Rate Limits**: 10,000 requests per day, 5,000 per hour
- **Data Format**: JSON
- **Geolocation**: Requires latitude/longitude coordinates

## Implementation Approach
1. Create a weather widget component
2. Implement geolocation or manual location input
3. Fetch weather data from Open-Meteo API
4. Display weather information with responsive design
5. Deploy updates to Vercel

---

# Step-by-Step Implementation Plan

## Step 1: Plan Your Widget Design
**Duration: 15 minutes**

1. Decide on widget placement on your website
2. Choose what weather data to display:
   - Current temperature
   - Weather condition/icon
   - Humidity, wind speed
   - 5-day forecast (optional)
3. Sketch or wireframe the widget layout

## Step 2: Create the HTML Structure
**Duration: 10 minutes**

Add this HTML to your webpage where you want the widget:

```html
<div id="weather-widget" class="weather-widget">
  <div class="weather-header">
    <h3>Weather</h3>
    <button id="location-btn" class="location-btn">📍 Get Location</button>
  </div>
  <div id="weather-content" class="weather-content">
    <div class="loading">Loading weather data...</div>
  </div>
</div>
```

## Step 3: Add CSS Styling
**Duration: 20 minutes**

Create or add to your CSS file:

```css
.weather-widget {
  max-width: 300px;
  background: linear-gradient(135deg, #74b9ff, #0984e3);
  border-radius: 12px;
  padding: 20px;
  color: white;
  font-family: Arial, sans-serif;
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.weather-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.weather-header h3 {
  margin: 0;
  font-size: 1.2em;
}

.location-btn {
  background: rgba(255,255,255,0.2);
  border: none;
  color: white;
  padding: 5px 10px;
  border-radius: 15px;
  cursor: pointer;
  font-size: 0.8em;
}

.location-btn:hover {
  background: rgba(255,255,255,0.3);
}

.weather-content {
  text-align: center;
}

.current-weather {
  margin-bottom: 15px;
}

.temperature {
  font-size: 2.5em;
  font-weight: bold;
  margin: 10px 0;
}

.condition {
  font-size: 1.1em;
  margin-bottom: 10px;
}

.weather-details {
  display: flex;
  justify-content: space-around;
  font-size: 0.9em;
}

.loading, .error {
  padding: 20px;
  text-align: center;
}

.error {
  color: #ff6b6b;
}
```

## Step 4: Implement JavaScript Functionality
**Duration: 30 minutes**

Create or add to your JavaScript file:

```javascript
class WeatherWidget {
  constructor() {
    this.widgetElement = document.getElementById('weather-widget');
    this.contentElement = document.getElementById('weather-content');
    this.locationBtn = document.getElementById('location-btn');
    
    this.init();
  }

  init() {
    this.locationBtn.addEventListener('click', () => this.getUserLocation());
    // Try to load weather for a default location (e.g., Seattle)
    this.loadWeather(47.6062, -122.3321);
  }

  async getUserLocation() {
    if (!navigator.geolocation) {
      this.showError('Geolocation is not supported by this browser.');
      return;
    }

    this.showLoading();
    
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        this.loadWeather(latitude, longitude);
      },
      (error) => {
        this.showError('Unable to retrieve your location.');
        console.error('Geolocation error:', error);
      }
    );
  }

  async loadWeather(lat, lon) {
    try {
      this.showLoading();
      
      const response = await fetch(
        `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true&hourly=temperature_2m,relative_humidity_2m,wind_speed_10m&timezone=auto`
      );
      
      if (!response.ok) {
        throw new Error('Weather data not available');
      }
      
      const data = await response.json();
      this.displayWeather(data);
      
    } catch (error) {
      this.showError('Failed to load weather data.');
      console.error('Weather API error:', error);
    }
  }

  displayWeather(data) {
    const { current_weather } = data;
    const temp = Math.round(current_weather.temperature);
    const windSpeed = Math.round(current_weather.windspeed);
    
    // Get weather condition based on WMO code
    const condition = this.getWeatherCondition(current_weather.weathercode);
    
    this.contentElement.innerHTML = `
      <div class="current-weather">
        <div class="condition">${condition.description}</div>
        <div class="temperature">${temp}°C</div>
        <div class="weather-details">
          <div>
            <div>💨 Wind</div>
            <div>${windSpeed} km/h</div>
          </div>
          <div>
            <div>🕐 Time</div>
            <div>${new Date(current_weather.time).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}</div>
          </div>
        </div>
      </div>
    `;
  }

  getWeatherCondition(code) {
    const conditions = {
      0: { description: '☀️ Clear Sky' },
      1: { description: '🌤️ Mainly Clear' },
      2: { description: '⛅ Partly Cloudy' },
      3: { description: '☁️ Overcast' },
      45: { description: '🌫️ Foggy' },
      48: { description: '🌫️ Rime Fog' },
      51: { description: '🌦️ Light Drizzle' },
      53: { description: '🌦️ Drizzle' },
      55: { description: '🌦️ Heavy Drizzle' },
      61: { description: '🌧️ Light Rain' },
      63: { description: '🌧️ Rain' },
      65: { description: '🌧️ Heavy Rain' },
      71: { description: '🌨️ Light Snow' },
      73: { description: '🌨️ Snow' },
      75: { description: '🌨️ Heavy Snow' },
      77: { description: '🌨️ Snow Grains' },
      80: { description: '🌦️ Light Showers' },
      81: { description: '🌦️ Showers' },
      82: { description: '🌦️ Heavy Showers' },
      85: { description: '🌨️ Light Snow Showers' },
      86: { description: '🌨️ Snow Showers' },
      95: { description: '⛈️ Thunderstorm' },
      96: { description: '⛈️ Thunderstorm with Hail' },
      99: { description: '⛈️ Heavy Thunderstorm' }
    };
    
    return conditions[code] || { description: '🌤️ Unknown' };
  }

  showLoading() {
    this.contentElement.innerHTML = '<div class="loading">Loading weather data...</div>';
  }

  showError(message) {
    this.contentElement.innerHTML = `<div class="error">${message}</div>`;
  }
}

// Initialize the weather widget when the page loads
document.addEventListener('DOMContentLoaded', () => {
  new WeatherWidget();
});
```

## Step 5: Test Locally
**Duration: 10 minutes**

1. Open your website in a browser
2. Test the location button functionality
3. Verify weather data displays correctly
4. Check responsive design on mobile devices
5. Test error handling (disable internet, block location)

## Step 6: Deploy to Vercel
**Duration: 5 minutes**

1. Commit your changes to your repository:
   ```bash
   git add .
   git commit -m "Add Open-Meteo weather widget"
   git push
   ```

2. Vercel will automatically deploy your changes
3. Monitor the deployment in your Vercel dashboard
4. Test the live version once deployed

## Step 7: Optional Enhancements
**Duration: Variable**

Consider adding these features later:
- 5-day forecast display
- Location search by city name
- Temperature unit toggle (°C/°F)
- Weather alerts
- Local storage for user preferences
- Dark/light theme toggle

## Troubleshooting Tips

- **CORS Issues**: Open-Meteo supports CORS, but if you encounter issues, ensure you're making requests from your domain
- **Geolocation**: Some browsers require HTTPS for geolocation to work
- **API Limits**: Monitor your usage; consider caching responses for frequently visited pages
- **Mobile Testing**: Test thoroughly on mobile devices for responsive design

## Maintenance
- Monitor API usage in browser dev tools
- Update weather condition codes if Open-Meteo adds new ones
- Consider adding error logging for production debugging

This implementation provides a clean, functional weather widget that integrates seamlessly with your Vercel-hosted website while maintaining good performance and user experience.