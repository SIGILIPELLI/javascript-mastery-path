# 10 · Project — Weather Dashboard

A capstone project combining everything from Level 2: closures and modules,
`async`/`await`, the Fetch API, custom error handling, and unit testing with
Jest.

## What you'll build

A browser-based weather dashboard that:

- Looks up a location's current conditions and a short forecast using the
  free [Open-Meteo](https://open-meteo.com/) API (no API key required)
- Renders current temperature, wind speed, and a multi-day forecast into the
  DOM
- Handles network/API failures gracefully, showing a readable error instead
  of a blank page
- Separates pure data-transformation logic into its own module so it can be
  unit tested with Jest, without making real network calls in tests

## Project layout

```text
weather-dashboard/
    index.html
    style.css
    app.js
    weather.js
    weather.test.js
    package.json
```

## index.html — page structure

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Weather Dashboard</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <h1>Weather Dashboard</h1>

    <form id="search-form">
      <input id="lat-input" placeholder="Latitude" value="51.5072" required />
      <input id="lon-input" placeholder="Longitude" value="-0.1276" required />
      <button type="submit">Get Weather</button>
    </form>

    <div id="status"></div>

    <section id="current" hidden>
      <h2>Current Conditions</h2>
      <p id="current-temp"></p>
      <p id="current-wind"></p>
    </section>

    <section id="forecast" hidden>
      <h2>Forecast</h2>
      <ul id="forecast-list"></ul>
    </section>

    <script type="module" src="app.js"></script>
  </body>
</html>
```

Latitude/longitude default to London so the form works immediately without
needing a geocoding step.

## style.css — minimal styling

```css
body {
  font-family: system-ui, sans-serif;
  max-width: 600px;
  margin: 2rem auto;
  padding: 0 1rem;
}

#status.error {
  color: #b00020;
  font-weight: bold;
}

#forecast-list {
  list-style: none;
  padding: 0;
}

#forecast-list li {
  display: flex;
  justify-content: space-between;
  padding: 0.4rem 0;
  border-bottom: 1px solid #ddd;
}
```

## weather.js — pure, testable data-handling module

This module has no DOM code and no `fetch` calls — it only transforms plain
JSON into a display-ready shape, which is what makes it easy to unit test.

```javascript
// weather.js

// Maps Open-Meteo's numeric weather codes to short human-readable labels.
const WEATHER_CODES = {
  0: "Clear sky",
  1: "Mainly clear",
  2: "Partly cloudy",
  3: "Overcast",
  45: "Fog",
  61: "Rain",
  71: "Snow",
  95: "Thunderstorm",
};

export function describeWeatherCode(code) {
  return WEATHER_CODES[code] ?? "Unknown conditions";
}

/**
 * Transforms raw Open-Meteo forecast JSON into a display-ready shape.
 * Throws a descriptive error if the shape isn't what we expect, instead of
 * letting a confusing TypeError happen deep inside the rendering code.
 */
export function toDisplayForecast(rawData) {
  if (!rawData || !rawData.current || !rawData.daily) {
    throw new Error("unexpected API response shape");
  }

  const current = {
    temperature: rawData.current.temperature_2m,
    windSpeed: rawData.current.wind_speed_10m,
    condition: describeWeatherCode(rawData.current.weather_code),
  };

  const daily = rawData.daily.time.map((date, index) => ({
    date,
    high: rawData.daily.temperature_2m_max[index],
    low: rawData.daily.temperature_2m_min[index],
    condition: describeWeatherCode(rawData.daily.weather_code[index]),
  }));

  return { current, daily };
}

export function buildForecastUrl(lat, lon) {
  const params = new URLSearchParams({
    latitude: lat,
    longitude: lon,
    current: "temperature_2m,wind_speed_10m,weather_code",
    daily: "temperature_2m_max,temperature_2m_min,weather_code",
    timezone: "auto",
  });

  return `https://api.open-meteo.com/v1/forecast?${params}`;
}
```

## app.js — fetching and rendering

```javascript
// app.js
import { toDisplayForecast, buildForecastUrl } from "./weather.js";

const form = document.getElementById("search-form");
const latInput = document.getElementById("lat-input");
const lonInput = document.getElementById("lon-input");
const status = document.getElementById("status");
const currentSection = document.getElementById("current");
const forecastSection = document.getElementById("forecast");
const currentTemp = document.getElementById("current-temp");
const currentWind = document.getElementById("current-wind");
const forecastList = document.getElementById("forecast-list");

async function loadWeather(lat, lon) {
  status.textContent = "Loading...";
  status.className = "";
  currentSection.hidden = true;
  forecastSection.hidden = true;

  try {
    const response = await fetch(buildForecastUrl(lat, lon));

    if (!response.ok) {
      throw new Error(`weather service responded with ${response.status}`);
    }

    const rawData = await response.json();
    const forecast = toDisplayForecast(rawData);

    render(forecast);
    status.textContent = "";
  } catch (error) {
    status.textContent = `Couldn't load weather: ${error.message}`;
    status.className = "error";
  }
}

function render(forecast) {
  currentTemp.textContent = `${forecast.current.temperature}°C, ${forecast.current.condition}`;
  currentWind.textContent = `Wind: ${forecast.current.windSpeed} km/h`;
  currentSection.hidden = false;

  forecastList.innerHTML = "";
  forecast.daily.forEach((day) => {
    const item = document.createElement("li");
    item.textContent = `${day.date}: ${day.low}°–${day.high}°C, ${day.condition}`;
    forecastList.appendChild(item);
  });
  forecastSection.hidden = false;
}

form.addEventListener("submit", (event) => {
  event.preventDefault();
  const lat = latInput.value.trim();
  const lon = lonInput.value.trim();
  if (!lat || !lon) return;
  loadWeather(lat, lon);
});

// Initial load using the default London coordinates already in the inputs
loadWeather(latInput.value, lonInput.value);
```

## weather.test.js — unit tests for the pure logic

These tests use a pre-canned JSON fixture — no real network calls, so the
suite is fast and deterministic. Only `weather.js`'s pure functions are
under test; `app.js`'s DOM code is intentionally left out of scope here.

```javascript
// weather.test.js
import { describe, test, expect } from "@jest/globals";
import { describeWeatherCode, toDisplayForecast, buildForecastUrl } from "./weather.js";

const sampleResponse = {
  current: {
    temperature_2m: 18.4,
    wind_speed_10m: 12.1,
    weather_code: 2,
  },
  daily: {
    time: ["2024-06-01", "2024-06-02"],
    temperature_2m_max: [21.0, 19.5],
    temperature_2m_min: [12.0, 11.2],
    weather_code: [1, 61],
  },
};

describe("describeWeatherCode", () => {
  test("maps known codes to labels", () => {
    expect(describeWeatherCode(0)).toBe("Clear sky");
    expect(describeWeatherCode(61)).toBe("Rain");
  });

  test("falls back for unknown codes", () => {
    expect(describeWeatherCode(9999)).toBe("Unknown conditions");
  });
});

describe("toDisplayForecast", () => {
  test("transforms current conditions correctly", () => {
    const result = toDisplayForecast(sampleResponse);

    expect(result.current).toEqual({
      temperature: 18.4,
      windSpeed: 12.1,
      condition: "Partly cloudy",
    });
  });

  test("transforms the daily forecast array correctly", () => {
    const result = toDisplayForecast(sampleResponse);

    expect(result.daily).toHaveLength(2);
    expect(result.daily[0]).toEqual({
      date: "2024-06-01",
      high: 21.0,
      low: 12.0,
      condition: "Mainly clear",
    });
    expect(result.daily[1].condition).toBe("Rain");
  });

  test("throws a descriptive error for malformed input", () => {
    expect(() => toDisplayForecast({})).toThrow("unexpected API response shape");
    expect(() => toDisplayForecast(null)).toThrow("unexpected API response shape");
  });
});

describe("buildForecastUrl", () => {
  test("includes the coordinates and requested fields", () => {
    const url = buildForecastUrl(51.5072, -0.1276);

    expect(url).toContain("latitude=51.5072");
    expect(url).toContain("longitude=-0.1276");
    expect(url).toContain("temperature_2m");
  });
});
```

## package.json — dependencies and scripts

```json
{
  "name": "weather-dashboard",
  "type": "module",
  "scripts": {
    "test": "jest",
    "serve": "npx serve ."
  },
  "devDependencies": {
    "jest": "^29.0.0"
  }
}
```

## Running it

```bash
npm install
npm test       # runs weather.test.js — no network access needed
npm run serve  # starts a local static server, then open the printed URL
```

In the browser:

- The default London coordinates load automatically on page open
- Try different latitude/longitude pairs (e.g. `40.7128, -74.0060` for New
  York) and click "Get Weather"
- Turn off your network connection and retry — confirm the status area
  shows a readable error instead of a blank or broken page

## Stretch goals

- Add a geocoding step: let users type a city name and look up its
  coordinates via Open-Meteo's free geocoding endpoint before fetching the
  forecast.
- Cache the last successful forecast in `localStorage` (as practiced in
  [Level 1's to-do app](../level-1/10-project-todo-app.md)) and show it
  immediately while a fresh request is in flight.
- Add a loading spinner instead of plain "Loading..." text, and disable the
  form's submit button while a request is pending.
- Extract `render()` into its own testable module too, using a lightweight
  DOM testing library, and add tests for it.
- Add units toggle (Celsius/Fahrenheit) as a pure conversion function in
  `weather.js`, with its own Jest tests.

Completing this project means you're ready for **Level 3 · Advanced**.
