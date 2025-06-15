<template>
  <div class="home-view">
    <div class="hero-section">
      <div class="content-box">
        <img src="/images/wordle-logo.png" class="logo" />
        <h1>{{ translations[currentLanguage]?.title || "Wordle" }}</h1>
        <p>{{ translations[currentLanguage]?.description }}</p>

        <Button
          size="medium"
          customClass="primary"
          @click="$router.push('/game')"
        >
          {{ translations[currentLanguage]?.playButton }}
        </Button>
      </div>
    </div>
  </div>
</template>

<script setup>
import Button from "@/components/Button.vue";
import { ref, onMounted } from "vue";

const currentLanguage = ref(localStorage.getItem("wordleLanguage") || "en");

const translations = {
  en: {
    title: "Wordle",
    description: "You have 6 chances to guess a 5-letter word.",
    playButton: "Play",
  },
  fr: {
    title: "Wordle",
    description: "Vous avez 6 chances de deviner un mot de 5 lettres.",
    playButton: "Jouer",
  },
  es: {
    title: "Wordle",
    description: "Tienes 6 intentos para adivinar una palabra de 5 letras.",
    playButton: "Jugar",
  },
  it: {
    title: "Wordle",
    description: "Hai 6 tentativi per indovinare una parola di 5 lettere.",
    playButton: "Gioca",
  },
  de: {
    title: "Wordle",
    description: "Sie haben 6 Versuche, ein 5-Buchstaben-Wort zu erraten.",
    playButton: "Spielen",
  },
};

onMounted(() => {
  window.addEventListener("languageChanged", (event) => {
    currentLanguage.value = event.detail?.language || "en";
  });
});
</script>

<style scoped>
.home-view {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.hero-section {
  flex: 1;
  background-image: url("/images/word-background-light.jpg");
  background-size: cover;
  background-position: center;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 2rem;
  transition: background-color 0.3s ease;
}

.content-box {
  background-color: var(--dark-grey);
  width: 30rem;
  height: 30rem;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
  transition: background-color 0.3s ease, color 0.3s ease;
}

.logo {
  width: 100px;
  margin-bottom: 30px;
}

h1 {
  font-family: var(--font-family);
  font-size: 3.5rem;
  color: var(--white);
  margin-bottom: 20px;
  font-weight: 800;
}

p {
  font-family: var(--font-family);
  font-size: 1.2rem;
  color: var(--white);
  margin-bottom: 40px;
  text-align: center;
}

/* Dark mode styles */
.dark .hero-section {
  background-image: url("/images/word-background-dark.jpg");
}

.dark .content-box {
  background-color: var(--white);
  color: var(--dark-grey);
}

.dark h1,
.dark p {
  color: var(--dark-grey);
}
</style>
