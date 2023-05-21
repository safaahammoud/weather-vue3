<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";

import CityList from '../components/CityList.vue';
import CityCardSkeleton from '../components/CityCardSkeleton.vue';

const apiKey = '993eef70ad22f3e2a63a59153360bcf9';
const searchQuery = ref<string>('');
const searchError = ref<boolean>(false);
let searchQueryTimeout = ref(null);
let searchResult = ref(null);
const router = useRouter();

const getSearchResult = () => {
  searchError.value = false;

  try {
    clearTimeout(searchQueryTimeout.value);
    
    searchQueryTimeout.value = setTimeout(async () => {
      if(searchQuery.value) {
        const results = await axios.get(`https://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value}&limit=5&appid=${apiKey}`);
  
        searchResult.value = results.data;
  
        return;
      }
  
      searchResult.value = null;
    }, 300);
  } catch {
    searchError.value = true;
  }
};

const previewCity = (city) => {
  console.log('router', router);
  router.push({
    name: 'cityView',
    params: {
      id: Math.round(Math.random()*10),
      state: city.state, city: city.name },
    query: {
      lat: city.lat,
      long: city.lon,
      preview: true,
    }
  });
};
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="search for a city or state"
        class="
          py-2 px-1 w-full bg-transparent border-b
        focus:bg-weather-secondary focus:outline-none
          focus:border-[#004E71]
          focus:shadow-[0px_1px_0_0_#004E71]
        "
        @input="getSearchResult"
      />

      <p v-if="searchError">
        Sorry, something went wrong, please try again.
      </p>

      <p v-if="!searchError && searchQuery && searchResult && searchResult.length === 0">
        No results match your query, try a different term.
      </p>

      <ul
        v-else-if="searchResult && searchResult.length"
        class="absolute top-[66px] w-full shadow-md py-2 px-1 bg-weather-secondary"
      >
        <li
          v-for="city in searchResult"
         :key="city.name"
         class="cursor-pointer"
         @click="previewCity(city)"
        >
          {{ city.name }}
        </li>
      </ul>

      <suspense>
        <CityList />

        <template #fallback>
           <CityCardSkeleton />
        </template>
      </suspense>
    </div>
  </main>
</template>