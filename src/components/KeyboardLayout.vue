<template>
  <div class="keyboard-layout">
    <div v-for="(row, i) in rows" :key="i" class="keyboard-row">
      <KeyboardKey
        v-for="key in row"
        :key="key"
        :key-value="key"
        :status="getStatus(key)"                    
        @key-press="handleKeyPress"                 
        :class="{ 'special-key': key === 'ENTER' || key === 'DEL' }"  
      />
    </div>
  </div>
</template>

<script setup>
// Lifecycle imports
import { onMounted, onUnmounted } from 'vue'
// Reusable key button component
import KeyboardKey from './KeyboardKey.vue'

// Props: receives keyStatuses from parent (used for coloring keys)
const props = defineProps({
  keyStatuses: { type: Object, default: () => ({}) }
})

// Emit custom events to GameBoard (keyPress, submitWord, deleteLetter)
const emit = defineEmits(['keyPress', 'submitWord', 'deleteLetter'])

// Define AZERTY keyboard layout (French)
const rows = [
  'AZERTYUIOP'.split(''),
  'QSDFGHJKLM'.split(''),
  ['ENTER', 'W', 'X', 'C', 'V', 'B', 'N', 'DEL']
]

// Return status of the letter: 'correct', 'present', 'absent', or fallback to 'default'
const getStatus = (letter) => props.keyStatuses[letter] || 'default'

// Handle click or keyboard event and emit the appropriate action
const handleKeyPress = (key) => {
  if (key === 'ENTER') {
    emit('submitWord')
  } else if (key === 'DEL') {
    emit('deleteLetter')
  } else {
    emit('keyPress', key)
  }
}

// Handle physical keyboard input
const onKeyDown = (event) => {
  const key = event.key.toUpperCase()
  if (/^[A-Z]$/.test(key) || key === 'ENTER' || key === 'BACKSPACE') {
    handleKeyPress(key === 'BACKSPACE' ? 'DEL' : key)
  }
}

// Register keyboard listener when component is mounted
onMounted(() => {
  window.addEventListener('keydown', onKeyDown)
})

// Clean up keyboard listener when component is destroyed
onUnmounted(() => {
  window.removeEventListener('keydown', onKeyDown)
})
</script>

<style scoped>
.keyboard-row {
  display: flex;
  justify-content: center;
}
</style>
