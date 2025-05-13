<template>
  <div class="game-board">
    <Word
      v-for="(word, index) in words"
      :key="index"
      :letters="word.letters"
      :statuses="word.statuses"
    />
    <Keyboard
      :keyStatuses="keyStatuses"        
      @keyPress="handleKeyPress"         
      @submitWord="submitWord"           
      @deleteLetter="deleteLetter"       
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import Word from './Word.vue'
import Keyboard from './Keyboard.vue'

// State variables
const words = ref([])  // Holds the current game board (attempts)
const currentAttempt = ref(0)  // Tracks the current attempt number
const currentInput = ref([])  // Tracks the current input for the guess
const targetWord = ref('')  // The target word to guess
const keyStatuses = ref({})  // Stores the status of each key (gray, yellow, green)

// Initialize the board with empty attempts
function initBoard() {
  words.value = Array.from({ length: 6 }, () => ({
    letters: Array(5).fill(''),
    statuses: Array(5).fill('')
  }))
}

// Handle key presses for letters
function handleKeyPress(key) {
  if (!/^[A-Z]$/.test(key)) return // Only allow uppercase alphabetic characters
  if (currentInput.value.length < 5 && currentAttempt.value < 6) {
    currentInput.value.push(key)
    updateBoard()
  }
}

// Delete the last entered letter
function deleteLetter() {
  if (currentInput.value.length > 0) {
    currentInput.value.pop()
    updateBoard()
  }
}

// Update the game board with the current input
function updateBoard() {
  const attempt = words.value[currentAttempt.value]
  attempt.letters = [...currentInput.value, ...Array(5 - currentInput.value.length).fill('')]
}

// Submit the current word and check if it's correct
function submitWord() {
  if (currentInput.value.length === 5) {
    const guess = currentInput.value.join('')
    const statuses = Array(5).fill('gray')
    const targetArray = targetWord.value.split('')
    const used = Array(5).fill(false)

    // First pass: correct positions (green)
    for (let i = 0; i < 5; i++) {
      if (guess[i] === targetArray[i]) {
        statuses[i] = 'green'
        used[i] = true
        keyStatuses.value[guess[i]] = 'green'
      }
    }

    // Second pass: wrong positions (yellow)
    for (let i = 0; i < 5; i++) {
      if (statuses[i] === 'green') continue
      const index = targetArray.findIndex((c, idx) => c === guess[i] && !used[idx])
      if (index !== -1) {
        statuses[i] = 'yellow'
        used[index] = true
        if (keyStatuses.value[guess[i]] !== 'green') {
          keyStatuses.value[guess[i]] = 'yellow'
        }
      } else {
        if (!keyStatuses.value[guess[i]]) {
          keyStatuses.value[guess[i]] = 'gray'
        }
      }
    }

    words.value[currentAttempt.value].statuses = statuses

    if (guess === targetWord.value) {
      alert('Bravo ! Vous avez trouvé le mot.')
    } else if (currentAttempt.value < 5) {
      currentAttempt.value++
      currentInput.value = []
    } else {
      alert(`Dommage ! Le mot était : ${targetWord.value}`)
    }
  }
}

// Handle physical keyboard input
function onKeydown(e) {
  const key = e.key.toUpperCase()
  if (key === 'BACKSPACE') deleteLetter()
  else if (key === 'ENTER') submitWord()
  else handleKeyPress(key)
}

// Fetch a word to guess from the API
onMounted(async () => {
  window.addEventListener('keydown', onKeydown)
  try {
    const response = await fetch('https://trouve-mot.fr/api/size/5')
    const data = await response.json()
    const word = data[0]?.name
    if (!word) throw new Error('Mot non trouvé dans la réponse')
    targetWord.value = word.toUpperCase()
    console.log('Mot à deviner :', targetWord.value)

    initBoard()
  } catch (error) {
    console.error('Erreur lors de la récupération du mot :', error)
  }
})

// Clean up by removing the event listener on unmount
onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKeydown)
})
</script>

<style scoped>
.game-board {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
}
</style>
