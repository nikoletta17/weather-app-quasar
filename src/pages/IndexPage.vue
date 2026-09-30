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
  weatherIcon: 'clear',
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
  try {
    const response = await fetch(API_URL)
    if (!response.ok) throw new Error()
    const data = await response.json()

    const temperature = Math.floor(data.current.temp_c)
    const description = data.current.condition.text
    const weatherIcon = Object.keys(weatherCodes).find((icon) =>
      weatherCodes[icon].includes(data.current.condition.code),
    )

    currentWeather.value = { temperature, description, weatherIcon }

    const combinedHourlyData = [
      ...data.forecast.forecastday[0].hour,
      ...data.forecast.forecastday[1].hour,
    ]

    // Нормалізація назви: замінюємо застарілі чи районні назви на Dnipro
    let cityName = data.location.name
    const lowerCity = cityName.toLowerCase()

    if (
      lowerCity.includes('dnepropetrovsk') ||
      lowerCity.includes('lotskamenka') ||
      lowerCity.includes('amur') ||
      lowerCity.includes('dnepr')
    ) {
      cityName = 'Dnipro'
    }

    if (searchSectionComp.value?.searchInputRef) {
      searchSectionComp.value.searchInputRef.value = cityName
    }

    filterHourlyForecast(combinedHourlyData)
  } catch {
    hasNoResults.value = true
  }
}

onMounted(() => {
  const API_KEY = import.meta.env.VITE_WEATHER_API_KEY || 'e0af1c4c4dd149cb8ed103703262309'
  getWeatherDetails(
    `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=Dnepropetrovsk&days=2`,
  )
})
</script>
