<template>
  <div class="game-board-wrapper">
    <div class="game-board">
      <!-- Toggle between normal and hard mode -->
      <button @click="toggleGameMode" class="toggle-mode">
        {{ isHardMode ? "Switch to Normal Mode" : "Switch to Hard Mode" }}
      </button>

      <!-- Render the words already attempted-->
      <Word
        v-for="(word, index) in words"
        :key="index"
        :letters="word.letters"
        :statuses="word.statuses"
      />

      <!-- Keyboard input component -->
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
import { ref, onMounted, onUnmounted, watch, computed } from "vue";
import Word from "./Word.vue";
import Keyboard from "./KeyboardLayout.vue";

const words = ref([]);
const currentAttempt = ref(0);
const currentInput = ref([]);
const targetWord = ref(""); // Game logic (no accents)
const targetWordRaw = ref(""); // Original word (possibly with accents)
const keyStatuses = ref({});
const currentLanguage = ref("en");
const isHardMode = ref(localStorage.getItem("wordleMode") === "hard");

// Dynamic word length and max attempts based on mode
const wordLength = computed(() => (isHardMode.value ? 7 : 5));
const maxAttempts = computed(() => (isHardMode.value ? 5 : 6));

// Fallback words in case API fails
const fallbackWords = {
  en: {
    easy: [
      "APPLE",
      "BEACH",
      "CHAIR",
      "DANCE",
      "EARTH",
      "FRUIT",
      "GRASS",
      "HOUSE",
      "JUICE",
      "LIGHT",
    ],
    hard: [
      "BALLOON",
      "CAPTAIN",
      "DIAMOND",
      "ELEGANT",
      "FREEDOM",
      "GALLERY",
      "HARMONY",
      "JOURNEY",
      "KITCHEN",
      "LIBRARY",
    ],
  },
  fr: {
    easy: [
      "ARBRE",
      "BLEUE",
      "CHAIR",
      "DINER",
      "ÉTAGE",
      "FLEUR",
      "GOÛTER",
      "HIVER",
      "JOUER",
      "LUMIÈRE",
    ],
    hard: [
      "JARDINS",
      "KILOMÈTRE",
      "LUMINEUX",
      "MAGNIFIQUE",
      "NATUREL",
      "OPÉRATION",
      "PARFAIT",
      "QUESTION",
      "RÉALITÉ",
      "SÉCURITÉ",
    ],
  },
  es: {
    easy: [
      "AGUA",
      "BARCO",
      "CIELO",
      "DULCE",
      "ESCUELA",
      "FLOR",
      "GATO",
      "HOJA",
      "IGUAL",
      "JUGAR",
    ],
    hard: [
      "AMISTAD",
      "BICICLETA",
      "CREATIVO",
      "DESCUBRIR",
      "ESPERANZA",
      "FELICIDAD",
      "GENEROSO",
      "HONESTIDAD",
      "IMAGINAR",
      "JUBILACIÓN",
    ],
  },
  it: {
    easy: [
      "ACQUA",
      "BAMBINO",
      "CANE",
      "DOLCE",
      "EUROPA",
      "FIORE",
      "GATTO",
      "HOTEL",
      "ISOLA",
      "LIBRO",
    ],
    hard: [
      "ARMONIA",
      "BELLEZZA",
      "CIVILTÀ",
      "DOLCEZZA",
      "ELEGANZA",
      "FANTASIA",
      "GENEROSO",
      "IMMAGINE",
      "LEGGEREZZA",
      "MERAVIGLIA",
    ],
  },
  de: {
    easy: [
      "APFEL",
      "BLUME",
      "CHAIR",
      "DANK",
      "ESSEN",
      "FREUND",
      "GARTEN",
      "HAUS",
      "IGEL",
      "JAHR",
    ],
    hard: [
      "ABENTEUER",
      "BESCHREIBEN",
      "CHAMPAGNER",
      "DOKUMENT",
      "ERFINDUNG",
      "FREIHEIT",
      "GEDICHT",
      "HANDWERK",
      "IDEALIST",
      "JOURNALIST",
    ],
  },
};

function removeAccents(str) {
  return str.normalize("NFD").replace(/[̀-ͯ]/g, "");
}

// Watches for changes in reactive data and triggers action. When any watched value changes, calls saveGame() function
watch(
  [words, currentAttempt, currentInput, keyStatuses, targetWord, isHardMode],
  saveGame,
  {
    deep: true,
  }
);

const STORAGE_KEY = "wordleGameState";

function initBoard() {
  words.value = Array.from({ length: maxAttempts.value }, () => ({
    letters: Array(wordLength.value).fill(""),
    statuses: Array(wordLength.value).fill(""),
  }));
}

function saveGame() {
  const state = {
    words: words.value,
    currentAttempt: currentAttempt.value,
    currentInput: currentInput.value,
    keyStatuses: keyStatuses.value,
    targetWord: targetWord.value,
    targetWordRaw: targetWordRaw.value,
    language: currentLanguage.value,
    isHardMode: isHardMode.value,
  };
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

function loadGame() {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      const state = JSON.parse(saved);
      if (state.targetWord && state.targetWord.length === wordLength.value) {
        words.value = state.words || [];
        currentAttempt.value = state.currentAttempt || 0;
        currentInput.value = state.currentInput || [];
        keyStatuses.value = state.keyStatuses || {};
        targetWord.value = state.targetWord;
        targetWordRaw.value = state.targetWordRaw || state.targetWord;
        currentLanguage.value = state.language || "en";
        isHardMode.value = state.isHardMode || false;
        updateBoard();
        console.log("Resumed word:", targetWordRaw.value);
        return true;
      }
    } catch (err) {
      console.warn("Failed to load saved game:", err);
    }
  }
  return false;
}

function handleKeyPress(key) {
  if (!/^[A-Z]$/.test(key)) return;
  if (
    currentInput.value.length < wordLength.value &&
    currentAttempt.value < maxAttempts.value
  ) {
    currentInput.value.push(key);
    updateBoard();
  }
}

function deleteLetter() {
  if (currentInput.value.length > 0) {
    currentInput.value.pop();
    updateBoard();
  }
}

function updateBoard() {
  const attempt = words.value[currentAttempt.value];
  attempt.letters = [
    ...currentInput.value,
    ...Array(wordLength.value - currentInput.value.length).fill(""),
  ];
}

async function submitWord() {
  if (currentInput.value.length !== wordLength.value) return;

  const guess = currentInput.value.join("");
  const statuses = Array(wordLength.value).fill("gray");
  const targetArray = targetWord.value.split("");
  const used = Array(wordLength.value).fill(false);

  // Green pass
  for (let i = 0; i < wordLength.value; i++) {
    if (guess[i] === targetArray[i]) {
      statuses[i] = "green";
      used[i] = true;
      keyStatuses.value[guess[i]] = "green";
    }
  }

  // Yellow pass
  for (let i = 0; i < wordLength.value; i++) {
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
    alert("Congratulations! You found the word.");
  } else if (currentAttempt.value < maxAttempts.value - 1) {
    currentAttempt.value++;
    currentInput.value = [];
  } else {
    alert(`Too bad! The word was: ${targetWordRaw.value}`);
  }
}

async function fetchWord(lang) {
  try {
    const url = `https://random-word-api.herokuapp.com/word?lang=${lang}&number=1&length=${wordLength.value}`;
    const res = await fetch(url);
    const data = await res.json();
    return data[0]?.toUpperCase() || "";
  } catch (err) {
    console.error("Error fetching word:", err);
    const list =
      fallbackWords[lang]?.[isHardMode.value ? "hard" : "easy"] ||
      fallbackWords["en"].easy;
    return list[Math.floor(Math.random() * list.length)].toUpperCase();
  }
}

// Loads or creates a new game. Gets current language from localStorage (defaults to English)
async function initGame() {
  const lang = localStorage.getItem("wordleLanguage") || "en";
  currentLanguage.value = lang;

  // Tries to load saved game (returns if successful)
  if (loadGame()) return;

  // Fetches a new word of correct length from API (with fallback)
  let raw = "";
  do {
    raw = await fetchWord(lang);
  } while (!raw || raw.length !== wordLength.value);

  // Stores original and processed word
  targetWordRaw.value = raw;
  // Stores accent-free version
  targetWord.value = removeAccents(raw);
  console.log("New word:", targetWordRaw.value);

  // Clears current input
  currentInput.value = [];
  // Resets attempt counter
  currentAttempt.value = 0;
  // Clears keyboard statuses
  keyStatuses.value = {};
  initBoard();
  saveGame();
}

function handleLanguageChange() {
  localStorage.removeItem(STORAGE_KEY);
  initGame();
}

function toggleGameMode() {
  isHardMode.value = !isHardMode.value;
  localStorage.setItem("wordleMode", isHardMode.value ? "hard" : "normal");
  handleLanguageChange();
}

// When the page loads for the first time loads or creates a new game and it calls handleLanguageChange
onMounted(() => {
  initGame();
  window.addEventListener("languageChanged", handleLanguageChange);
});

// When navigating away from the game page removes the event listener to prevent memory leaks
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

.toggle-mode {
  margin-bottom: 1rem;
  padding: 0.5rem 1rem;
  background-color: #444;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  transition: background-color 0.3s;
}
.toggle-mode:hover {
  background-color: #666;
}

.dark .game-board-wrapper {
  background-image: url("/images/word-background-dark.jpg");
  color: var(--white);
}

.dark .game-board {
  background-color: var(--white);
  color: var(--white);
}
</style>
