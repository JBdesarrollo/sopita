<template>

      <div id="container">
        <div class="tabla">
          <table>
            <tbody>
            <tr v-for="(fila, rowIndex) in tabla" :key="rowIndex">
              <td
                v-for="(letra, colIndex) in fila"
                :key="colIndex"
                :style="{ backgroundColor: colores[rowIndex][colIndex] }"
                @click="cambiarColor(rowIndex, colIndex)"
              >
                {{ letra || 'X' }}
              </td>
            </tr>
            </tbody>
          </table>
        </div>
        <div id="lista">
          <ul>
            <li
              v-for="(palabra, index) in listaPalabras"
              :key="index"
              :style="{
    textDecoration: palabra.completada ? 'line-through' : 'none',
    color: palabra.completada ? 'red' : 'black',
    listStyle: 'none', fontSize: '3rem',
  }"
            >
              {{ palabra.text }}
            </li>
          </ul>
        </div>

      </div>

</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import { palabras } from '../diccionario.js';
const listaPalabras = ref([]); // Cambia esto por un array de objetos
const size = 20;
const direcciones = [
  { dx: 1, dy: 0 },  // Horizontal
  { dx: 0, dy: 1 },  // Vertical
  { dx: 1, dy: 1 },  // Diagonal (abajo derecha)
  { dx: -1, dy: 1 }, // Diagonal (arriba derecha)
  { dx: 1, dy: -1 }, // Diagonal (abajo izquierda)
  { dx: -1, dy: 0 }, // Horizontal (invertido)
  { dx: 0, dy: -1 }, // Vertical (invertido)
  { dx: -1, dy: -1 } // Diagonal (arriba izquierda)
];

const tabla = ref(Array.from({ length: size }, () => Array(size).fill('')));
const colores = ref(Array.from({ length: size }, () => Array(size).fill(''))); // Inicializa colores
// Lista que almacena las coordenadas de cada palabra
const palabrasCoordenadas = ref([]);
onMounted(() => {
  cargarDiccionario();
});

function cargarDiccionario() {
  const temp = [];
  while (temp.length < 20) {
    const randomIndex = Math.floor(Math.random() * palabras.length);
    const randomWord = palabras[randomIndex].toUpperCase();
    if (!temp.some(item => item.text === randomWord)) {
      temp.push({ text: randomWord, completada: false }); // Almacena el estado
      colocarPalabra(randomWord);
    }
  }
  listaPalabras.value = temp;
  // Rellenar los espacios vacíos con letras aleatorias
  llenarEspaciosConLetrasAleatorias();
}


function colocarPalabra(palabra) {
  const longitud = palabra.length;

  // Intentar colocar la palabra en la tabla
  for (let intentos = 0; intentos < 100; intentos++) {
    const direccion = direcciones[Math.floor(Math.random() * direcciones.length)];
    const startX = Math.floor(Math.random() * size);
    const startY = Math.floor(Math.random() * size);

    const endX = startX + direccion.dx * (longitud - 1);
    const endY = startY + direccion.dy * (longitud - 1);

    if (endX >= 0 && endX < size && endY >= 0 && endY < size) {
      let espacioLibre = true;
      const coordenadasPalabra = [];

      for (let i = 0; i < longitud; i++) {
        const x = startX + direccion.dx * i;
        const y = startY + direccion.dy * i;
        if (tabla.value[y][x] !== '' && tabla.value[y][x] !== palabra[i]) {
          espacioLibre = false;
          break;
        }
        coordenadasPalabra.push({ x, y }); // Guardamos las coordenadas de la palabra
      }

      if (espacioLibre) {
        for (let i = 0; i < longitud; i++) {
          const x = startX + direccion.dx * i;
          const y = startY + direccion.dy * i;
          tabla.value[y][x] = palabra[i];
        }
        palabrasCoordenadas.value.push(coordenadasPalabra); // Guardamos las coordenadas de la palabra
        return true; // Palabra colocada con éxito
      }
    }
  }
  return false; // No se pudo colocar la palabra
}

function cambiarColor(rowIndex, colIndex) {
  // Verificar si la celda ya está marcada como "completada" (amarillo)
  const letraSeleccionada = colores.value[rowIndex][colIndex];

  // Si ya está completada (amarillo), no hacemos nada
  if (letraSeleccionada === 'yellow') {
    return;
  }

  // Si la letra está seleccionada (lightgreen), la deseleccionamos
  if (letraSeleccionada === 'lightgreen') {
    colores.value[rowIndex][colIndex] = '';
  } else {
    // Si no está seleccionada, la seleccionamos
    colores.value[rowIndex][colIndex] = 'lightgreen';
  }

  verificarPalabraCompleta(); // Verificar si una palabra se ha completado
}


function verificarPalabraCompleta() {
  palabrasCoordenadas.value.forEach((coordenadas, index) => {
    const todasSeleccionadas = coordenadas.every(({ x, y }) => colores.value[y][x] === 'lightgreen');

    if (todasSeleccionadas) {
      coordenadas.forEach(({ x, y }) => {
        colores.value[y][x] = 'yellow'; // Cambia el color a amarillo
      });
      listaPalabras.value[index].completada = true; // Marca la palabra como completada
    }
  });
}

function letraAleatoria() {
  const letras = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  const indice = Math.floor(Math.random() * letras.length);
  return letras[indice];
}

function llenarEspaciosConLetrasAleatorias() {
  for (let rowIndex = 0; rowIndex < size; rowIndex++) {
    for (let colIndex = 0; colIndex < size; colIndex++) {
      if (tabla.value[rowIndex][colIndex] === '') {
        tabla.value[rowIndex][colIndex] = letraAleatoria();
      }
    }
  }
}

</script>

<style scoped>
#container {
  display: flex;
  flex-direction: row;
}

#sopa {
  width: 100%;
  background-color: burlywood;
  padding: 20px;
}

#lista {
  width: 100%;
  background-color: lavender;
  padding: 20px;
}

table {
  width: 40vw; /* Ajusta el tamaño en relación al ancho de la pantalla */
  height: 40vw; /* Forza la misma altura que el ancho para mantenerlo cuadrado */
  margin: auto;
  border: 1px solid black;
  aspect-ratio: 1 / 1; /* Mantén la relación de aspecto 1:1 */
}

td {
  width: calc(40vw / 20); /* Asegúrate de que cada celda tenga una relación constante */
  height: calc(40vw / 20);
  border: 1px solid #ccc;
  text-align: center;
  vertical-align: middle;
  cursor: pointer; /* Cambia el cursor al pasar sobre la celda */
  font-size: 2rem;
  font-weight: bold;
}

.tabla {
  margin-top: 40px;
  padding: 30px;
}

@media (max-width: 768px) {
  #container {
    flex-direction: column;
  }

  #sopa {
    width: 100%;
    background-color: burlywood;
    padding: 5px;
  }

  #lista {
    width: 100%;
    background-color: lavender;
    padding: 5px;
  }
  li{
    font-size: 1rem !important;
  }
  table {
    width: 90vw; /* Usa el 90% del ancho de la vista para la tabla */
    height: 90vw; /* Asegúrate de que la tabla tenga la misma altura que el ancho */
    margin: auto;
    border: 1px solid black;
    aspect-ratio: 1 / 1; /* Mantén la relación de aspecto 1:1 */
  }

  td {
    width: calc(90vw / 20); /* Ajusta el tamaño de las celdas según el tamaño de la tabla */
    height: calc(90vw / 20);
    font-size: 1rem;
    font-weight: bold;

  }

  .tabla {
    padding: 5px;
    margin-bottom: 10px;
  }
}

</style>
