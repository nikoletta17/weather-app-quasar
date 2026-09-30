<template>
  <button
    type="button"
    class="theme-switcher-btn"
    @click="toggleTheme"
    title="Змінити тему"
  >
    <span class="material-symbols-rounded">
      {{ $q.dark.isActive ? 'light_mode' : 'dark_mode' }}
    </span>
  </button>
</template>

<script setup>
import { onMounted } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

const updateBodyClass = (isDark) => {
  document.body.classList.toggle('dark', isDark)
  document.body.classList.toggle('light', !isDark)
}

const toggleTheme = () => {
  $q.dark.toggle()
  updateBodyClass($q.dark.isActive)
}

onMounted(() => {
  // Встановлюємо світлу тему за замовчуванням
  $q.dark.set(false)
  updateBodyClass(false)
})
</script>
