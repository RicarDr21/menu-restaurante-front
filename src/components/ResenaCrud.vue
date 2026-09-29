<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const API = 'http://localhost:3001/api/resenas'

const resenas = ref([])
const cargando = ref(false)
const error = ref('')

const formulario = ref({
  platoId: '',
  nombre: '',
  comentario: '',
  calificacion: 5
})

const editandoId = ref(null)

async function cargarResenas() {
  cargando.value = true
  error.value = ''
  try {
    const respuesta = await axios.get(API)
    resenas.value = respuesta.data
  } catch (err) {
    error.value = 'No se pudieron cargar las reseñas. Verifica que el backend esté corriendo.'
  } finally {
    cargando.value = false
  }
}

async function guardar() {
  try {
    if (editandoId.value) {
      await axios.put(`${API}/${editandoId.value}`, {
        nombre: formulario.value.nombre,
        comentario: formulario.value.comentario,
        calificacion: formulario.value.calificacion
      })
    } else {
      await axios.post(API, formulario.value)
    }
    limpiarFormulario()
    await cargarResenas()
  } catch (err) {
    error.value = 'No se pudo guardar la reseña.'
  }
}

function editar(resena) {
  editandoId.value = resena._id
  formulario.value = {
    platoId: resena.platoId,
    nombre: resena.nombre,
    comentario: resena.comentario,
    calificacion: resena.calificacion
  }
}

async function eliminar(id) {
  if (!confirm('¿Seguro que quieres eliminar esta reseña?')) return
  try {
    await axios.delete(`${API}/${id}`)
    await cargarResenas()
  } catch (err) {
    error.value = 'No se pudo eliminar la reseña.'
  }
}

function limpiarFormulario() {
  editandoId.value = null
  formulario.value = { platoId: '', nombre: '', comentario: '', calificacion: 5 }
}

function estrellas(n) {
  return '⭐'.repeat(n) + '☆'.repeat(5 - n)
}

onMounted(cargarResenas)
</script>

<template>
  <div class="pagina">
    <div class="titulo-seccion">
      <h2>Opiniones de nuestros clientes</h2>
      <p class="tagline">Comparte tu experiencia con cada plato</p>
    </div>

    <p v-if="error" class="alerta">{{ error }}</p>

    <div class="contenido">
      <form @submit.prevent="guardar" class="formulario">
        <h3>{{ editandoId ? 'Editar reseña' : 'Nueva reseña' }}</h3>

        <label>
          <span>Id del plato</span>
          <input v-model="formulario.platoId" type="number" required :disabled="!!editandoId" />
        </label>

        <label>
          <span>Nombre</span>
          <input v-model="formulario.nombre" type="text" required />
        </label>

        <label>
          <span>Comentario</span>
          <textarea v-model="formulario.comentario" required></textarea>
        </label>

        <label>
          <span>Calificación</span>
          <select v-model="formulario.calificacion">
            <option :value="5">5 - Excelente</option>
            <option :value="4">4 - Muy bueno</option>
            <option :value="3">3 - Bueno</option>
            <option :value="2">2 - Regular</option>
            <option :value="1">1 - Malo</option>
          </select>
        </label>

        <div class="acciones-form">
          <button type="submit" class="btn btn-primario">
            {{ editandoId ? 'Guardar cambios' : 'Crear reseña' }}
          </button>
          <button type="button" v-if="editandoId" class="btn btn-secundario" @click="limpiarFormulario">
            Cancelar
          </button>
        </div>
      </form>

      <section class="listado">
        <h3>Reseñas ({{ resenas.length }})</h3>
        <p v-if="cargando" class="mensaje">Cargando...</p>
        <p v-if="!cargando && resenas.length === 0" class="mensaje">No hay reseñas registradas.</p>

        <div class="tarjeta" v-for="resena in resenas" :key="resena._id">
          <div class="tarjeta-cabecera">
            <strong>{{ resena.nombre }}</strong>
            <span class="estrellas">{{ estrellas(resena.calificacion) }}</span>
          </div>
          <p class="tarjeta-plato">Plato #{{ resena.platoId }}</p>
          <p class="tarjeta-comentario">{{ resena.comentario }}</p>
          <div class="acciones-tarjeta">
            <button class="btn btn-editar" @click="editar(resena)">Editar</button>
            <button class="btn btn-eliminar" @click="eliminar(resena._id)">Eliminar</button>
          </div>
        </div>
      </section>
    </div>
  </div>
</template>

<style scoped>
.pagina {
  max-width: 1000px;
  margin: 0 auto;
  padding: 40px 20px 60px;
  text-align: left;
}
.titulo-seccion {
  text-align: center;
  margin-bottom: 28px;
}
.titulo-seccion h2 {
  font-size: 1.9rem;
  margin: 0;
  color: var(--acento-suave);
}
.tagline {
  color: var(--texto-muted);
  margin-top: 6px;
  font-style: italic;
}

.alerta {
  background: rgba(224, 92, 92, 0.15);
  color: #ff9a9a;
  padding: 10px 14px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: center;
}

.contenido {
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 28px;
}
@media (max-width: 720px) {
  .contenido {
    grid-template-columns: 1fr;
  }
}

.formulario {
  display: flex;
  flex-direction: column;
  gap: 14px;
  background: var(--superficie);
  border: 1px solid var(--borde);
  border-radius: 16px;
  padding: 22px;
  height: fit-content;
}
.formulario h3,
.listado h3 {
  margin: 0 0 4px;
  font-size: 1.1rem;
  color: var(--acento-suave);
}
.formulario label {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 0.85rem;
  color: var(--texto-muted);
}
.formulario input,
.formulario textarea,
.formulario select {
  background: #16120f;
  border: 1px solid var(--borde);
  border-radius: 8px;
  padding: 9px 11px;
  color: var(--texto);
  font-size: 0.95rem;
}
.formulario textarea {
  resize: vertical;
  min-height: 70px;
}

.acciones-form {
  display: flex;
  gap: 8px;
  margin-top: 6px;
}

.btn {
  border: none;
  border-radius: 8px;
  padding: 9px 16px;
  font-size: 0.9rem;
  cursor: pointer;
  transition: opacity 0.15s ease, transform 0.1s ease;
  font-family: inherit;
}
.btn:hover {
  opacity: 0.88;
  transform: translateY(-1px);
}
.btn-primario {
  background: var(--acento);
  color: #1a1310;
  font-weight: 600;
}
.btn-secundario {
  background: #332b26;
  color: var(--texto);
}
.btn-editar {
  background: var(--exito);
  color: white;
}
.btn-eliminar {
  background: var(--peligro);
  color: white;
}

.mensaje {
  color: var(--texto-muted);
}

.tarjeta {
  background: var(--superficie);
  border: 1px solid var(--borde);
  border-radius: 16px;
  padding: 18px 20px;
  margin-bottom: 14px;
  transition: border-color 0.15s ease;
}
.tarjeta:hover {
  border-color: var(--acento);
}
.tarjeta-cabecera {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.estrellas {
  font-size: 0.9rem;
}
.tarjeta-plato {
  color: var(--texto-muted);
  font-size: 0.85rem;
  margin: 4px 0 8px;
}
.tarjeta-comentario {
  margin: 0 0 12px;
}
.acciones-tarjeta {
  display: flex;
  gap: 8px;
}
</style>