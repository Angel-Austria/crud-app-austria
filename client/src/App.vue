<script setup>
import { ref, onMounted } from 'vue'

const API = 'http://localhost:4000/api/items'
const items = ref([])
const form = ref({ name: '', description: '', price: '' })
const editId = ref(null)

async function load() {
  items.value = await fetch(API).then(r => r.json())
}

async function save() {
  if (editId.value) {
    await fetch(`${API}/${editId.value}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
    editId.value = null
  } else {
    await fetch(API, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(form.value)
    })
  }
  form.value = { name: '', description: '', price: '' }
  load()
}

function startEdit(item) {
  editId.value = item.id
  form.value = { name: item.name, description: item.description, price: item.price }
}

async function remove(id) {
  await fetch(`${API}/${id}`, { method: 'DELETE' })
  load()
}

onMounted(load)
</script>

<template>
  <main>
    <h1>Items</h1>

    <form @submit.prevent="save" class="item-form">
      <input v-model="form.name" placeholder="Name" required />
      <input v-model="form.description" placeholder="Description" />
      <input v-model="form.price" placeholder="Price" type="number" step="0.01" min="0" />
      <button type="submit">{{ editId ? 'Update' : 'Add' }}</button>
    </form>

    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Description</th>
          <th>Price</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in items" :key="item.id">
          <td>{{ item.name }}</td>
          <td>{{ item.description }}</td>
          <td>₱{{ item.price != null ? Number(item.price).toFixed(2) : '0.00' }}</td>
          <td>
            <button class="btn-edit" @click="startEdit(item)">Edit</button>
            <button class="btn-delete" @click="remove(item.id)">Delete</button>
          </td>
        </tr>
        <tr v-if="items.length === 0">
          <td colspan="4" class="empty">No items found.</td>
        </tr>
      </tbody>
    </table>
  </main>
</template>

<style>
* { box-sizing: border-box; margin: 0; padding: 0; }

body {
  font-family: Arial, sans-serif;
  background: #f5f5f5;
  padding: 2rem;
}

main {
  max-width: 800px;
  margin: 0 auto;
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

h1 {
  font-size: 1.8rem;
  margin-bottom: 1.5rem;
  color: #333;
}

.item-form {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1.5rem;
  flex-wrap: wrap;
}

.item-form input {
  flex: 1;
  min-width: 120px;
  padding: 0.5rem 0.75rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 0.95rem;
}

.item-form input:focus {
  outline: none;
  border-color: #4a90e2;
}

.item-form button {
  padding: 0.5rem 1.2rem;
  background: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 0.95rem;
  cursor: pointer;
}

.item-form button:hover { background: #357abd; }

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 0.75rem 1rem;
  text-align: left;
  border-bottom: 1px solid #eee;
}

th {
  background: #f0f0f0;
  font-weight: bold;
  color: #555;
}

tbody tr:hover { background: #fafafa; }

.btn-edit, .btn-delete {
  padding: 0.3rem 0.75rem;
  border: none;
  border-radius: 4px;
  font-size: 0.85rem;
  cursor: pointer;
  margin-right: 0.4rem;
}

.btn-edit { background: #e8f0fe; color: #1a73e8; }
.btn-delete { background: #fce8e8; color: #d93025; }
.btn-edit:hover { background: #d2e3fc; }
.btn-delete:hover { background: #f5c6c6; }

.empty {
  text-align: center;
  color: #999;
  padding: 2rem;
}
</style>