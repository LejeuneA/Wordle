<template>
  <div class="home-view">
    <Navbar />

    <div class="hero-section">
      <div class="content-box">
        <img src="@/assets/wordle-logo.png" alt="Wordle Logo" class="logo" />
        <h1>WORDLE</h1>
        <p>Vous avez 6 chances de deviner un mot de 5 lettres.</p>
        <router-link to="/game" class="play-button">Jouer</router-link>
      </div>
    </div>
  </div>
</template>

<script setup>
import Navbar from "@/components/Navbar.vue";
</script>

<style scoped>
.home-view {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.hero-section {
  flex: 1;
  background-image: url("@/assets/word-background.jpg");
  background-size: cover;
  background-position: center;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.content-box {
  width: 650px;
  height: 650px;
  background-color: white;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.logo {
  width: 200px;
  margin-bottom: 30px;
}

h1 {
  font-family: var(--font-family);
  font-size: 3.5rem;
  color: var(--dark-grey);
  margin-bottom: 20px;
}

p {
  font-family: var(--font-family);
  font-size: 1.2rem;
  color: var(--dark-grey);
  margin-bottom: 40px;
  text-align: center;
}

.play-button {
  background-color: var(--green);
  color: white;
  font-family: var(--font-family);
  font-size: 1.2rem;
  font-weight: 600;
  padding: 15px 50px;
  border-radius: 8px;
  text-decoration: none;
  transition: background-color 0.3s;
}

.play-button:hover {
  background-color: #2e6636;
}
</style>
