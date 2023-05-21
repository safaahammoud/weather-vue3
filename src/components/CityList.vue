<template>
  <div>
    <div
      v-for="city in cityList"
      :key="city.id"
    >
      <CityCard
        :city="city"
        @click="goToCityDetails(city)"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import axios from "axios";
import { useRouter } from 'vue-router'

import CityCard from "../components/CityCard.vue";

const router = useRouter();
const cityList = ref([]);
const apiKey = '993eef70ad22f3e2a63a59153360bcf9';

const getCities = () => {
  const savedCitiesFromLocalStorage = JSON.parse(localStorage.getItem('savedCities'));

  const values = savedCitiesFromLocalStorage.map(async (city) => {
    const cityWeatherData = await axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${city.coordinates.lat}&lon=${city.coordinates.long}&exclude={part}&appid=${apiKey}&units=imperial`);
    console.log(cityWeatherData);
    
    return {
      id: city.id,
      state: city.state,
      city: city.city,
      temp: Math.round(cityWeatherData.data.main.temp),
      high: Math.round(cityWeatherData.data.main.temp_max),
      low: Math.round(cityWeatherData.data.main.temp_min),
      coordinates: cityWeatherData.data.coord
    };
  });
  
  return Promise.all(values);
};
const goToCityDetails = (city) => {
  console.log(city);
  
  router.push({
    name: 'cityView',
    params: {
      id: city.id, 
      state: city.state,
      city: city.city
    },
    query: {
      lat: city.coordinates.lat,
      long: city.coordinates.lon,
    }
  });
};

cityList.value = await getCities();
</script>
