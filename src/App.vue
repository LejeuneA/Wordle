<template>
  <div id="app">
    <Navbar />
    <router-view />
  </div>
</template>

<script>
import Button from "./components/Button.vue";
import Navbar from "./components/Navbar.vue";
import "./assets/main.css";

export default {
  name: "App",
  components: {
    Navbar,
    Button,
  },
};
</script>

<style></style>
