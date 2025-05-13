<template>
  <div :class="['letter', status]">
    {{ char }}
  </div>
</template>

<script setup>
defineProps({
  char: String,
  status: String
})
</script>

<style scoped>
.letter {
  width: 2.5rem;
  height: 2.5rem;
  border: 1px solid #000;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  text-transform: uppercase;
  background-color: white;
  color: black;
}

.letter.green {
  background-color: #6aaa64;
  color: white;
}

.letter.yellow {
  background-color: #c9b458;
  color: white;
}

.letter.gray {
  background-color: #787c7e;
  color: white;
}
</style>
