<template>
  <div class="search-section">
    <form action="#" class="search-form" @submit.prevent="handleCitySearch">
      <span class="material-symbols-rounded">search</span>
      <input
        ref="searchInputRef"
        type="search"
        placeholder="Enter a city name"
        required
        class="search-input"
        v-model="searchQuery"
      />
      <ul v-if="isOpen && suggestions.length > 0" class="suggestions-list">
        <li
          v-for="item in suggestions"
          :key="item.id"
          class="suggestion-item"
          @click="selectCity(item)"
        >
          <strong>{{ item.name }}</strong
          >, {{ item.country }}
        </li>
      </ul>
    </form>
    <button type="button" class="location-button" @click="handleLocationSearch">
      <span class="material-symbols-rounded">my_location</span>
    </button>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const emit = defineEmits(['get-weather-details'])

const API_KEY = import.meta.env.VITE_WEATHER_API_KEY || 'e0af1c4c4dd149cb8ed103703262309'

const searchQuery = ref('')
const suggestions = ref([])
const isOpen = ref(false)
const searchInputRef = ref(null)

let skipSearch = false
let debounceTimer = null

watch(searchQuery, (newVal) => {
  if (skipSearch) {
    skipSearch = false
    return
  }

  const query = newVal.trim()
  // Підказки ховаються, якщо менше 3 літер
  if (query.length < 2 || /\d/.test(query)) {
    suggestions.value = []
    isOpen.value = false
    return
  }

  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(async () => {
    try {
      const url = `https://api.weatherapi.com/v1/search.json?key=${API_KEY}&q=${encodeURIComponent(query)}`
      const res = await fetch(url)
      const data = await res.json()

      if (Array.isArray(data)) {
        const filtered = data.filter((item) =>
          item.name.toLowerCase().includes(query.toLowerCase()),
        )
        suggestions.value = filtered
        isOpen.value = filtered.length > 0
      } else {
        suggestions.value = []
        isOpen.value = false
      }
    } catch (error) {
      console.error('Ошибка при получении списка городов:', error)
      suggestions.value = []
      isOpen.value = false
    }
  }, 250)
})

const handleCitySearch = () => {
  const query = searchQuery.value.trim()
  if (!query) return

  isOpen.value = false
  suggestions.value = []

  // перевірка на цифри
  if (/\d/.test(query)) {
    emit('get-weather-details', 'INVALID_SEARCH')
    return
  }

  const exactMatch = suggestions.value.find((s) => s.name.toLowerCase() === query.toLowerCase())
  const targetQuery = exactMatch ? `${exactMatch.name}, ${exactMatch.country}` : query

  const API_URL = `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${encodeURIComponent(
    targetQuery,
  )}&days=2`

  emit('get-weather-details', API_URL)
}

const selectCity = (item) => {
  skipSearch = true
  searchQuery.value = item.name
  isOpen.value = false
  suggestions.value = []

  const weatherUrl = `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${encodeURIComponent(
    `${item.name},${item.country}`,
  )}&days=2`
  emit('get-weather-details', weatherUrl)
}

let isLocating = false

const handleLocationSearch = () => {
  if (isLocating) return
  isLocating = true

  navigator.geolocation.getCurrentPosition(
    (position) => {
      try {
        const { latitude, longitude } = position.coords
        const API_URL = `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${latitude},${longitude}&days=2`

        isOpen.value = false
        suggestions.value = []
        skipSearch = true

        emit('get-weather-details', API_URL)
      } catch (err) {
        console.error('Не вдалося віднайти координати:', err)
      } finally {
        isLocating = false
      }
    },
    () => {
      isLocating = false
      alert('Location access denied. Please enable permissions to use this feature.')
    },
    { timeout: 10000 },
  )
}

defineExpose({ searchInputRef })
</script>
