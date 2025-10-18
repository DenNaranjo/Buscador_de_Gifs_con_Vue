<script setup>
// Importamos 'ref' desde Vue para crear variables reactivas.
// Una variable reactiva es aquella que, cuando su valor cambia, actualiza automáticamente la vista (el HTML)
import { ref } from 'vue';

// --- ESTADO DE LA APLICACIÓN ---
// Aquí definimos todas las variables que nuestra aplicación necesita para funcionar.

// 'searchTerm' almacena el texto que el usuario escribe en la barra de búsqueda.
const searchTerm = ref('');
// 'gifs' es un array que guardará la lista de GIFs obtenidos de la API.
const gifs = ref([]);
// 'isLoading' es un booleano (true/false) que nos dice si estamos esperando una respuesta de la API.
// Lo usamos para mostrar un indicador de carga.
const isLoading = ref(false);
// 'error' almacenará cualquier mensaje de error que queramos mostrar al usuario
const error = ref(null);
// Aqui puse mi key personal para acceder a la API creada de Giphy.
const giphyApiKey = 'R9vwapLiWJhftgmBNjWGLlDKPpQgGsOB';
// --- LÓGICA DE BÚSQUEDA ---
// 'async function' nos permite usar 'await', que pausa la ejecución de la función
// hasta que una promesa (como la llamada a la API) se resuelva.
async function searchGifs() {
  // Si el usuario intenta buscar sin haber escrito nada (o solo espacios), no hacemos nada.
  if (searchTerm.value.trim() === '') return;

  // Inicia el proceso de búsqueda: activamos el indicador de carga.
  isLoading.value = true;
  // Limpiamos cualquier error anterior.
  error.value = null;
  // Vaciamos la lista de GIFs para mostrar los nuevos resultados.
  gifs.value = [];

  // Usamos un bloque try/catch para manejar posibles errores durante la llamada a la API.
  try {
    // 'fetch' realiza la petición a la URL de Giphy.
    // Usamos `encodeURIComponent` para asegurarnos de que el texto de búsqueda se envíe correctamente en la URL.
    const response = await fetch(`https://api.giphy.com/v1/gifs/search?api_key=${giphyApiKey}&q=${encodeURIComponent(searchTerm.value)}&limit=24&offset=0&rating=g&lang=es`);
    
    // Si la respuesta de la red no fue exitosa (ej: error 404, 401 por API key inválida),
    // lanzamos un error para que sea capturado por el bloque 'catch'.
    if (!response.ok) {
      throw new Error('La respuesta del servidor no fue exitosa. Revisa tu API Key.');
    }

    // Convertimos la respuesta de la API, que está en formato JSON, a un objeto de JavaScript.
    const data = await response.json();
    
    // Si la búsqueda no devolvió ningún GIF...
    if (data.data.length === 0) {
      // ...guardamos un mensaje de error para mostrarlo al usuario.
      error.value = `No se encontraron GIFs para "${searchTerm.value}". Intenta con otra búsqueda.`;
    } else {
      // Si se encontraron GIFs, los guardamos en nuestra variable reactiva 'gifs'.
      // Vue actualizará automáticamente la vista para mostrarlos.
      gifs.value = data.data;
    }

  } catch (err) {
    // Si ocurre cualquier error en el bloque 'try' (problema de red, API key incorrecta, etc.)...
    // ...lo mostramos en la consola para depuración.
    console.error('Error al obtener los GIFs:', err);
    // Y guardamos un mensaje de error genérico para el usuario.
    error.value = 'Ocurrió un error. No se pudo obtener una respuesta del servidor. Revisa tu conexión o la API key.';
  } finally {
    // El bloque 'finally' se ejecuta SIEMPRE, tanto si la búsqueda fue exitosa como si falló.
    // Es el lugar perfecto para desactivar el indicador de carga.
    isLoading.value = false;
  }
}
</script>

<template>
  <div class="min-h-screen text-white font-sans flex flex-col items-center p-4 sm:p-6 md:p-8">
    <header class="w-full max-w-4xl text-center mb-8">
      <h1 class="text-4xl sm:text-5xl md:text-6xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-purple-400 to-teal-400">
        Buscador de GIFs con Vue.js
      </h1>
      <p class="text-lg text-gray-400 mt-2">
        Creado con Vue 3, Vite y Tailwind CSS
      </p>
    </header>

    <main class="w-full max-w-4xl">
      <form @submit.prevent="searchGifs" class="flex flex-col sm:flex-row gap-3 mb-8">
        <input 
          type="text"
          v-model="searchTerm"
          placeholder="Busca osos, gatos, memes..."
          class="flex-grow bg-gray-700 border-2 border-gray-600 rounded-lg px-4 py-3 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:border-transparent transition-shadow duration-300"
        />
        <button 
          type="submit"
          class="bg-gradient-to-r from-purple-500 to-teal-500 hover:from-purple-600 hover:to-teal-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg hover:shadow-xl transform hover:-translate-y-1 transition-all duration-300"
        >
          <span v-if="!isLoading">Buscar</span>
          <span v-else>Buscando...</span>
        </button>
      </form>

      <!-- Mensaje de Error -->
      <div v-if="error" class="bg-red-900 border border-red-700 text-red-200 px-4 py-3 rounded-lg text-center mb-8">
        {{ error }}
      </div>
      
      <!-- Indicador de Carga -->
      <div v-if="isLoading" class="text-center">
        <svg class="animate-spin h-10 w-10 text-white mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
      </div>

      <!-- Galería de GIFs -->
      <div v-if="gifs.length > 0" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-4">
        <div v-for="gif in gifs" :key="gif.id" class="relative group aspect-square">
          <img :src="gif.images.fixed_width.url" :alt="gif.title" class="w-full h-full object-cover rounded-lg shadow-md transition-transform duration-300 group-hover:scale-105" />
          <div class="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity duration-300 rounded-lg">
            <a :href="gif.url" target="_blank" class="text-white text-center text-sm p-2">Ver en Giphy</a>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

