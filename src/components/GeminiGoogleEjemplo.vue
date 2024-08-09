<template>
  <div class="container mt-5">
    <div class="text-center mb-4">
      <h1>Gemini-1.5-flash , Gemini de Google</h1>
    </div>
    <div class="row justify-content-center">
      <div class="col-md-6">
        <div class="card p-4 shadow-sm">
          <h2 class="card-title mb-4 text-center">Generador de Contenido</h2>
          <div class="mb-3">
            <input
              v-model="userInput"
              @keyup.enter="generateContent"
              class="form-control"
              placeholder="Escribe tu prompt aquí"
            />
          </div>
          <button @click="generateContent" class="btn btn-primary w-100">
            Generar
          </button>
          <div v-if="isLoading" class="d-flex justify-content-center mt-4">
            <div class="spinner-border" role="status">
              <span class="visually-hidden">Cargando...</span>
            </div>
          </div>
          <div v-if="generatedText && !isLoading" class="mt-4">
            <h3 class="text-center">Resultado:</h3>
            <div class="border p-3 rounded bg-light" v-html="generatedText"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { GoogleGenerativeAI } from '@google/generative-ai';
import 'bootstrap/dist/css/bootstrap.min.css';

// Inicializa la instancia de GoogleGenerativeAI
const genAI = new GoogleGenerativeAI(import.meta.env.VITE_API_KEY_GEMINI_GOOGLE); // Asegúrate de que tu API key esté configurada en el archivo .env

const model = genAI.getGenerativeModel({ model: 'gemini-1.5-flash' });

const userInput = ref('');
const generatedText = ref('');
const isLoading = ref(false);

const processGeneratedText = (text) => {
  let formattedText = text
    .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>') // Negritas
    .replace(/\*(.*?)\*/g, '<em>$1</em>') // Cursivas
    .replace(/\n/g, '<br>'); // Saltos de línea
  
  return formattedText;
};

const generateContent = async () => {
  if (userInput.value.trim() === '') {
    generatedText.value = 'Por favor, introduce un texto.';
    return;
  }

  isLoading.value = true;
  generatedText.value = ''; // Limpiar el texto generado antes de mostrar el spinner

  try {
    const result = await model.generateContent(userInput.value);
    const response = await result.response;
    let text = await response.text();
    generatedText.value = processGeneratedText(text); // Procesa el texto antes de asignarlo
  } catch (error) {
    console.error('Error generando contenido:', error);
    generatedText.value = 'Hubo un error al generar el contenido.';
  } finally {
    isLoading.value = false;
  }
};
</script>

<style scoped>
/* Añade estilos adicionales si es necesario */
</style>
