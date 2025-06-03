<template>
  <div class="game-board-wrapper">
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
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from "vue";
import Word from "./Word.vue";
import Keyboard from "./KeyboardLayout.vue";

// State
const words = ref([]);
const currentAttempt = ref(0);
const currentInput = ref([]);
const targetWord = ref("");
const keyStatuses = ref({});
const currentLanguage = ref("en");

// Watch for state changes to save
watch([words, currentAttempt, currentInput, keyStatuses, targetWord], saveGame, {
  deep: true,
});

// LocalStorage Keys
const STORAGE_KEY = "wordleGameState";

// Setup empty board
function initBoard() {
  words.value = Array.from({ length: 6 }, () => ({
    letters: Array(5).fill(""),
    statuses: Array(5).fill(""),
  }));
}

// Save current game to LocalStorage
function saveGame() {
  const state = {
    words: words.value,
    currentAttempt: currentAttempt.value,
    currentInput: currentInput.value,
    keyStatuses: keyStatuses.value,
    targetWord: targetWord.value,
    language: currentLanguage.value,
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

// Load game from LocalStorage if exists
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
        currentLanguage.value = state.language || "en";
        updateBoard(); // Reapply current input
        console.log("Resumed word:", targetWord.value);
        return true;
      }
    } catch (err) {
      console.warn("Failed to load saved game:", err);
    }
  }
  return false;
}

// Handle letter input
function handleKeyPress(key) {
  if (!/^[A-Z]$/.test(key)) return;
  if (currentInput.value.length < 5 && currentAttempt.value < 6) {
    currentInput.value.push(key);
    updateBoard();
  }
}

// Delete last letter
function deleteLetter() {
  if (currentInput.value.length > 0) {
    currentInput.value.pop();
    updateBoard();
  }
}

// Apply currentInput to current board row
function updateBoard() {
  const attempt = words.value[currentAttempt.value];
  attempt.letters = [
    ...currentInput.value,
    ...Array(5 - currentInput.value.length).fill(""),
  ];
}

// Submit guess
async function submitWord() {
  if (currentInput.value.length !== 5) return;

  const guess = currentInput.value.join("");

  // Validate French words
  if (currentLanguage.value === "fr") {
    const isValid = await validateFrenchWord(guess);
    if (!isValid) {
      alert("Ce mot n'existe pas dans le dictionnaire.");
      return;
    }
  }

  const statuses = Array(5).fill("gray");
  const targetArray = targetWord.value.split("");
  const used = Array(5).fill(false);

  // First pass (green)
  for (let i = 0; i < 5; i++) {
    if (guess[i] === targetArray[i]) {
      statuses[i] = "green";
      used[i] = true;
      keyStatuses.value[guess[i]] = "green";
    }
  }

  // Second pass (yellow)
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
    alert("Bravo ! Vous avez trouvé le mot.");
  } else if (currentAttempt.value < 5) {
    currentAttempt.value++;
    currentInput.value = [];
  } else {
    alert(`Dommage ! Le mot était : ${targetWord.value}`);
  }
}

// French dictionary validator
async function validateFrenchWord(word) {
  const url = `https://fr.wiktionary.org/w/api.php?action=parse&page=${word.toLowerCase()}&prop=text&format=json&origin=*`;
  try {
    const res = await fetch(url);
    const data = await res.json();
    const html = data.parse?.text["*"];
    if (!html) return false;
    return html.includes('langue="fr"') || html.includes("Français");
  } catch (err) {
    console.error("Erreur de validation avec Wiktionary :", err);
    return false;
  }
}

// Fetch word from API
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

// Start a new game
async function initGame() {
  const lang = localStorage.getItem("wordleLanguage") || "en";
  currentLanguage.value = lang;

  // Try to restore from saved state
  if (loadGame()) return;

  let word = "";
  do {
    word = await fetchWord(lang);
  } while (!word || word.length !== 5);

  targetWord.value = word;
  console.log("Word to guess:", targetWord.value);

  currentInput.value = [];
  currentAttempt.value = 0;
  keyStatuses.value = {};
  initBoard();
  saveGame(); // Save initial state
}

// Handle language change
function handleLanguageChange() {
  localStorage.removeItem(STORAGE_KEY);
  initGame();
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
</style>
