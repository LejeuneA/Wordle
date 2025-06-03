<template>
  <div class="keyboard-layout">
    <div v-for="(row, i) in rows" :key="i" class="keyboard-row">
      <KeyboardKey
        v-for="key in row"
        :key="key"
        :key-value="key"
        :status="getStatus(key)"
        @key-press="handleKeyPress"
      />
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted} from 'vue'
import KeyboardKey from './KeyboardKey.vue'

const props = defineProps({
  keyStatuses: { type: Object, default: () => ({}) }
})

const emit = defineEmits(['keyPress', 'submitWord', 'deleteLetter'])

const rows = [
  'AZERTYUIOP'.split(''),
  'QSDFGHJKLM'.split(''),
  ['ENTER', 'W', 'X', 'C', 'V', 'B', 'N', 'DEL']
]

// Get key status from prop or fallback
const getStatus = (letter) => props.keyStatuses[letter] || 'default'

const handleKeyPress = (key) => {
  if (key === 'ENTER') {
    emit('submitWord')
  } else if (key === 'DEL') {
    emit('deleteLetter')
  } else {
    emit('keyPress', key)
  }
}

const onKeyDown = (event) => {
  const key = event.key.toUpperCase()
  if (/^[A-Z]$/.test(key) || key === 'ENTER' || key === 'BACKSPACE') {
    handleKeyPress(key === 'BACKSPACE' ? 'DEL' : key)
  }
}

onMounted(() => {
  window.addEventListener('keydown', onKeyDown)
})

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
