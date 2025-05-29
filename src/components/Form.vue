<template>
  <form @submit.prevent="handleSubmit" class="form">
    <div>
      <label>Nombre:</label>
      <input v-model="localProducto.nombre" required />
    </div>

    <div>
      <label>Precio:</label>
      <input v-model.number="localProducto.precio" type="number" step="0.01" required />
    </div>

    <div>
      <label>Descripción:</label>
      <textarea v-model="localProducto.descripcion" required></textarea>
    </div>

    <div>
      <label>Estado:</label>
      <select v-model="localProducto.estado" required>
        <option value="disponible">Disponible</option>
        <option value="agotado">Agotado</option>
        <option value="inactivo">Inactivo</option>
      </select>
    </div>

    <div>
      <label>Categorías:</label>
      <Multiselect
        v-model="selectedCategorias"
        :options="categorias"
        :multiple="true"
        :close-on-select="false"
        placeholder="Selecciona categorías"
        label="nombre"
        track-by="id"
        class="custom-multiselect"
        :searchable="true"
        :show-labels="false"
        select-label=""
        selected-label=""
        deselect-label=""
      />
    </div>

    <div class="actions">
      <button type="submit">{{ isEditMode ? 'Actualizar' : 'Agregar' }}</button>
      <button type="button" @click="$emit('cancel')">Cancelar</button>
    </div>
  </form>
</template>

<script setup lang="ts">
import { reactive, watch, computed, onMounted, ref } from 'vue'
import Multiselect from 'vue-multiselect'

const categorias = ref<{ id: number; nombre: string }[]>([])

const props = defineProps({
  producto: {
    type: Object,
    default: () => ({
      nombre: '',
      precio: 0,
      descripcion: '',
      estado: 'disponible',
      categoria_ids: []
    })
  }
})

const emit = defineEmits(['submit', 'cancel'])

const localProducto = reactive({
  nombre: '',
  precio: 0,
  descripcion: '',
  estado: 'disponible',
  categoria_ids: [] as number[]
})

const selectedCategorias = ref<{ id: number; nombre: string }[]>([])

const isEditMode = computed(() => !!props.producto?.id)

const updateSelectedCategorias = (producto: any) => {
  if (categorias.value.length && producto?.categoria_ids?.length) {
    selectedCategorias.value = categorias.value.filter(cat =>
      producto.categoria_ids.includes(cat.id)
    )
  } else {
    selectedCategorias.value = []
  }
}

watch(
  () => props.producto,
  (producto) => {
    Object.assign(localProducto, {
      nombre: producto.nombre || '',
      precio: producto.precio || 0,
      descripcion: producto.descripcion || '',
      estado: producto.estado || 'disponible',
      categoria_ids: producto.categoria_ids || []
    })

    updateSelectedCategorias(producto)
  },
  { immediate: true }
)

watch(
  () => categorias.value,
  () => {
    updateSelectedCategorias(props.producto)
  }
)

onMounted(async () => {
  try {
    const res = await fetch('http://localhost:8080/categorias')
    categorias.value = await res.json()    
    updateSelectedCategorias(props.producto)
  } catch (error) {
    console.error('Error cargando categorías:', error)
  }
})

function handleSubmit() {
  emit('submit', {
    nombre: localProducto.nombre,
    precio: localProducto.precio,
    descripcion: localProducto.descripcion,
    estado: localProducto.estado,
    categoria_ids: selectedCategorias.value.map(c => c.id)
  })
}
</script>

<style scoped>
@import 'vue-multiselect/dist/vue-multiselect.min.css';

.form {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1rem;
  max-width: 450px;
  background: var(--form-bg, #fff);
}

input,
textarea,
select {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.actions {
  display: flex;
  justify-content: space-between;
}

button {
  padding: 0.5rem 1rem;
  border: none;
  background-color: #10b981;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

button[type='button'] {
  background-color: #6b7280;
}

.custom-multiselect {
  font-size: 14px;
}

:deep(.multiselect) {
  border: 1px solid #ccc;
  border-radius: 4px;
  background: white;
  min-height: 38px;
}

:deep(.multiselect__input) {
  background: transparent;
  border: none;
  font-size: 14px;
  padding: 4px 8px;
}

:deep(.multiselect__placeholder) {
  color: #9ca3af;
  font-size: 14px;
  padding-left: 8px;
  margin-bottom: 0;
}

:deep(.multiselect__tags) {
  min-height: 38px;
  padding: 4px 40px 0 8px;
  border: none;
  background: transparent;
}

:deep(.multiselect__tag) {
  background: #10b981;
  color: white;
  border-radius: 4px;
  padding: 4px 8px;
  margin: 2px 4px 2px 0;
  font-size: 12px;
  font-weight: 500;
}

:deep(.multiselect__tag-icon) {
  border-radius: 0 4px 4px 0;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  font-weight: bold;
}

:deep(.multiselect__tag-icon:hover) {
  background: rgba(255, 255, 255, 0.3);
}

:deep(.multiselect__select) {
  height: 38px;
  width: 40px;
  background: transparent;
  border-left: 1px solid #e5e7eb;
}

:deep(.multiselect__select:before) {
  border-color: #6b7280 transparent transparent;
  border-width: 5px 4px 0;
  top: 50%;
  transform: translateY(-50%);
}

:deep(.multiselect__content-wrapper) {
  border: 1px solid #d1d5db;
  border-top: none;
  border-radius: 0 0 4px 4px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  background: white;
  max-height: 200px;
}

:deep(.multiselect__content) {
  background: white;
}

:deep(.multiselect__option) {
  padding: 8px 12px;
  font-size: 14px;
  color: #374151;
  transition: background-color 0.2s;
}

/* Opción al hacer hover */
:deep(.multiselect__option--highlight) {
  background: #f3f4f6;
  color: #1f2937;
}

/* Opción seleccionada */
:deep(.multiselect__option--selected) {
  background: #10b981;
  color: white;
  font-weight: 500;
}

:deep(.multiselect__option--selected.multiselect__option--highlight) {
  background: #059669;
  color: white;
}

/* Estado focus */
:deep(.multiselect--active) {
  border-color: #10b981;
  box-shadow: 0 0 0 2px rgba(16, 185, 129, 0.2);
}

/* Tema oscuro */
@media (prefers-color-scheme: dark) {
  .form {
    background: #1f2937;
    color: #f9fafb;
  }

  input,
  textarea,
  select {
    background: #374151;
    color: #f9fafb;
    border-color: #4b5563;
    padding: 0.6rem 1.2rem;
  }

  /* Multiselect tema oscuro */
  :deep(.multiselect) {
    background: #374151;
    border-color: #4b5563;
  }

  :deep(.multiselect__input) {
    background: transparent;
    color: #f9fafb;
  }

  :deep(.multiselect__placeholder) {
    color: #6b7280;
  }

  :deep(.multiselect__tags) {
    background: transparent;
  }

  :deep(.multiselect__select) {
    border-left-color: #4b5563;
  }

  :deep(.multiselect__select:before) {
    border-color: #9ca3af transparent transparent;
  }

  :deep(.multiselect__content-wrapper) {
    background: #374151;
    border-color: #4b5563;
  }

  :deep(.multiselect__content) {
    background: #374151;
  }

  :deep(.multiselect__option) {
    color: #f9fafb;
  }

  :deep(.multiselect__option--highlight) {
    background: #4b5563;
    color: #f9fafb;
  }

  :deep(.multiselect__option--selected) {
    background: #10b981;
    color: white;
  }

  :deep(.multiselect__option--selected.multiselect__option--highlight) {
    background: #059669;
  }

  :deep(.multiselect--active) {
    border-color: #10b981;
  }
}
</style>