<template>
  <div class="flex bg-weather-secondary p-4 rounded-md my-4 cursor-pointer">
    <div class="flex flex-1 flex-col">
      <span class="text-xl">
        {{ city.city }}
      </span>

      <span class="text-md">
        {{ city.state }}
      </span>
    </div>

    <div class="flex flex-col justify-end">
      <span class="text-3xl text-end">
        {{ city.temp }}&deg;
      </span>
      
      <span class="flex gap-2 text-md">
        <span>H: {{ city.high }}&deg;</span>
        <span>L: {{ city.low }}&deg;</span>
      </span>
    </div>
  </div>
</template>

<script setup>
defineProps({
  city: { type: Object }
});
</script>
