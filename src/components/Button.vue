<template>
  <button :class="['btn', size, customClass]" @click="onClick">
    <slot></slot>
  </button>
</template>

<script>
export default {
  name: "Button",
  props: {
    size: {
      type: String,
      default: "medium",
      validator: (value) => ["small", "medium", "large"].includes(value),
    },
    customClass: {
      type: String,
      default: "",
    },
  },
  methods: {
    onClick() {
      this.$emit("click");
    },
  },
};
</script>

<style scoped>
.btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 5px;
  font-family: var(--font-family);
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease;

  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  display: inline-block;
  text-align: center;
  text-decoration: none;
  color: var(--white);
  background-color: var(--green);
  font-size: 1rem;
  margin: 0.5rem;
  width: auto;
  min-width: 100px;
  max-width: 100%;
  box-sizing: border-box;
  outline: none;
  user-select: none;
}

.btn.small {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}

.btn.medium {
  padding: 0.5rem 1rem;
  font-size: 1rem;
}

.btn.large {
  padding: 0.75rem 1.5rem;
  font-size: 1.25rem;
}

.btn.primary {
  background-color: var(--green);
  color: var(--white);
}

.btn.secondary {
  background-color: var(--orange);
  color: var(--white);
}

.btn:hover {
  opacity: 0.9;
}
</style>
