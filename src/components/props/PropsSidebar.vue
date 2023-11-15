<template>
  <vue-resizable :min-width="minWidth" :max-width="maxWidth" :width="width" :active="['r']">
    <div :class="['sidebar', { active: currentView === 'props' }]">
      <button @click="addLayout">Add New Layout</button>
      <select v-model="selectedLayoutIndex" @change="switchSelectedLayout">
        <option v-for="(layout, index) in layouts" :key="index" :value="index">{{ layout.name }}</option>
      </select>
      <AreaProps :area="currentArea" />
    </div>
  </vue-resizable>
</template>

<script setup lang="ts">
import VueResizable from 'vue-resizable'
import { useAppState, layouts, switchLayout, addLayout, mainArea } from '../../store.js'
import { onMounted, onUnmounted, ref, defineProps } from 'vue'
import { debounce } from '../../utils'

const { currentArea, currentView } = useAppState()
// console.log('The current area is: ', currentArea.value )
const maxWidth = ref(0)
const minWidth = ref(0)
const width = ref(0)
const selectedLayoutIndex = ref(0)

onMounted(() => {
  window.addEventListener('resize', debounce(handleResize))
})

onUnmounted(() => {
  window.removeEventListener('resize', debounce(handleResize))
})

function switchSelectedLayout() {
  switchLayout(selectedLayoutIndex.value)
}

function handleResize() {
  if (window.innerWidth < 768) {
    maxWidth.value = 0
    minWidth.value = 0
    width.value = 0
  } else {
    maxWidth.value = 320
    minWidth.value = 240
    width.value = 275
  }
}

handleResize()

defineProps<{ area }>()
</script>

<style scoped lang="postcss">
.sidebar {
  z-index: 20000;
  color: var(--color-white);
  overflow: auto;
  padding: 0;
  text-align: left;
  transition: transform 0.2s ease-in;
  user-select: none;
  display: flex;
  flex-direction: column;
  @media screen and (max-width: 768px) {
    transform: translateX(-100%);
    position: fixed;
    bottom: 0;
    top: 48px;
    width: 100%;
    background: var(--color-gray-darkest);
    a.brand {
      display: none;
    }
    &.active {
      transform: translateX(0);
    }
    .sidebar-logo {
      display: none;
    }
  }
}

.sidebar button {
  margin-bottom: 1rem;
  margin-top: 1rem;
}

.sidebar select {
  margin-bottom: 1rem;
}

@media screen and (max-width: 768px) {
  .btn-undo,
  .btn-redo,
  .btn-github {
    display: none;
  }
}
</style>
