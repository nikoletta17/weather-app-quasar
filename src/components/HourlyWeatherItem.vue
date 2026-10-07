<template>
  <li class="weather-item">
    <p class="time">{{ time }}</p>
    <img :src="`/icons/${weatherIcon}.svg`" alt="Weather icon" class="weather-icon" />
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

const temperature = computed(() => Math.floor(props.hourlyWeather?.temp_c ?? 0))

const time = computed(() => {
  if (!props.hourlyWeather?.time) return ''
  return props.hourlyWeather.time.split(' ')[1].substring(0, 5)
})

const weatherIcon = computed(() => {
  const code = Number(props.hourlyWeather?.condition?.code)

  // Шукаємо назву іконки за кодом
  const found = Object.keys(weatherCodes).find((icon) => weatherCodes[icon].includes(code))

  // Якщо коду немає в списку, ставимо дефолтну 'clouds', щоб картинка не ламалася
  return found || 'clouds'
})
</script>
