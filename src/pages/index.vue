<template>
  <v-app-bar :elevation="2">
    <v-app-bar-title>Sopita instantánea</v-app-bar-title>
  </v-app-bar>
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
            listStyle: 'none',
            fontSize: '3rem'
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

const listaPalabras = ref([]);
const size = 20;
const direcciones = [
  { dx: 1, dy: 0 },  // Horizontal
  { dx: 0, dy: 1 },  // Vertical
  { dx: 1, dy: 1 },  // Diagonal (abajo derecha)
  { dx: -1, dy: 1 }  // Diagonal (arriba derecha)
];
const tabla = ref(Array.from({ length: size }, () => Array(size).fill('')));
const colores = ref(Array.from({ length: size }, () => Array(size).fill('')));
const palabrasCoordenadas = ref([]);

onMounted(() => {
  cargarDiccionario();
});

function cargarDiccionario() {
  const temp = [];
  while (temp.length < 15) {
    const randomIndex = Math.floor(Math.random() * palabras.length);
    const randomWord = palabras[randomIndex].toUpperCase();
    if (!temp.some(item => item.text === randomWord)) {
      temp.push({ text: randomWord, completada: false });
      colocarPalabraSinCruces(randomWord);
    }
  }
  listaPalabras.value = temp;
  llenarEspaciosConLetrasAleatorias();
}

function colocarPalabraSinCruces(palabra) {
  const longitud = palabra.length;
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
        if (tabla.value[y][x] !== '') {
          espacioLibre = false;
          break;
        }
        coordenadasPalabra.push({ x, y });
      }
      if (espacioLibre) {
        for (let i = 0; i < longitud; i++) {
          const x = startX + direccion.dx * i;
          const y = startY + direccion.dy * i;
          tabla.value[y][x] = palabra[i];
        }
        palabrasCoordenadas.value.push(coordenadasPalabra);
        return true;
      }
    }
  }
  return false;
}

function cambiarColor(rowIndex, colIndex) {
  const letraSeleccionada = colores.value[rowIndex][colIndex];

  if (letraSeleccionada === 'yellow') {
    return;
  }

  if (letraSeleccionada === 'lightgreen') {
    colores.value[rowIndex][colIndex] = '';
  } else {
    colores.value[rowIndex][colIndex] = 'lightgreen';
  }

  verificarPalabraCompleta();
}

function verificarPalabraCompleta() {
  palabrasCoordenadas.value.forEach((coordenadas, index) => {
    const todasSeleccionadas = coordenadas.every(({ x, y }) => colores.value[y][x] === 'lightgreen');
    if (todasSeleccionadas) {
      coordenadas.forEach(({ x, y }) => {
        colores.value[y][x] = 'yellow';
      });
      listaPalabras.value[index].completada = true;
    }
  });
}

function letraAleatoria() {
  const letras = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  const indice = Math.floor(Math.random() * letras.length);
  return letras[indice];
}

function llenarEspaciosConLetrasAleatorias() {
  /*for (let rowIndex = 0; rowIndex < size; rowIndex++) {
    for (let colIndex = 0; colIndex < size; colIndex++) {
      if (tabla.value[rowIndex][colIndex] === '') {
        tabla.value[rowIndex][colIndex] = letraAleatoria();
      }
    }
  }*/
  return;
}
</script>

<style scoped>
#container {
  display: flex;
  flex-direction: row;
  background-color: white;
  color: black;
  font-weight: bolder;
}
#lista {
  width: 100%;
  background-color: white;
  padding: 20px;
}
table {
  width: 40vw;
  height: 40vw;
  margin: auto;
  border: 1px solid black;
  aspect-ratio: 1 / 1;
}
td {
  width: calc(40vw / 20);
  height: calc(40vw / 20);
  border: 1px solid #ccc;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  font-size: 2rem;
  font-weight: bold;
}
#lista {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
#lista ul {
  margin: auto;
}
.tabla {
  margin-top: 40px;
  padding: 30px;
}
@media (max-width: 768px) {
  #container {
    flex-direction: column;
  }
  #lista {
    width: 100%;
    padding: 5px;
    padding-bottom: 30px;
  }
  li {
    font-size: 1rem !important;
  }
  .tabla {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 10px;
  }
  table {
    width: 90vw;
    height: 90vw;
    max-height: 90vh;
    margin: auto;
    border: 1px solid black;
  }
  td {
    width: calc(90vw / 20);
    height: calc(90vw / 20);
    font-size: 1rem;
    font-weight: bold;
  }
}
</style>
