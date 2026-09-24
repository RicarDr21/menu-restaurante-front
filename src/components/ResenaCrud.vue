<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

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
    const respuesta = await axios.get('/api/resenas')
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
      await axios.put(`/api/resenas/${editandoId.value}`, {
        nombre: formulario.value.nombre,
        comentario: formulario.value.comentario,
        calificacion: formulario.value.calificacion
      })
    } else {
      await axios.post('/api/resenas', formulario.value)
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
    await axios.delete(`/api/resenas/${id}`)
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
  <div class="crud">
    <h1>Reseñas del restaurante</h1>

    <p v-if="error" class="error">{{ error }}</p>

    <form @submit.prevent="guardar" class="formulario">
      <h2>{{ editandoId ? 'Editar reseña' : 'Nueva reseña' }}</h2>

      <label>
        Id del plato
        <input v-model="formulario.platoId" type="number" required :disabled="!!editandoId" />
      </label>

      <label>
        Nombre
        <input v-model="formulario.nombre" type="text" required />
      </label>

      <label>
        Comentario
        <textarea v-model="formulario.comentario" required></textarea>
      </label>

      <label>
        Calificación
        <select v-model="formulario.calificacion">
          <option :value="5">5 - Excelente</option>
          <option :value="4">4 - Muy bueno</option>
          <option :value="3">3 - Bueno</option>
          <option :value="2">2 - Regular</option>
          <option :value="1">1 - Malo</option>
        </select>
      </label>

      <div class="acciones-form">
        <button type="submit">{{ editandoId ? 'Guardar cambios' : 'Crear reseña' }}</button>
        <button type="button" v-if="editandoId" @click="limpiarFormulario">Cancelar</button>
      </div>
    </form>

    <h2>Listado</h2>
    <p v-if="cargando">Cargando...</p>
    <p v-if="!cargando && resenas.length === 0">No hay reseñas registradas.</p>

    <div class="lista">
      <div v-for="resena in resenas" :key="resena._id" class="tarjeta">
        <p><strong>{{ resena.nombre }}</strong> — plato #{{ resena.platoId }} — {{ resena.calificacion }}/5</p>
        <p>{{ resena.comentario }}</p>
        <div class="acciones-tarjeta">
          <button @click="editar(resena)">Editar</button>
          <button @click="eliminar(resena._id)">Eliminar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.crud {
  max-width: 700px;
  margin: 0 auto;
  padding: 20px;
  text-align: left;
}
.formulario {
  display: flex;
  flex-direction: column;
  gap: 10px;
  border: 1px solid #444;
  padding: 16px;
  border-radius: 8px;
  margin-bottom: 24px;
}
.formulario label {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.acciones-form,
.acciones-tarjeta {
  display: flex;
  gap: 8px;
}
.tarjeta {
  border: 1px solid #444;
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 12px;
}
.error {
  color: #ff6b6b;
}
</style>