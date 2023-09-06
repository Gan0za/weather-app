<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';
import { apiKey, dictionary } from '../../config';

const success_ref = ref({});
const navigator_error = ref(null);
const weather_ref = ref({
  current: {
    feelslike_c: null,
    condition: { text: null, icon: null },
    humidity: null,
    uv: null,
    vis_km: null,
    wind_dir: null,
    wind_kph: null,
    cloud: null,
    pressure_mb:null
  },
  forecast: {
    forecastday: [
      {
        astro: {
          sunrise: null,
          sunset: null
        },
        date: null,
        day: {
          condition: { text: null, icon: '../assets/sunny.png' },
          maxtemp_c: null,
          mintemp_c: null
        },
        hour: [
          {
            condition: { text: null, icon: null },
            temp_c: null,
            time: null,
            time_epoch: null
          }
        ]
      }
    ]
  }
});
const systemLang = ref('en');
const region = ref({
  status: null, // error if status is fail and message is not null
  message: null,
  country: null,
  countryCode: null,
  region: null,
  regionName: null,
  city: null,
  zip: null,
  lat: null,
  lon: null,
  timezone: null,
  query: null
});
const localDictionary = ref({
  temp: "Temperature",
  like: "Feels like",
  description: "Description",
  humidity: "Humidity",
  pressure: "Pressure",
  city: "City",
  visibility: "Visibility",
  wind: "Wind speed",
  cloud: "Cloud cover",
  sunrise: "Sunrise",
  sunset: "Sunset"
});

const getWeatherLocal = (latitude, longitude) => {
  axios
    .get('https://api.weatherapi.com/v1/forecast.json?' +
      '&q=' + latitude + ',' + longitude +
      '&key=' + apiKey +
      '&days=8' +
      '&lang=' + systemLang.value)
    .then(response => {
      weather_ref.value = response.data;
      // console.log(weather_ref.value);
    })
    .catch(error => {
      console.log(error);
    });
};

const getRegion = (success) => {
  axios
    .get('http://ip-api.com/json/?fields=57855' +
      '&lang=' + systemLang.value)
    .then(response => {
      region.value = response.data;
      if (success && region.value.status != "fail") {
        getWeatherCity(region.value.city);
      } else if (success && region.value.status == "fail") {
        getWeatherCity("London");
      }
    })
    .catch(error => {
      console.log(error);
    });
};

const getWeatherCity = (city) => {
  axios
    .get('https://api.weatherapi.com/v1/forecast.json?' +
      '&q=' + city +
      '&key=' + apiKey +
      '&days=8' +
      '&lang=' + systemLang.value)
    .then(response => {
      weather_ref.value = response.data;
      // console.log(weather_ref.value);
    })
    .catch(error => {
      console.log(error);
    })
};

const getDatefromUnis = (time) => {
  let date = new Date(time * 1000);
  let hours = date.getHours();
  let minutes = "0" + date.getMinutes();
  let seconds = "0" + date.getSeconds();
  return hours + ':' + minutes.substr(-2); // + ':' + seconds.substr(-2);
}

const getDictionary = () => {
  let lang = systemLang.value;
  if (dictionary.lang[lang] == null) {
    localDictionary.value = dictionary.lang.en;
  } else {
    localDictionary.value = dictionary.lang[lang];
  }
}

onMounted(async () => {
  navigator.geolocation.getCurrentPosition(
    function success({ coords }) {
      const { latitude, longitude } = coords
      success_ref.value = { latitude, longitude };
      getWeatherLocal(latitude, longitude);
      getRegion(false)
    }, function error({ message }) {
      console.log(message);
      navigator_error.value = message;
      getRegion(true);
    },
    { enableHighAccuracy: true }
  );

  systemLang.value = (navigator.language || navigator.systemLanguage || navigator.userLanguage).substr(0, 2).toLowerCase();
  getDictionary();
});
</script>

<template>
  <header>
    <h1>Weather App</h1>
  </header>

  <main>
    <section class="weather-section">
      <div class="weather-card">
        <h2>Today's Weather</h2>
        <div class="weather-info">
          <div>
            <img class="weather-img" 
            :src="weather_ref.forecast.forecastday[0].day.condition.icon?.replace('64x64', '128x128')"
              :alt="weather_ref.forecast.forecastday[0].day.condition.text">
            <p>Now:</p>
            <h1>{{ weather_ref.current.feelslike_c }}°C</h1>
            <p>{{ weather_ref.current.condition.text }}</p>
          </div>
          <div style="
          display: flex;
					flex-direction: column;
					justify-content: center;
					align-items: center;">
            <h2>{{ region.city }}</h2>
            <p>Day: {{weather_ref.forecast.forecastday[0].day.maxtemp_c}}°C</p>
            <p>Night: {{weather_ref.forecast.forecastday[0].day.mintemp_c}}°C</p>
            <p>{{ weather_ref.forecast.forecastday[0].day.condition.text }}</p>
          </div>
        </div>
      </div>
      <div class="weather-block-info">
        <div class="weather-card">
          <h3>UV-index</h3>
          <div class="card-info">
            <h1>{{ weather_ref.current.uv }}</h1>
          </div>
        </div>
        <div class="weather-card">
          <h3>Wind</h3>
          <div class="card-info">
            <div>
              <div style="display: flex;">
                <h1>{{ weather_ref.current.wind_kph }}</h1>
                <p>km/h</p>
              </div>
              <p>{{ weather_ref.current.wind_dir }}</p>
            </div>
          </div>
        </div>
        <div class="weather-card">
          <h3>Humidity</h3>
          <div class="card-info">
            <h1>{{ weather_ref.current.humidity }}</h1>
          </div>
        </div>
        <div class="weather-card">
          <h3>Visibility</h3>
          <div class="card-info">
            <h1>{{ weather_ref.current.vis_km }}</h1>
            <p>km</p>
          </div>
        </div>
        <div class="weather-card">
          <h3>Cloud</h3>
          <div class="card-info">
            <h1>{{ weather_ref.current.cloud }}</h1>
            <p>%</p>
          </div>
        </div>
        <div class="weather-card">
          <h3>Pressure</h3>
          <div class="card-info">
            <h1>{{ weather_ref.current.pressure_mb }}</h1>
            <p>mB</p>
          </div>
        </div>
        <div class="weather-card">
          <h3>Sunrise & Sunset</h3>
          <div class="cards-inf">
            <div class="card-info">
              <img src="../assets/sunny.png" alt="Sunrise">
              <p>{{ weather_ref.forecast.forecastday[0].astro.sunrise }}</p>
            </div>
            <div class="card-info">
              <img src="../assets/night.png" alt="Sunset">
              <p>{{ weather_ref.forecast.forecastday[0].astro.sunset }}</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section class="forecast-section">
      <h2>Hourly Forecast</h2>
      <div class="forecast-cards">
        <div class="forecast-card"
        v-for="item in weather_ref.forecast.forecastday[0].hour">
          <p>{{getDatefromUnis(item.time_epoch)}}</p>
          <img :src="item.condition.icon" 
          :alt="item.condition.text">
          <p>Temp: {{item.temp_c}}°C</p>
          <p>{{item.condition.text}}</p>
        </div>
      </div>
    </section>

    <section class="forecast-section">
      <h2>7-Day Forecast</h2>
      <div class="forecast-cards">
        <div class="forecast-card"
        v-for="item in weather_ref.forecast.forecastday">
          <p>{{item.date}}</p>
          <img :src="item.day.condition.icon" 
          :alt="item.day.condition.text">
          <p>Day: {{item.day.maxtemp_c}}°C</p>
          <p>Night: {{item.day.mintemp_c}}°C</p>
          <p>{{item.day.condition.text}}</p>
        </div>
      </div>
    </section>
  </main>

  <footer>
    <p>&copy; 2021 Weather App. All rights reserved.</p>
    <a href="https://www.weatherapi.com/" title="Free Weather API">
      <img src='https://cdn.weatherapi.com/v4/images/weatherapi_logo.png' alt="Weather data by WeatherAPI.com" border="0"
        style="width: 104px;height: 50px">
    </a>
  </footer>
</template>
