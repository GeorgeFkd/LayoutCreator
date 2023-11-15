<template>
  <button
    aria-label="Toggle dark mode"
    :class="['btn-dark', { active: darkmode }]"
    @click.left="toggleDarkmode"
    @click.prevent.right="switchToSystemTheme"
  >
    <IconDark />
  </button>
</template>

<script setup lang="ts">
import { Ref, onMounted, watch } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import { darkmode } from '../../store'
type Theme = 'dark' | 'light'
//prepei na mathw ti einai to RemovableRef<T>
let themeStorage: Ref<Theme> = useLocalStorage('layoutit-grid-theme', 'dark')

function toggleDarkmode() {
  darkmode.value = !darkmode.value
  themeStorage.value = darkmode ? 'dark' : 'light'
}

function switchToSystemTheme() {
  darkmode.value = getSystemTheme() === 'dark'
  themeStorage = null
}

onMounted(() => {
  darkmode.value = (themeStorage || getSystemTheme()) === 'dark'
})

watch($$(darkmode), () => {
  document.getElementById('app').classList[darkmode ? 'add' : 'remove']('darkmode')
})

function getSystemTheme() {
  return 'light'
  /* For the moment, we are going to use light by default ignoring the system preference until
     the dark mode in the app is more polished
     
  const darkModeMediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
  return darkModeMediaQuery.media !== 'not all' && darkModeMediaQuery.matches ? 'dark' : 'light'
  */
}
</script>

<style scoped lang="postcss">
button.btn-dark {
  margin-bottom: 10px;
  background: var(--color-black);
  border: solid 1px var(--color-gray-darkest);
  color: var(--color-white);
  border-radius: 25px;
  cursor: pointer;
  width: 42px;
  padding: 0px;
  svg {
    width: 100%;
  }
  &.active {
    background: var(--color-darkmode-active);
  }
}
.buttons button.btn-dark {
  padding: 10px;
}
</style>
