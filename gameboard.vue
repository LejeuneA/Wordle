<template>
  <div class="game-board-wrapper">
    <div class="game-board">
      <!-- Show game content only when game is not over -->
      <template v-if="!gameOver">
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
      </template>

      <!-- Show only result when game is over -->
      <Result
        v-else
        :isWin="isWin"
        :word="targetWordAccented"
        :attempts="currentAttempt"
        :hintsUsed="hintsUsed"
        @play-again="resetGame"
      />

      <button
        @click="revealHint"
        :disabled="hintsUsed >= 2"
        class="hint-button"
      >
        Use Hint ({{ 2 - hintsUsed }} left)
      </button>

      <div class="hints-container" v-if="revealedLetters.length > 0">
        <p>Revealed Letters:</p>
        <div class="hint-letter" v-for="(letter, idx) in revealedLetters" :key="idx">
          {{ letter }}
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted, onUnmounted, watch, computed } from "vue";
import Word from "./Word.vue";
import Keyboard from "./KeyboardLayout.vue";
import Result from "./Result.vue";

const attemptsUsed = computed(() => {
  if (!gameOver.value) return currentAttempt.value;
  return currentAttempt.value + 1;
});
const isWin = ref(false);
const gameOver = ref(false);
const words = ref([]);
const currentAttempt = ref(0);
const currentInput = ref([]);
const targetWord = ref("");          // Without accents, used for game logic
const targetWordAccented = ref("");  // With accents, used for validation
const keyStatuses = ref({});
const currentLanguage = ref("en");
const hintsUsed = ref(0);
const revealedLetters = ref([]); 
     

// Utility function to remove accents from a string
function removeAccents(str) {
  return str.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
}

// Watch relevant state to save the game in localStorage
watch([words, currentAttempt, currentInput, keyStatuses, targetWord], saveGame, {
  deep: true,
});

const STORAGE_KEY = "wordleGameState";

// Initialize the game board
function initBoard() {
  words.value = Array.from({ length: 6 }, () => ({
    letters: Array(5).fill(""),
    statuses: Array(5).fill(""),
  }));
}

// Save game state to localStorage
function saveGame() {
  const state = {
    words: words.value,
    currentAttempt: currentAttempt.value,
    currentInput: currentInput.value,
    keyStatuses: keyStatuses.value,
    targetWord: targetWord.value,
    targetWordAccented: targetWordAccented.value,
    language: currentLanguage.value,
    hintsUsed: hintsUsed.value,
    revealedLetters: revealedLetters.value,
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

// Load game state from localStorage
function loadGame() {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      const state = JSON.parse(saved);
      if (state.targetWord && state.targetWord.length === 5) {
        words.value = state.words || [];
        currentAttempt.value = state.currentAttempt || 0;
        currentInput.value = state.currentInput || [];
        keyStatuses.value = state.keyStatuses || {};
        targetWord.value = state.targetWord;
        targetWordAccented.value = state.targetWordAccented || targetWord.value;
        currentLanguage.value = state.language || "en";
        hintsUsed.value = state.hintsUsed || 0;               // <--- add this line
        revealedLetters.value = state.revealedLetters || [];  // <--- if you save revealed letters
        updateBoard();
        console.log("Resumed word:", targetWordAccented.value);
        return true;
      }
    } catch (err) {
      console.warn("Failed to load saved game:", err);
    }
  }
  return false;
}


// Handle key press for letters
function handleKeyPress(key) {
  if (!/^[A-Z]$/.test(key)) return;
  if (currentInput.value.length < 5 && currentAttempt.value < 6) {
    currentInput.value.push(key);
    updateBoard();
  }
}

// Delete last letter entered
function deleteLetter() {
  if (currentInput.value.length > 0) {
    currentInput.value.pop();
    updateBoard();
  }
}

// Update the board's current attempt with the current input letters
function updateBoard() {
  const attempt = words.value[currentAttempt.value];
  attempt.letters = [
    ...currentInput.value,
    ...Array(5 - currentInput.value.length).fill(""),
  ];
}

// Validate a French word using Wiktionary
// Tries first with accented word, then without accents if not found
async function validateFrenchWord(word) {
  // Internal function to check Wiktionary for a given word
  async function checkWord(w) {
    const url = `https://fr.wiktionary.org/w/api.php?action=parse&page=${w.toLowerCase()}&prop=text&format=json&origin=*`;
    try {
      const res = await fetch(url);
      const data = await res.json();
      const html = data.parse?.text["*"];
      if (!html) return false;
      return html.includes('langue="fr"') || html.includes("Français");
    } catch (err) {
      console.error("Validation error with Wiktionary:", err);
      return false;
    }
  }

  // Try accented version first
  let valid = await checkWord(word);
  if (valid) return true;

  // If not valid, try without accents
  const noAccentWord = removeAccents(word);
  if (noAccentWord !== word) {
    valid = await checkWord(noAccentWord);
  }

  return valid;
}

// Submit the current guess, validate and update the game state
async function submitWord() {
  if (currentInput.value.length !== 5) return;

  const guess = currentInput.value.join("");

  if (currentLanguage.value === "fr") {
    const isValid = await validateFrenchWord(guess);
    if (!isValid) {
      alert("This word does not exist in the dictionary.");
      return;
    }
  }

  const statuses = Array(5).fill("gray");
  const targetArray = targetWord.value.split("");
  const used = Array(5).fill(false);

  // First pass - green (correct letter and position)
  for (let i = 0; i < 5; i++) {
    if (guess[i] === targetArray[i]) {
      statuses[i] = "green";
      used[i] = true;
      keyStatuses.value[guess[i]] = "green";
    }
  }

  // Second pass - yellow (correct letter, wrong position)
  for (let i = 0; i < 5; i++) {
    if (statuses[i] === "green") continue;
    const index = targetArray.findIndex(
      (c, idx) => c === guess[i] && !used[idx]
    );
    if (index !== -1) {
      statuses[i] = "yellow";
      used[index] = true;
      if (keyStatuses.value[guess[i]] !== "green") {
        keyStatuses.value[guess[i]] = "yellow";
      }
    } else {
      if (!keyStatuses.value[guess[i]]) {
        keyStatuses.value[guess[i]] = "gray";
      }
    }
  }

  words.value[currentAttempt.value].statuses = statuses;

    if (guess === targetWord.value) {
    isWin.value = true;
    gameOver.value = true;
  } else if (currentAttempt.value < 5) {
    currentAttempt.value++;
    currentInput.value = [];
  } else {
    isWin.value = false;
    gameOver.value = true;
  }
}

// Fetch a random 5-letter word for the chosen language
async function fetchWord(lang) {
  try {
    const url = `https://random-word-api.herokuapp.com/word?lang=${lang}&number=1&length=5`;
    const res = await fetch(url);
    const data = await res.json();
    return data[0]?.toUpperCase() || "";
  } catch (err) {
    console.error("Error fetching word:", err);
    return "";
  }
}

// Initialize a new game or restore from localStorage
async function initGame() {
  const lang = localStorage.getItem("wordleLanguage") || "en";
  currentLanguage.value = lang;

  if (loadGame()) return;

  let wordAccented = "";
  do {
    wordAccented = await fetchWord(lang);
  } while (!wordAccented || wordAccented.length !== 5);

  targetWordAccented.value = wordAccented.toUpperCase();
  targetWord.value = removeAccents(targetWordAccented.value);

  currentInput.value = [];
  currentAttempt.value = 0;
  keyStatuses.value = {};
  initBoard();
  saveGame();
}

// Handle language change event, reset game
function handleLanguageChange() {
  localStorage.removeItem(STORAGE_KEY);
  initGame();
}

function resetGame() {
  currentAttempt.value = 0;
  currentInput.value = [];
  keyStatuses.value = {};
  hintsUsed.value = 0;
  revealedLetters.value = [];
  gameOver.value = false;  
  isWin.value = false;     
  initBoard();
  fetchAndSetNewWord();
  saveGame();
}


function revealHint() {
  if (hintsUsed.value >= 2) return;

  const wordLetters = targetWord.value.split("");
  const letterCounts = countLetters(targetWord.value);

  // Count letters found green in guesses
  const foundLettersCounts = {};

  for (let i = 0; i <= currentAttempt.value; i++) {
    const attemptStatuses = words.value[i]?.statuses || [];
    const attemptLetters = words.value[i]?.letters || [];

    attemptStatuses.forEach((status, idx) => {
      if (status === "green" & "yellow") {
        const letter = attemptLetters[idx];
        foundLettersCounts[letter] = (foundLettersCounts[letter] || 0) + 1;
      }
    });
  }

  // Filter letters to reveal:
  // - Not already fully found (green) in guesses
  // - Not already revealed by hint
  const unrevealedLetters = [];

  for (const letter of new Set(wordLetters)) {
    const foundCount = foundLettersCounts[letter] || 0;
    const totalCount = letterCounts[letter];

    const alreadyRevealed = revealedLetters.value.includes(letter);

    if (foundCount < totalCount && !alreadyRevealed) {
      unrevealedLetters.push(letter);
    }
  }

  if (unrevealedLetters.length === 0) return;

  // Randomly pick one letter
  const letter =
    unrevealedLetters[
      Math.floor(Math.random() * unrevealedLetters.length)
    ];

  revealedLetters.value.push(letter);
  hintsUsed.value++;
}


function countLetters(word) {
  const counts = {};
  for (const letter of word) {
    counts[letter] = (counts[letter] || 0) + 1;
  }
  return counts;
}



onMounted(() => {
  initGame();
  window.addEventListener("languageChanged", handleLanguageChange);
});

onUnmounted(() => {
  window.removeEventListener("languageChanged", handleLanguageChange);
});
</script>


<style scoped>
.game-board-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  background-size: cover;
  background-position: center;
  height: 100vh;
  padding: 2rem;
  background-image: url("/images/word-background-light.jpg");
  color: var(--dark-grey);
  transition: background-color 0.3s ease;
}

.game-board {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: center;
  background-color: var(--dark-grey);
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  max-width: 500px;
  width: 100%;
  transition: background-color 0.3s ease, color 0.3s ease;
}

/* Dark mode styles */
.dark .game-board-wrapper {
  background-image: url("/images/word-background-dark.jpg");
  color: var(--white);
}

.dark .game-board {
  background-color: var(--white);
  color: var(--white);
}
.hint-button {
  padding: 0.5rem 1rem;
  margin-top: 1rem;
  background-color: #ffd54f;
  color: #000;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.3s ease;
}

.hint-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
.hints-container {
  margin-top: 1rem;
  display: flex;
  gap: 1rem;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  color: #ffd54f;
}

.hint-letter {
  background: #222;
  padding: 0.5rem 0.7rem;
  border-radius: 6px;
  font-size: 1.4rem;
  user-select: none;
}


</style>
