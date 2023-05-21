<template>
  <div class="text-center text-white">
    <!--Banner-->
    <div
      v-if="route.query.preview"
      class="w-full bg-weather-secondary py-4"
    >
      You are currently previewing the city click the '+' icon to start tracking the city
    </div>

    <!--City Weather Details-->
    <div class="mt-10 text-3xl">
      {{ route.params.city }}
    </div>

    <div class="mt-2 text-lg">
      {{ new Date(cityWeather.current.dt * 1000)
          .toLocaleString("en-US", { weekday: "short", month: "long", day: "2-digit"})
      }}
      {{ new Date(cityWeather.currentTime * 1000)
          .toLocaleString("en-US", { timeStyle: "short" })
      }}
    </div>

    <div class="text-7xl my-10">
      {{ Math.round(cityWeather.current.temp) }}&deg;
    </div>

    <div class="text-lg mt-10">
      Feels like {{ Math.round(cityWeather.current.feels_like) }}&deg;
    </div>

    <div class="text-lg">
      {{ cityWeather.current.weather[0].description }}
    </div>

    <img class="m-auto" :src="`https://openweathermap.org/img/wn/${cityWeather.current.weather[0].icon}@2x.png`"/>

    <hr class="w-full text-white opacity-10 my-10">

    <!--Hourly Weather Details-->
    <div class="container text-left">
      <p class="mb-4">Hourly weather</p>
   
      <div class="flex gap-10 overflow-x-scroll">
        <div
          v-for="hourData in cityWeather.hourly"
          :key="hourData.dt"
          class="flex flex-col gap-4 items-center"
        >
          <p class="whitespace-nowrap text-md">
            {{ new Date(hourData.currentTime)
                .toLocaleString("en-US", { hour: "numeric" })
            }}
          </p>

          <img class="m-auto" :src="`https://openweathermap.org/img/wn/${hourData.weather[0].icon}@2x.png`"/>

          <p class="whitespace-nowrap text-md">
            {{ Math.round(hourData.temp) }}&deg;
          </p>
        </div>
      </div>
    </div>

    <hr class="w-full text-white opacity-10 my-10">
    
    <!--7 days forecast-->
    <div class="container text-left">
      <p class="mb-4">7 Day Forecast</p>

      <div
        v-for="dailyData in cityWeather.daily"
        :key="dailyData.dt"
        class="flex gap-4 justify-between"
      >
        <p class="flex-1">
          {{ new Date(dailyData.dt * 1000)
              .toLocaleString("en-US", { weekday: "long" })
          }}
        </p>

        <img
          class="object-cover w-[50px] h-[50px]"
          :src="`https://openweathermap.org/img/wn/${dailyData.weather[0].icon}@2x.png`
        "/>

        <div class="flex flex-1 justify-end gap-2">
          <p>
            H: {{ Math.round(dailyData.temp.max) }}&deg;
          </p>
          <p>
            L: {{ Math.round(dailyData.temp.min) }}&deg;
          </p>
        </div>
      </div>
    </div>

    <div
      class="text-center my-6 cursor-pointer"
      @click="removeCity"
    >
      <i class="fa-sharp fa-solid fa-trash"></i>
      Remove city
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const apiKey = '993eef70ad22f3e2a63a59153360bcf9';
const route = useRoute();
const router = useRouter();

const getCityWeather = async () => {
 try {
  const weatherData = await axios.get(`https://api.openweathermap.org/data/2.5/onecall?lat=${route.query.lat}&lon=${route.query.long}&exclude={part}&appid=${apiKey}&units=imperial`);

  // cal current date & time
  const localOffset = new Date().getTimezoneOffset() * 60000;
  const utc = weatherData.data.current.dt * 1000 + localOffset;
  weatherData.data.currentTime =
    utc + 1000 * weatherData.data.timezone_offset;

  // cal hourly weather offset
  weatherData.data.hourly.forEach((hour) => {
    const utc = hour.dt * 1000 + localOffset;
    hour.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;
  });

  await new Promise((res) => setTimeout(res, 1000));

  return weatherData.data;
 } catch(err) {
  console.log(err);
 }
};

const removeCity = () => {
  const savedCities = JSON.parse(localStorage.getItem('savedCities'));
  console.log(route.params.id);

  const filteredCities = savedCities.filter((city) => {
    console.log(city);
    
    return city.id !== +route.params.id;
  });

  localStorage.setItem('savedCities', JSON.stringify(filteredCities));

  router.push({
    name: 'home'
  });
};

const cityWeather = await getCityWeather();
</script>
