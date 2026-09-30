<template>
  <li class="weather-item">
    <p class="time">{{ time }}</p>
    <img :src="`icons/${weatherIcon}.svg`" alt="Weather icon" class="weather-icon" />
    <p class="temperature">
      {{ temperature }}
      <span>°</span>
    </p>
  </li>
</template>

<script setup>
import { computed } from 'vue'
import { weatherCodes } from '../constants.js'

const props = defineProps({
  hourlyWeather: {
    type: Object,
    required: true,
  },
})

const temperature = computed(() => Math.floor(props.hourlyWeather.temp_c))

const time = computed(() => {
  return props.hourlyWeather.time.split(' ')[1].substring(0, 5)
})

const weatherIcon = computed(() => {
  return Object.keys(weatherCodes).find((icon) =>
    weatherCodes[icon].includes(props.hourlyWeather.condition.code),
  )
})
</script>
