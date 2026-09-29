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

onMounted(cargarResenas)
</script>

<template>
  <div class="pagina">
    <header class="encabezado">
      <h1>Reseñas del restaurante</h1>
      <p class="subtitulo">Gestiona las opiniones de tus clientes sobre cada plato</p>
    </header>

    <p v-if="error" class="alerta">{{ error }}</p>

    <div class="contenido">
      <form @submit.prevent="guardar" class="formulario">
        <h2>{{ editandoId ? 'Editar reseña' : 'Nueva reseña' }}</h2>

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
        <h2>Listado ({{ resenas.length }})</h2>
        <p v-if="cargando" class="mensaje">Cargando...</p>
        <p v-if="!cargando && resenas.length === 0" class="mensaje">No hay reseñas registradas.</p>

        <div class="tarjeta" v-for="resena in resenas" :key="resena._id">
          <div class="tarjeta-cabecera">
            <strong>{{ resena.nombre }}</strong>
            <span class="badge">{{ resena.calificacion }}/5 ⭐</span>
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
  max-width: 960px;
  margin: 0 auto;
  padding: 32px 20px;
  font-family: system-ui, -apple-system, Segoe UI, sans-serif;
  text-align: left;
}

.encabezado {
  margin-bottom: 28px;
}
.encabezado h1 {
  margin: 0;
  font-size: 2rem;
}
.subtitulo {
  color: #888;
  margin-top: 4px;
}

.alerta {
  background: #3a1f1f;
  color: #ff8a8a;
  padding: 10px 14px;
  border-radius: 8px;
  margin-bottom: 20px;
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
  background: #1e1e1e;
  border: 1px solid #333;
  border-radius: 12px;
  padding: 20px;
  height: fit-content;
}
.formulario h2,
.listado h2 {
  margin: 0 0 4px;
  font-size: 1.1rem;
}
.formulario label {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 0.85rem;
  color: #bbb;
}
.formulario input,
.formulario textarea,
.formulario select {
  background: #2a2a2a;
  border: 1px solid #444;
  border-radius: 8px;
  padding: 8px 10px;
  color: inherit;
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
  transition: opacity 0.15s ease;
}
.btn:hover {
  opacity: 0.85;
}
.btn-primario {
  background: #4f8cff;
  color: white;
}
.btn-secundario {
  background: #3a3a3a;
  color: #ddd;
}
.btn-editar {
  background: #2f9e6e;
  color: white;
}
.btn-eliminar {
  background: #d9534f;
  color: white;
}

.mensaje {
  color: #888;
}

.tarjeta {
  background: #1e1e1e;
  border: 1px solid #333;
  border-radius: 12px;
  padding: 16px 18px;
  margin-bottom: 14px;
  transition: transform 0.15s ease, border-color 0.15s ease;
}
.tarjeta:hover {
  transform: translateY(-2px);
  border-color: #4f8cff;
}

.tarjeta-cabecera {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.badge {
  background: #2a2a2a;
  border-radius: 999px;
  padding: 3px 10px;
  font-size: 0.85rem;
}
.tarjeta-plato {
  color: #888;
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