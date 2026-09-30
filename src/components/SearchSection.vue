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
          :key="item.geonameId"
          class="suggestion-item"
          @click="selectCity(item)"
        >
          <strong>{{ item.name }}</strong
          >, {{ item.countryName }}
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
const GEONAMES_USER = import.meta.env.VITE_GEONAMES_USERNAME || 'nikoletta17'

const searchQuery = ref('')
const suggestions = ref([])
const isOpen = ref(false)
const searchInputRef = ref(null)

let skipSearch = false
let debounceTimer = null

// Аналог useEffect з дебаунсом у Vue через watch
watch(searchQuery, (newVal) => {
  if (skipSearch) {
    skipSearch = false
    return
  }

  if (newVal.trim().length < 2) {
    suggestions.value = []
    isOpen.value = false
    return
  }

  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(async () => {
    try {
      const url = `https://secure.geonames.org/searchJSON?name_startsWith=${encodeURIComponent(
        newVal.trim(),
      )}&maxRows=5&featureClass=P&username=${GEONAMES_USER}`

      const res = await fetch(url)
      const data = await res.json()

      if (data.geonames) {
        suggestions.value = data.geonames
        isOpen.value = true
      }
    } catch (error) {
      console.error('Помилка при отриманні списку міст:', error)
    }
  }, 200)
})

const handleCitySearch = () => {
  const query = searchQuery.value.trim()
  if (!query) return
  const API_URL = `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${encodeURIComponent(
    query,
  )}&days=2`
  emit('get-weather-details', API_URL)
}

const selectCity = (item) => {
  skipSearch = true
  searchQuery.value = item.name
  isOpen.value = false
  suggestions.value = []

  const weatherUrl = `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${encodeURIComponent(
    item.name,
  )}&days=2`
  emit('get-weather-details', weatherUrl)
}

let isLocating = false

// Функція для приведення назв районів до міста
const normalizeCityName = (rawName = '') => {
  const lower = rawName.toLowerCase()
  if (
    lower.includes('dnepr') ||
    lower.includes('dnipro') ||
    lower.includes('amur') ||
    lower.includes('lotskamenka')
  ) {
    return 'Dnipro'
  }
  return rawName
}

const handleLocationSearch = () => {
  if (isLocating) return // Захист від повторних багаторазових кліків
  isLocating = true

  navigator.geolocation.getCurrentPosition(
    async (position) => {
      try {
        const { latitude, longitude } = position.coords
        const API_URL = `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=${latitude},${longitude}&days=2`

        emit('get-weather-details', API_URL)

        isOpen.value = false
        suggestions.value = []
        skipSearch = true

        const result = await fetch(
          `https://secure.geonames.org/findNearbyPlaceNameJSON?lat=${latitude}&lng=${longitude}&username=${GEONAMES_USER}`,
        )
        const data = await result.json()

        if (data?.geonames?.[0]?.name) {
          // Застосовуємо нормалізацію:
          searchQuery.value = normalizeCityName(data.geonames[0].name)
        }
      } catch (err) {
        console.error('Не вдалося визначити назву міста:', err)
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
