<script setup>
import { ref, computed } from 'vue';

// --- aqui van las variantes reactivas (5 requeridas) ---
const inputTexto = ref('');        // Captura la entrada del usuario
const formatoDestino = ref('csv'); // Controla el formato de salida
const errorMensaje = ref('');      // Almacena errores de validación
const historial = ref([]);         // Guarda las últimas conversiones exitosas
const mostrarHistorial = ref(true); // Controla la visibilidad del panel

// --- LÓGICA DE CONVERSIÓN Y VALIDACIÓN ---
const procesarDatos = () => {
  // Validación de datos vacíos (Punto 6)
  if (!inputTexto.value.trim()) {
    errorMensaje.value = "Error: El campo de entrada no puede estar vacío.";
    return;
  }

  try {
    // Intentamos parsear el JSON para validar que el formato sea correcto
    const objetoJson = JSON.parse(inputTexto.value);
    errorMensaje.value = ""; // Limpiamos errores si el JSON es válido

    let resultado = "";
    if (formatoDestino.value === 'csv') {
      resultado = jsonToCsv(objetoJson);
    } else {
      resultado = JSON.stringify(objetoJson, null, 4); // Formateado Prettify
    }

    // Guardar en el historial (Uso de v-for más adelante)
    historial.value.unshift({
      id: Date.now(),
      fecha: new Date().toLocaleTimeString(),
      tipo: formatoDestino.value.toUpperCase()
    });

    // Alertar éxito
    alert("Conversión exitosa y añadida al historial.");

  } catch (e) {
    errorMensaje.value = "Error de Formato: El texto ingresado no es un JSON válido.";
  }
};

// Función auxiliar para convertir JSON a CSV
const jsonToCsv = (items) => {
  if (!Array.isArray(items)) return "Error: Para CSV, el JSON debe ser una lista de objetos [{},{}].";
  const cabecera = Object.keys(items[0]);
  const filas = items.map(obj => cabecera.map(nombre => JSON.stringify(obj[nombre])).join(','));
  return [cabecera.join(','), ...filas].join('\r\n');
};

// Función para limpiar los campos (Nueva variable reactiva no es necesaria, usamos las existentes)
const limpiarCampos = () => {
  inputTexto.value = "";
  errorMensaje.value = "";
  alert("Formulario limpiado");
};
</script>




<template>
  <div id="app" style="font-family: sans-serif; max-width: 800px; margin: 0 auto; padding: 20px;">
    
    <h1>Transformador de Datos Pro</h1>
    <p>Copia tu JSON y conviértelo rápidamente a CSV o JSON formateado.</p>

    <hr>

    <section>
      <label>Selecciona el formato de salida:</label>
      <select v-model="formatoDestino" style="margin-left: 10px; padding: 5px;">
        <option value="csv">CSV (Excel)</option>
        <option value="pretty">JSON Formateado</option>
      </select>

      <br><br>

      <textarea 
        v-model="inputTexto" 
        placeholder='Pega tu JSON aquí... ej: [{"id":1, "nombre":"Ariel"}]'
        style="width: 100%; height: 150px; border-radius: 8px; padding: 10px;"
        v-bind:style="{ borderColor: errorMensaje ? 'red' : '#ccc' }"
      ></textarea> <p v-if="errorMensaje" style="color: red; font-weight: bold;">
        {{ errorMensaje }}
      </p>

      <br>

      <button @click="procesarDatos" style="padding: 10px 20px; cursor: pointer; background-color: #42b883; color: white; border: none; border-radius: 5px;">
        Procesar y Validar Datos
      </button>

<button @click="limpiarCampos" style="padding: 10px 20px; cursor: pointer; background-color: #ff4d4d; color: white; border: none; border-radius: 5px;">
  Limpiar Campos
</button>

    </section>

    <hr>

    <section>
      <button @click="mostrarHistorial = !mostrarHistorial">
        {{ mostrarHistorial ? 'Ocultar Historial' : 'Mostrar Historial' }}
      </button>

<div v-if="mostrarHistorial" class="historial-section">
    <h3>Historial de conversiones</h3>
        <p v-if="historial.length === 0">No hay conversiones registradas aún.</p>
        
        <ul>
          <li v-for="item in historial" :key="item.id">
            Se procesó un archivo <strong>{{ item.tipo }}</strong> a las {{ item.fecha }}
          </li>
        </ul>
      </div>
    </section>

  </div>

</template>





<style scoped>


#app {
  background-color: #f4f7f6; /* Un gris muy claro azulado */
  border: 2px solid #34495e;  /* Marco sólido color azul oscuro */
  border-radius: 15px;       /* Bordes redondeados del marco */
  padding: 30px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.1); /* Sombra para dar profundidad */
  margin-top: 50px !important;
}

/* Estilo para los títulos */
h1 {
  color: #2c3e50;
  text-align: center;
  border-bottom: 2px solid #42b883; /* Línea decorativa bajo el título */
  padding-bottom: 10px;
}

/* Estilo del marco del área de texto (Input) */
textarea {
  font-family: 'Courier New', Courier, monospace;
  background-color: #ffffff;
  border: 1.5px solid #bdc3c7;
  transition: all 0.3s ease; /* Suaviza el cambio de color al error */
}

/* Estilo para los botones */
button {
  font-weight: bold;
  text-transform: uppercase;
  transition: transform 0.2s;
}

button:hover {
  transform: scale(1.02); /* Efecto de que el botón resalta al pasar el mouse */
}

/* Marco para la sección del historial */
.historial-section {
  background-color: #ffffff;
  border-left: 5px solid #42b883; /* Marco lateral tipo "nota" */
  padding: 15px;
  margin-top: 20px;
  border-radius: 5px;
}
</style>