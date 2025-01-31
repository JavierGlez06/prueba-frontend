<template>
  <div :class="{ dark: darkMode }">

    <!-- Barra de búsqueda -->
    <input
      type="text"
      v-model="searchQuery"
      class="form-control mb-3"
      placeholder="Buscar por nombre..."
    />

    <!-- Mensaje de error si la API falla -->
    <div v-if="errorMessage" class="alert alert-danger">{{ errorMessage }}</div>

    <!-- Tabla de usuarios -->
    <table class="table table-bordered table-striped">
      <thead>
        <tr>
          <th v-if="columns.id">#</th>
          <th v-if="columns.name">Nombre Completo</th>
          <th v-if="columns.phone">Teléfono</th>
          <th v-if="columns.email">Correo</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(user, index) in paginatedUsers" :key="user.id">
          <td v-if="columns.id">{{ index + 1 + (currentPage - 1) * itemsPerPage }}</td>
          <td v-if="columns.name">{{ user.name }}</td>
          <td v-if="columns.phone">{{ user.phone }}</td>
          <td v-if="columns.email">{{ user.email }}</td>
          <td>
            <button @click="removeColumn('email')" class="btn btn-warning btn-sm">Eliminar Email</button>
            <button @click="removeColumn('phone')" class="btn btn-warning btn-sm">Eliminar Teléfono</button>
            <button @click="openCamera(user.id)" class="btn btn-primary btn-sm">Tomar Foto</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Paginación -->
    <nav>
      <ul class="pagination">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <button class="page-link" @click="prevPage">Anterior</button>
        </li>
        <li class="page-item" v-for="page in totalPages" :key="page" :class="{ active: page === currentPage }">
          <button class="page-link" @click="setPage(page)">{{ page }}</button>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <button class="page-link" @click="nextPage">Siguiente</button>
        </li>
      </ul>
    </nav>

    <!-- Botón para restaurar columnas -->
    <button @click="restoreColumns" class="btn btn-info mt-2">Restaurar Columnas</button>

    <!-- Componente de Cámara -->
    <CameraComponent v-if="cameraActive" @close="cameraActive = false" />
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';
import CameraComponent from './CameraComponent.vue';

export default {
  components: { CameraComponent },
  setup() {
    const users = ref([]);
    const searchQuery = ref('');
    const cameraActive = ref(false);
    const errorMessage = ref('');
    const darkMode = ref(localStorage.getItem('darkMode') === 'true');

    const columns = ref({
      id: true,
      name: true,
      phone: true,
      email: true,
    });

    const fetchUsers = async () => {
      try {
        const response = await axios.get('https://www.4sides.com.mx/api/prueba-tecnica/usuarios/index?results=5');
        
        // Verificamos si 'usuarios' está presente en la respuesta
        if (response.data && response.data.usuarios) {
          users.value = response.data.usuarios.map((user, index) => ({
            id: index + 1,
            name: user.usuarioNombre + ' ' + user.usuarioApellidoPaterno + ' ' + user.usuarioApellidoMaterno,
            phone: user.usuarioTelefono,
            email: user.usuarioEmail,
          }));
        } else {
          errorMessage.value = 'La respuesta de la API no contiene los datos esperados.';
          console.error('La respuesta de la API no contiene los datos esperados:', response.data);
        }
      } catch (error) {
        errorMessage.value = `Error al cargar los usuarios: ${error.message}`;
        console.error('Detalles del error:', error);
      }
    };

    const filteredUsers = computed(() => {
      return users.value.filter((user) =>
        user.name.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    });

    // Paginación
    const currentPage = ref(1);
    const itemsPerPage = 10;
    const totalPages = computed(() => Math.ceil(filteredUsers.value.length / itemsPerPage));

    const paginatedUsers = computed(() => {
      const start = (currentPage.value - 1) * itemsPerPage;
      return filteredUsers.value.slice(start, start + itemsPerPage);
    });

    const nextPage = () => {
      if (currentPage.value < totalPages.value) currentPage.value++;
    };
    const prevPage = () => {
      if (currentPage.value > 1) currentPage.value--;
    };
    const setPage = (page) => {
      currentPage.value = page;
    };

    // Eliminar columnas
    const removeColumn = (column) => {
      columns.value[column] = false;
    };

    // Restaurar columnas
    const restoreColumns = () => {
      columns.value = { id: true, name: true, phone: true, email: true };
    };

    // Cámara
    const openCamera = () => {
      cameraActive.value = true;
    };

    // Modo oscuro
    const toggleDarkMode = () => {
      darkMode.value = !darkMode.value;
      localStorage.setItem('darkMode', darkMode.value);
    };

    onMounted(fetchUsers);

    return {
      searchQuery,
      filteredUsers,
      paginatedUsers,
      removeColumn,
      restoreColumns,
      openCamera,
      cameraActive,
      columns,
      darkMode,
      toggleDarkMode,
      errorMessage,
      currentPage,
      totalPages,
      nextPage,
      prevPage,
      setPage,
      itemsPerPage,
    };
  },
};
</script>
