<template>
    <div class="flex flex-col flex-1 items-center">
      <!-- Banner -->
      <div
        v-if="route.query.preview"
        class="text-white p-4 bg-weather-secondary w-full text-center"
      >
        <p>
          You are currently previewing this city, click the "+"
          icon to start tracking this city.
        </p>
      </div>
      <!-- Weather Overview -->
      <div class="flex flex-col items-center text-white py-12">
        <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
        <p class="text-sm mb-12">
          {{
            new Date(weatherData.currentTime).toLocaleDateString(
              "en-us",
              {
                weekday: "short",
                day: "2-digit",
                month: "long",
              }
            )
          }}
          {{
            new Date(weatherData.currentTime).toLocaleTimeString(
              "en-us",
              {
                timeStyle: "short",
              }
            )
          }}
        </p>
        <p class="text-8xl mb-8">
          {{ Math.round(weatherData.main.temp) }}&deg;
        </p>
        <p>
          Feels like
          {{ Math.round(weatherData.main.feels_like) }} &deg;
        </p>
        <p class="capitalize">
          {{ weatherData.weather[0].description }}
        </p>
        <img
          class="w-[150px] h-auto"
          :src="
            `http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`
          "
          alt=""
        />
      </div>
  
      <hr class="border-white border-opacity-10 border w-full" />
  
  
      <!-- Weekly Weather -->
      <div class="max-w-screen-md w-full py-12">
        <div class="mx-8 text-white">
          <h2 class="mb-4">7 Day Forecast</h2>
          <div
              v-for="hourData in WeatherDataHourly"
              class="flex items-center"
            >
              <p class="flex-1">
                {{
                    new Date(hourData.dt_txt).toLocaleDateString(
                    "en-us",
                    {
                        weekday: "short",
                        day: "2-digit",
                        month: "long",
                    }
                    )
                }}
                {{
                    new Date(hourData.dt_txt).toLocaleTimeString(
                    "en-us",
                    {
                        timeStyle: "short",
                    }
                    )
                }}
              </p>
            <img
              class="w-[50px] h-[50px] object-cover"
              :src="
                `http://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`
              "
              alt=""
            />
            <div class="flex gap-2 flex-1 justify-end">
              <p>H: {{ Math.round(hourData.main.temp_max) }}</p>
              <p>L: {{ Math.round(hourData.main.temp_min) }}</p>
            </div>
          </div>
        </div>
      </div>

      <!-- <div class="flex items-center gap-2 py-12 text-white cursor-pointer duration-150 hover:text-red-500" @click="removeCity">
        <i class="fa-solid fa-trash"></i>
        <p>Remove City</p>
      </div> -->
    </div>

    
  </template>

<script setup>
import axios from 'axios';
import { useRoute } from 'vue-router';
import { useRouter } from 'vue-router';

const route = useRoute();
const apiKey =  import.meta.env.VITE_WEATHERMAP_API_KEY

function pad(number) {
  return number.toString().padStart(2, '0');
}

function getCorrectTimeStamp(date){
    const utcYear = date.getUTCFullYear();
    const utcMonth = date.getUTCMonth() + 1; // Note: Months are zero-indexed, so we add 1
    const utcDay = date.getUTCDate();
    const utcHours = date.getUTCHours();
    const utcMinutes = date.getUTCMinutes();
    const utcSeconds = date.getUTCSeconds();
    return `${utcYear}-${pad(utcMonth)}-${pad(utcDay)} ${pad(utcHours)}:${pad(utcMinutes)}:${pad(utcSeconds)}`;
}



const getWeatherData = async () => {
    
    try {
        
        const weatherData  = await axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${route.query.lat}&lon=${route.query.lng}&appid=${apiKey}&units=imperial`)
        const date = new Date(new Date().getTime() + (weatherData.data.timezone * 1000));
        const utcDateTime = getCorrectTimeStamp(date)
       
        // await new Promise(r => setTimeout(r, 1000));

        weatherData.data.currentTime = utcDateTime
        return weatherData.data
    } catch (error) {

        console.log(error);
    }

};

const getWeatherDataHourly = async () => {
    try {
        const WeatherDataHourly = await axios.get(`https://api.openweathermap.org/data/2.5/forecast?lat=${route.query.lat}&lon=${route.query.lng}&appid=${apiKey}&units=imperial`)
        return WeatherDataHourly.data.list

    } catch (error) {
        console.log(error);
    }
}



const weatherData = await getWeatherData();
const WeatherDataHourly = await getWeatherDataHourly();

</script>
