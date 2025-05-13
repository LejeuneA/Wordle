<template>
  <div class="word">
    <Letter
      v-for="(char, index) in letters"
      :key="index"
      :char="char"
      :status="statuses[index]"
    />
  </div>
</template>

<script setup>
import Letter from './Letter.vue'
defineProps({
  letters: Array,
  statuses: Array
})
</script>

<style scoped>
.word {
  display: flex;
  gap: 0.5rem;
}
</style>
