<!-- src/components/KeyboardLayout.vue -->
<template>
  <div class="keyboard-layout">
    <div v-for="(row, i) in rows" :key="i" class="keyboard-row">
      <KeyboardKey
        v-for="key in row"
        :key="key"
        :key-value="key"
        :status="getStatus(key)"
        @key-press="handleKeyPress"
        :class="{'special-key': key === 'ENTER' || key === 'DEL'}"  
      />
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import KeyboardKey from './KeyboardKey.vue'

// Track the last key pressed for display
const lastPressed = ref('')

// Simulate a letter status map
const letterStatuses = ref({
  A: 'correct',
  E: 'present',
  T: 'absent'
  // other letters are 'default'
})

// Return status for a letter or 'default'
const getStatus = (letter) => {
  return letterStatuses.value[letter] || 'default'
}
const rows = [
  'AZERTYUIOP'.split(''),
  'QSDFGHJKLM'.split(''),
  ['ENTER', 'W', 'X', 'C', 'V', 'B', 'N', 'DEL']
]

// Handle when a key is pressed
const handleKeyPress = (letter) => {
  console.log('Key pressed:', letter)
  lastPressed.value = letter
}
onMounted(() => {
  window.addEventListener('keydown', onKeyDown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', onKeyDown)
})

const onKeyDown = (event) => {
  const key = event.key.toUpperCase()
  if (/^[A-Z]$/.test(key) || key === 'ENTER' || key === 'BACKSPACE') {
    handleKeyPress(key === 'BACKSPACE' ? 'DEL' : key)
  }
}

</script>

<style scoped>
.keyboard-row {
  display: flex;
  justify-content: center;
}
</style>
