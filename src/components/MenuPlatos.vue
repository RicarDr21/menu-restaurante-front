<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const API_PLATOS = 'http://localhost:3001/api/platos'

const platos = ref([])
const cargando = ref(true)
const error = ref('')

const iconos = {
  entrada: '🥗',
  'plato fuerte': '🍖',
  sopa: '🍲',
  postre: '🍰'
}

function icono(categoria) {
  return iconos[categoria?.toLowerCase()] || '🍽️'
}

async function cargarPlatos() {
  try {
    const respuesta = await axios.get(API_PLATOS)
    platos.value = respuesta.data
  } catch (err) {
    error.value = 'No se pudieron cargar los platos.'
  } finally {
    cargando.value = false
  }
}

onMounted(cargarPlatos)
</script>

<template>
  <section class="menu">
    <div class="titulo-seccion">
      <h1>Nuestro Menú</h1>
      <p class="tagline">Sabores preparados con dedicación</p>
    </div>

    <p v-if="cargando" class="mensaje">Cargando platos...</p>
    <p v-if="error" class="mensaje error">{{ error }}</p>

    <div class="grid">
      <article class="plato" v-for="plato in platos" :key="plato.id">
        <div class="icono">{{ icono(plato.categoria) }}</div>
        <span class="etiqueta">{{ plato.categoria }}</span>
        <h3>{{ plato.nombre }}</h3>
        <p class="precio">${{ plato.precio.toLocaleString('es-CO') }}</p>
        <p class="chef">👨‍🍳 {{ plato.chef_nombre }}</p>
      </article>
    </div>
  </section>
</template>

<style scoped>
.menu {
  max-width: 1000px;
  margin: 0 auto;
  padding: 48px 20px 20px;
  text-align: left;
}
.titulo-seccion {
  text-align: center;
  margin-bottom: 32px;
}
.titulo-seccion h1 {
  font-size: 2.4rem;
  margin: 0;
  color: var(--acento-suave);
}
.tagline {
  color: var(--texto-muted);
  margin-top: 6px;
  font-style: italic;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(210px, 1fr));
  gap: 20px;
}

.plato {
  background: var(--superficie);
  border: 1px solid var(--borde);
  border-radius: 16px;
  padding: 20px;
  text-align: center;
  transition: transform 0.2s ease, border-color 0.2s ease;
}
.plato:hover {
  transform: translateY(-4px);
  border-color: var(--acento);
}
.icono {
  font-size: 2.2rem;
  margin-bottom: 6px;
}
.etiqueta {
  display: inline-block;
  background: rgba(224, 164, 88, 0.15);
  color: var(--acento-suave);
  font-size: 0.75rem;
  padding: 2px 10px;
  border-radius: 999px;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}
.plato h3 {
  margin: 4px 0;
  font-size: 1.15rem;
}
.precio {
  color: var(--acento);
  font-weight: 600;
  font-size: 1.1rem;
  margin: 6px 0;
}
.chef {
  color: var(--texto-muted);
  font-size: 0.85rem;
  margin: 0;
}
.mensaje {
  text-align: center;
  color: var(--texto-muted);
}
.error {
  color: var(--peligro);
}
</style>