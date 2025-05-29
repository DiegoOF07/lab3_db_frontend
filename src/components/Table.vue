<template>
  <div class="table-container">
    <table class="product-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombre</th>
          <th>Precio</th>
          <th>Descripción</th>
          <th>Estado</th>
          <th>Categorías</th>
          <th>Creado</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="producto in productos" :key="producto.id">
          <td>{{ producto.id }}</td>
          <td>{{ producto.nombre }}</td>
          <td>${{ producto.precio.toFixed(2) }}</td>
          <td>{{ producto.descripcion }}</td>
          <td>
            <span :class="['estado-badge', producto.estado]">{{ producto.estado }}</span>
          </td>
          <td>
            <ul>
              <li v-for="categoria in producto.categorias" :key="categoria.id">
                {{ categoria.nombre }}
              </li>
            </ul>
          </td>
          <td>{{ formatDate(producto.created_at) }}</td>
          <td>
            <div class="action-buttons">
                <button @click="$emit('edit', producto)">✏️</button>
                <button @click="$emit('delete', producto.id)">🗑️</button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script setup>
const props = defineProps({
  productos: Array
})
const emit = defineEmits(['edit', 'delete'])

function formatDate(dateStr) {
  return new Date(dateStr).toLocaleDateString('es-ES')
}
</script>

<style scoped>
.table-container {
  overflow-x: auto;
  margin: 1rem;
  border-radius: 8px;
  background-color: #ffffff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  min-width: 900px;
}

.product-table thead {
  background-color: #f9fafb;
  position: sticky;
  top: 0;
  z-index: 1;
}

.product-table th,
.product-table td {
  padding: 12px 16px;
  text-align: left;
  font-size: 14px;
  border-bottom: 1px solid #e5e7eb;
  color: #1f2937;
}

.product-table tbody tr:hover {
  background-color: #f1f5f9;
}

.estado-badge {
  display: inline-block;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
  text-transform: capitalize;
}

.estado-badge.disponible {
  background-color: #d1fae5;
  color: #065f46;
}

.estado-badge.pendiente {
  background-color: #fef3c7;
  color: #92400e;
}

.estado-badge.agotado,
.estado-badge.inactivo {
  background-color: #fee2e2;
  color: #991b1b;
}

ul {
  padding-left: 16px;
  margin: 0;
}

ul li {
  margin-bottom: 4px;
}

.action-buttons{
    display: flex;
    gap: 5px;
    flex-wrap: nowrap;
    align-items: center;
    justify-content: center;
    height: 67px;
}

button {
  margin-right: 8px;
  padding: 4px 8px;
  border: none;
  cursor: pointer;
  font-size: 14px;
  border-radius: 4px;
  background-color: #3b82f6;
  color: white;
}
button:last-child {
  background-color: #ef4444;
}

/* 🌙 Dark Mode */
@media (prefers-color-scheme: dark) {
  .table-container {
    background-color: #1f2937;
    box-shadow: 0 2px 8px rgba(255, 255, 255, 0.05);
  }

  .product-table thead {
    background-color: #374151;
  }

  .product-table th,
  .product-table td {
    color: #e5e7eb;
    border-bottom: 1px solid #4b5563;
  }

  .product-table tbody tr:hover {
    background-color: #2d3748;
  }

  .estado-badge.disponible {
    background-color: #0f773e;
    color: #d1fae5;
  }

  .estado-badge.pendiente {
    background-color: #92400e;
    color: #fef3c7;
  }

  .estado-badge.agotado,
  .estado-badge.inactivo {
    background-color: #991b1b;
    color: #fee2e2;
  }
  button {
    background-color: #2563eb;
  }
  button:last-child {
    background-color: #b91c1c;
  }
}
</style>
