<script setup>
import { onMounted, onUnmounted } from 'vue'
import KeyboardKey from './KeyboardKey.vue'

const emit = defineEmits(['key-press'])

const rows = [
  'AZERTYUIOP'.split(''),
  'QSDFGHJKLM'.split(''),
  ['ENTER', 'W', 'X', 'C', 'V', 'B', 'N', 'DEL']
]

const props = defineProps({
  keyStatuses: { type: Object, default: () => ({}) }
})

const getStatus = (letter) => {
  return props.keyStatuses[letter] || 'default'
}

const handleKeyPress = (key) => {
  emit('key-press', key)
}

const onKeyDown = (event) => {
  const key = event.key.toUpperCase()
  if (/^[A-Z]$/.test(key) || key === 'ENTER' || key === 'BACKSPACE') {
    handleKeyPress(key === 'BACKSPACE' ? 'DEL' : key)
  }
}

onMounted(() => window.addEventListener('keydown', onKeyDown))
onUnmounted(() => window.removeEventListener('keydown', onKeyDown))
</script>
