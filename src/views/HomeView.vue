<template>
  <div class="home-view">
    <div class="hero-section">
      <div class="content-box">
        <img src="/images/wordle-logo.png" class="logo" />
        <h1>Wordle</h1>
        <p>Vous avez 6 chances de deviner un mot de 5 lettres.</p>

        <Button
          size="medium"
          customClass="primary"
          @click="$router.push('/game')"
        >
          Jouer
        </Button>
      </div>
    </div>
  </div>
</template>

<script setup>
import Button from "@/components/Button.vue";
</script>

<style scoped>
.home-view {
  height: 100vh;
  display: flex;
  flex-direction: column;
}

.hero-section {
  flex: 1;
  background-image: url("/images/word-background.jpg");
  background-size: cover;
  background-position: center;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.content-box {
  background-color: white;
  width: 30rem;
  height: 30rem;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.logo {
  width: 100px;
  margin-bottom: 30px;
}

h1 {
  font-family: var(--font-family);
  font-size: 3.5rem;
  color: var(--dark-grey);
  margin-bottom: 20px;
  font-weight: 800;
}

p {
  font-family: var(--font-family);
  font-size: 1.2rem;
  color: var(--dark-grey);
  margin-bottom: 40px;
  text-align: center;
}
</style>
