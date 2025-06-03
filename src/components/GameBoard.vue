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
import { ref, onMounted } from "vue";
import Word from "./Word.vue";
import Keyboard from "./KeyboardLayout.vue";

// State
const words = ref([]);
const currentAttempt = ref(0);
const currentInput = ref([]);
const targetWord = ref("");
const keyStatuses = ref({});

// Setup empty board
function initBoard() {
  words.value = Array.from({ length: 6 }, () => ({
    letters: Array(5).fill(""),
    statuses: Array(5).fill(""),
  }));
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

  // Validate with Wiktionary API
  const isValid = await validateFrenchWord(guess);
  if (!isValid) {
    alert("Ce mot n’existe pas dans le dictionnaire.");
    return;
  }

  const statuses = Array(5).fill("gray");
  const targetArray = targetWord.value.split("");
  const used = Array(5).fill(false);

  // First pass: correct positions (green)
  for (let i = 0; i < 5; i++) {
    if (guess[i] === targetArray[i]) {
      statuses[i] = "green";
      used[i] = true;
      keyStatuses.value[guess[i]] = "green";
    }
  }

  // Second pass: wrong positions (yellow)
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

// Validate word existence with Wiktionary API
async function validateFrenchWord(word) {
  const url = `https://fr.wiktionary.org/w/api.php?action=parse&page=${word.toLowerCase()}&prop=text&format=json&origin=*`;
  try {
    const res = await fetch(url);
    const data = await res.json();

    const html = data.parse?.text["*"];
    if (!html) return false;

    // Check if the French section exists in the HTML (e.g., "<h2>Français")
    const isFrench = html.includes('langue="fr"') || html.includes("Français");
    return isFrench;
  } catch (err) {
    console.error("Erreur de validation avec Wiktionary :", err);
    return false;
  }
}

// Fetch target word from public API
onMounted(async () => {
  try {
    let word = "";
    do {
      const res = await fetch("https://trouve-mot.fr/api/size/5");
      const data = await res.json();
      word = data[0]?.name ?? "";
    } while (/[À-ÿ]/.test(word)); // Filter words with accents

    targetWord.value = word.toUpperCase();
    console.log("Mot à deviner :", targetWord.value);
    initBoard();
  } catch (err) {
    console.error("Erreur lors de la récupération du mot :", err);
  }
});
</script>

<style scoped>
.game-board-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  padding: 2rem;
  background-color: var(
    --white
  ); /* this changes automatically if dark mode updates it */
  color: var(--dark-grey);
}

.game-board {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  align-items: center;
  background-color: var(
    --light-grey
  ); /* or use var(--white) for more contrast */
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 0 12px rgba(0, 0, 0, 0.15);
  max-width: 500px;
  width: 100%;
  transition: background-color 0.3s ease, color 0.3s ease;
}
</style>
