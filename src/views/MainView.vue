<template>
  <div>
    <h1>Productos</h1>
    <button class="btn-add" @click="abrirFormulario">Agregar Producto</button>

      <Table
      :productos="productos"
      @edit="abrirFormulario"
      @delete="eliminarProducto" />

    <div v-if="mostrarFormulario" class="modal-overlay" @click.self="cerrarFormulario">
      <div class="modal">
        <Form
          :producto="productoSeleccionado"
          @submit="guardarProducto"
          @cancel="cerrarFormulario"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Table from '../components/Table.vue'
import Form from '../components/Form.vue'

const productos = ref([])
const mostrarFormulario = ref(false)
const productoSeleccionado = ref({
  nombre: '',
  precio: 0,
  descripcion: '',
  estado: 'disponible',
  categoria_ids: []
})

const fetchProductos = async () => {
  try {
    const res = await fetch('http://localhost:8080/productos')
    productos.value = await res.json()
  } catch (error) {
    console.error('Error cargando productos:', error)
  }
}

const guardarProducto = async (producto) => {
  try {
    let response;
    
    if (productoSeleccionado.value.id) {
      response = await fetch(`http://localhost:8080/productos/${productoSeleccionado.value.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(producto)
      })
    } else {
      response = await fetch('http://localhost:8080/productos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(producto)
      })
    }

    if (response.ok) {
      console.log('Producto guardado exitosamente')
      cerrarFormulario()
      await fetchProductos()
    } else {
      console.error('Error en la respuesta del servidor:', response.status)
    }
  } catch (error) {
    console.error('Error al guardar producto:', error)
  }
}

const eliminarProducto = async (id: number) => {
  if (!confirm('¿Estás seguro de eliminar este producto?')) return

  try {
    const response = await fetch(`http://localhost:8080/productos/${id}`, {
      method: 'DELETE'
    })
    
    if (response.ok) {
      await fetchProductos()
    } else {
      console.error('Error al eliminar producto:', response.status)
    }
  } catch (error) {
    console.error('Error al eliminar producto:', error)
  }
}

const abrirFormulario = (producto = null) => {  
  productoSeleccionado.value = producto
    ? {
        ...JSON.parse(JSON.stringify(producto)),
        categoria_ids: producto.categorias?.map(cat => cat.id) || []
      }
    : {
        nombre: '',
        precio: 0,
        descripcion: '',
        estado: 'disponible',
        categoria_ids: []
      }

  mostrarFormulario.value = true
}

const cerrarFormulario = () => {
  mostrarFormulario.value = false
  productoSeleccionado.value = {
    nombre: '',
    precio: 0,
    descripcion: '',
    estado: 'disponible',
    categoria_ids: []
  }
}

onMounted(fetchProductos)
</script>

<style scoped>
.btn-add {
  background-color: #0f773e;
  border: none;
  padding: 0.6rem 1.2rem;
  margin: 1rem 0;
  border-radius: 5px;
  color: white;
  cursor: pointer;
  font-size: 16px;
}

.modal-overlay {
  width: 100%;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.65);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
}

.modal {
  display: flex;
  justify-content: center;
  background: #1f2937;
  padding: 1.5rem;
  border-radius: 10px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 0 15px rgba(255, 255, 255, 0.1);
}

@media (prefers-color-scheme: light) {
  .container {
    color: #111;
  }
  .modal {
    background: #fff;
    color: #111;
  }
}
</style>