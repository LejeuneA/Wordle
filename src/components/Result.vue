<template>
  <div class="result-container">
    <h2 v-if="isWin" class="win-message"> You found the word! </h2>
    <h2 v-else class="lose-message"> You didn't find the word. </h2>
    <p>The word was: <strong>{{ word }}</strong></p>
    <p v-if="isWin">You found it in {{ attempts }} attempts.</p>
    <p v-else>You tried {{ attempts }} times.</p>
    <button @click="$emit('play-again')">Play Again</button>
  </div>
</template>


<script setup>
defineProps({
  isWin: Boolean,
  word: String,
  attempts: Number,
});
</script>

<style scoped>
.result-container {
  text-align: center;
  padding: 1.5rem;
  border-radius: 12px;
  background: #282c34;
  color: white;
  box-shadow: 0 0 15px rgba(0,0,0,0.7);
}
.win-message {
  color: #4caf50;
}
.lose-message {
  color: #f44336;
}
button {
  margin-top: 1rem;
  padding: 0.6rem 1.2rem;
  border: none;
  background-color: #61dafb;
  color: #282c34;
  font-weight: bold;
  border-radius: 6px;
  cursor: pointer;
}
button:hover {
  background-color: #21a1f1;
}
</style>
