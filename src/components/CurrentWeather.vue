<template>
  <div class="current-weather">
    <img
      :src="`icons/${currentWeather.weatherIcon}.svg`"
      alt="Weather icon"
      class="weather-icon"
    />
    <h2 class="temperature">
      {{ currentWeather.temperature }}
      <span>°C</span>
    </h2>
    <p class="description">{{ currentWeather.description }}</p>
  </div>
</template>

<script setup>
defineProps({
  currentWeather: {
    type: Object,
    required: true
  }
})
</script>
