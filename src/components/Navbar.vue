<template>
  <nav class="navbar">
    <div class="navbar-content">
      <router-link to="/" class="logo">
        <img src="/images/wordle-logo.png" alt="wordle-logo" />
        <span>WORDLE</span>
      </router-link>
      <div class="nav-right">
        <select
          v-model="selectedLanguage"
          @change="changeLanguage"
          class="language-select"
        >
          <option value="en">English</option>
          <option value="fr">Français</option>
          <option value="es">Español</option>
          <option value="it">Italiano</option>
          <option value="de">Deutsch</option>
        </select>

        <button @click="toggleDarkMode" class="theme-toggle">
          <svg
            v-if="isDarkMode"
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <circle cx="12" cy="12" r="5"></circle>
            <line x1="12" y1="1" x2="12" y2="3"></line>
            <line x1="12" y1="21" x2="12" y2="23"></line>
            <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
            <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
            <line x1="1" y1="12" x2="3" y2="12"></line>
            <line x1="21" y1="12" x2="23" y2="12"></line>
            <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
            <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
          </svg>
          <svg
            v-else
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
          </svg>
        </button>
        <router-link to="/" class="home-link">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
            <polyline points="9 22 9 12 15 12 15 22"></polyline>
          </svg>
        </router-link>
      </div>
    </div>
  </nav>
</template>

<script setup>
import { ref, onMounted } from "vue";

const isDarkMode = ref(false);
const selectedLanguage = ref(localStorage.getItem("wordleLanguage") || "en");

// Load saved preferences
onMounted(() => {
  const savedMode = localStorage.getItem("darkMode");
  if (savedMode) {
    isDarkMode.value = savedMode === "true";
    updateTheme();
  }

  const savedLang = localStorage.getItem("wordleLanguage");
  if (savedLang) {
    selectedLanguage.value = savedLang;
  }
});

// Toggle function
const toggleDarkMode = () => {
  isDarkMode.value = !isDarkMode.value;
  localStorage.setItem("darkMode", isDarkMode.value);
  updateTheme();
};

// Change language function
const changeLanguage = () => {
  localStorage.setItem("wordleLanguage", selectedLanguage.value);
  window.dispatchEvent(
    new CustomEvent("languageChanged", {
      detail: { language: selectedLanguage.value },
    })
  );
};

// Apply theme changes
const updateTheme = () => {
  if (isDarkMode.value) {
    document.documentElement.classList.add("dark");
  } else {
    document.documentElement.classList.remove("dark");
  }
};
</script>

<style scoped>
.navbar {
  background-color: var(--dark-grey);
  padding: 1rem 2rem;
  border-bottom: 1px solid var(--light-grey);
}

.navbar-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1200px;
  margin: 0 auto;
}

.logo {
  font-family: var(--font-family);
  font-weight: 700;
  font-size: 1.5rem;
  color: var(--white);
  text-decoration: none;
  display: flex;
  align-items: center;
}

.logo img {
  width: 30px;
  margin-right: 10px;
}

.logo:hover {
  color: var(--orange);
}

.nav-right {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.theme-toggle {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--white);
  padding: 0.5rem;
  display: flex;
  align-items: center;
}

.theme-toggle:hover {
  color: var(--orange);
}

.home-link {
  color: var(--white);
  transition: color 0.2s;
}

.home-link:hover {
  color: var(--orange);
}

.home-link svg {
  display: block;
}

.language-select {
  padding: 0.5rem 0.8rem;
  border-radius: 4px;
  border: 1px solid var(--light-grey);
  background-color: var(--dark-grey);
  color: var(--white);
  font-family: var(--font-family);
  cursor: pointer;
}

.language-select:focus {
  outline: none;
  border-color: var(--orange);
}

/* Dark mode styles */
.dark .navbar {
  background-color: var(--white);
  border-bottom-color: var(--dark-grey);
}

.dark .logo,
.dark .home-link {
  color: var(--dark-grey);
}

.dark .theme-toggle {
  color: var(--dark-grey);
}

.dark .language-select {
  background-color: var(--white);
  color: var(--dark-grey);
}
</style>
