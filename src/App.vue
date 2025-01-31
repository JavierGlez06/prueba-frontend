<template>
  <div :class="darkMode ? 'bg-dark text-white' : 'bg-light text-dark'">
    <div class="container mt-3">
      <h1 class="text-center">Usuarios</h1>
      <button @click="toggleDarkMode" class="btn btn-secondary mb-3">
        {{ darkMode ? 'Modo Claro' : 'Modo Oscuro' }}
      </button>
      <UsersTable />
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import UsersTable from './components/UsersTable.vue';

export default {
  components: { UsersTable },
  setup() {
    const darkMode = ref(false);

    const toggleDarkMode = () => {
      darkMode.value = !darkMode.value;
    };

    return { darkMode, toggleDarkMode };
  },
};
</script>

<style>
.bg-dark {
  background-color: #343a40 !important;
  color: white !important;
}

.bg-light {
  background-color: #f8f9fa !important;
  color: black !important;
}
</style>
