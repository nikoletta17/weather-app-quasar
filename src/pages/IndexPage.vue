<template>
  <div class="container">
    <ThemeSwitcher />
    <SearchSection ref="searchSectionComp" @get-weather-details="getWeatherDetails" />

    <NoResult v-if="hasNoResults" />

    <div v-else class="weather-section">
      <CurrentWeather :current-weather="currentWeather" />
      <div class="hourly-forecast">
        <ul class="weather-list">
          <HourlyWeatherItem
            v-for="hourlyWeather in hourlyForecast"
            :key="hourlyWeather.time_epoch"
            :hourly-weather="hourlyWeather"
          />
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import CurrentWeather from '../components/CurrentWeather.vue'
import HourlyWeatherItem from '../components/HourlyWeatherItem.vue'
import SearchSection from '../components/SearchSection.vue'
import ThemeSwitcher from '../components/ThemeSwitcher.vue'
import NoResult from '../components/NoResult.vue'
import { weatherCodes } from '../constants.js'

const currentWeather = ref({
  temperature: 0,
  description: '',
  weatherIcon: 'clouds',
})
const hourlyForecast = ref([])
const hasNoResults = ref(false)
const searchSectionComp = ref(null)

const filterHourlyForecast = (hourlyData) => {
  const currentHour = new Date().setMinutes(0, 0, 0)
  const next24Hours = currentHour + 24 * 60 * 60 * 1000

  const next24HoursData = hourlyData.filter(({ time }) => {
    const forecastTime = new Date(time).getTime()
    return forecastTime >= currentHour && forecastTime <= next24Hours
  })

  hourlyForecast.value = next24HoursData
}

const getWeatherDetails = async (API_URL) => {
  hasNoResults.value = false

  if (API_URL === 'INVALID_SEARCH') {
    hasNoResults.value = true
    return
  }

  try {
    const response = await fetch(API_URL)
    const data = await response.json()

    //API error occured
    if (!response.ok || data.error || !data.current || !data.forecast) {
      hasNoResults.value = true
      return
    }

    const temperature = Math.floor(data.current.temp_c)
    const description = data.current.condition.text

    const code = Number(data.current?.condition?.code)
    const matchedIcon = Object.keys(weatherCodes).find((icon) => weatherCodes[icon].includes(code))
    const weatherIcon = matchedIcon || 'clouds'

    currentWeather.value = { temperature, description, weatherIcon }

    const combinedHourlyData = [
      ...data.forecast.forecastday[0].hour,
      ...data.forecast.forecastday[1].hour,
    ]

    //Перевірка назви міста
    let cityName = data.location.name
    const lowerCity = cityName.toLowerCase()

    const isDniproArea =
      lowerCity.includes('dnepr') ||
      lowerCity.includes('dnepropetrovsk') ||
      lowerCity.includes('lotskamenka') ||
      lowerCity.includes('amur') ||
      (Math.abs(data.location.lat - 48.46) < 0.25 && Math.abs(data.location.lon - 35.04) < 0.25)

    if (isDniproArea && !lowerCity.includes('dniprorudne')) {
      cityName = 'Dnipro'
    }

    if (searchSectionComp.value?.searchInputRef) {
      searchSectionComp.value.searchInputRef.value = cityName
    }

    filterHourlyForecast(combinedHourlyData)
  } catch (error) {
    console.error('Ошибка при получении погоды:', error)
    hasNoResults.value = true
  }
}

onMounted(() => {
  const API_KEY = import.meta.env.VITE_WEATHER_API_KEY || 'e0af1c4c4dd149cb8ed103703262309'
  // Координати Дніпра
  getWeatherDetails(
    `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=48.4647,35.0462&days=2`,
  )
})
</script>
