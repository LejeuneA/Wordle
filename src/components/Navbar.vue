<template>
  <nav class="navbar">
    <div class="navbar-content">
      <router-link to="/" class="logo">
        <img src="/images/wordle-logo.png" alt="wordle-logo" />
        <span>WORDLE</span>
      </router-link>
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
  </nav>
</template>

<script setup>
// No props needed for this simple navbar
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

  img {
    width: 30px;
    margin-right: 10px;
  }
}

.logo:hover {
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
</style>
