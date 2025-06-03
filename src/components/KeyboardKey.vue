<script setup>
import { computed } from 'vue'

const props = defineProps({
  keyValue: { type: String, required: true },
  status: { type: String, default: 'default' }
})

defineEmits(['key-press'])

const statusClass = computed(() => {
  if (props.status === 'green') return 'correct'
  if (props.status === 'yellow') return 'present'
  if (props.status === 'gray') return 'absent'
  return ''
})
</script>

<template>
  <button
    class="keyboard-key"
    :class="[statusClass, { 'special-key': keyValue === 'ENTER' || keyValue === 'DEL', 'enter-key': keyValue === 'ENTER', 'del-key': keyValue === 'DEL' }]" 
    @click="$emit('key-press', keyValue)"
  >
    <span v-if="keyValue === 'DEL'">&#x21A9;</span>
    <span v-else>{{ keyValue }}</span>
  </button>
</template>

<style scoped>
.keyboard-key {
  width: 43px;
  height: 58px;
  margin: 2px;
  font-size: 18px;
  border: 1px solid #ccc;
  border-radius: 5px;
  cursor: pointer;
  color: white;
  background-color: #444;
  transition: background-color 0.3s;
}

/* ENTER and DEL sizing */
.special-key {
  width: 65.41px; 
  height: 58px; 
  display: flex;
  justify-content: center;
  align-items: center;
}

.enter-key {
  font-size: 15px;
}

.del-key {
  font-size: 20px;
}

/* Match previous appearance */
.correct {
  background-color: #6aaa64;
}

.present {
  background-color: #c9b458;
}

.absent {
  background-color: #787c7e;
}
</style>
