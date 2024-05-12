<script setup>
import { ref, computed } from "vue"
import { bakedGoods } from "./data/bakedGoodsData"

const tableData = ref(bakedGoods)
const filterText = ref('')
const newItem = ref({
  type: '',
  name: '',
  topping: ''
})
const sortColumn = ref(null)
const sortOrder = ref(1) // 1 for ascending, -1 for descending
const dialog = ref(false)

const filteredData = computed(() => {
  const searchText = filterText.value.toLowerCase()
  return tableData.value.filter(row => {
    const keys = Object.keys(row)
    return keys.some(key =>
      row[key].toString().toLowerCase().includes(searchText)
    )
  })
})

const sortedData = computed(() => {
  if (!sortColumn.value) {
    return filteredData.value
  }
  const data = [...filteredData.value]
  return data.sort((a, b) => {
    const sortValueA = a[sortColumn.value]
    const sortValueB = b[sortColumn.value]
    if (typeof sortValueA === 'string') {
      return sortValueA.localeCompare(sortValueB) * sortOrder.value
    } else {
      return (sortValueA - sortValueB) * sortOrder.value
    }
  })
})

const sortBy = (column) => {
  if (sortColumn.value === column) {
    sortOrder.value = sortOrder.value * -1
  } else {
    sortColumn.value = column
    sortOrder.value = 1
  }
}

const addNewItem = () => {
  newItem.value.id = tableData.value.reduce(function(max, obj) {
    return obj.id > max.id ? obj : max
  }).id + 1
  tableData.value.push(newItem.value)
  newItem.value = {
    type: '',
    name: '',
    topping: ''
  }
}
</script>

<template>
  <main>
    <section class="data-table">
      <h1>Baked Goods</h1>
      <div class="table-controls">
        <button @click="dialog.showModal()" class="primary">
          Add New Item
        </button>
        <form @submit.prevent>
          <div class="form-control form-inline">
            <label for="filterText">Filter</label>
            <input v-model="filterText" id="filterText" class="search-field" type="text">
          </div>
        </form>
      </div>
      <table>
        <thead>
          <tr>
            <th :class="[{active: sortColumn === 'id'}, sortOrder === 1 ? 'asc' : 'desc']">
              <button @click="sortBy('id')">ID</button>
            </th>
            <th :class="[{active: sortColumn === 'type'}, sortOrder === 1 ? 'asc' : 'desc']">
              <button @click="sortBy('type')">Type</button>
            </th>
            <th :class="[{active: sortColumn === 'topping'}, sortOrder === 1 ? 'asc' : 'desc']">
              <button @click="sortBy('topping')">Topping</button>
            </th>
            <th :class="[{active: sortColumn === 'name'}, sortOrder === 1 ? 'asc' : 'desc']">
              <button @click="sortBy('name')">Name</button>
            </th>
          </tr>
        </thead>
        <tbody v-if="sortedData.length">
          <tr v-for="item in sortedData" :key="item.id">
            <td>{{ item.id }}</td>
            <td>{{ item.type }}</td>
            <td>{{ item.topping }}</td>
            <td>{{ item.name }}</td>
          </tr>
        </tbody>
        <tbody v-else>
          <tr>
            <td colspan="4" role="alert" class="alert">No results. Please try another filter option.</td>
          </tr>
        </tbody>
      </table>
      <dialog ref="dialog" @click.self="dialog.close()">
        <div class="dialog-content">
          <h2>Add New Baked Good</h2>
          <form @submit.prevent="addNewItem">
            <div class="form-control">
              <label for="type">Type</label>
              <input v-model="newItem.type" id="type" required>
            </div>
            <div class="form-control">
              <label for="name">Name</label>
              <input v-model="newItem.name" id="name" required>
            </div>
            <div class="form-control">
              <label for="topping">Topping</label>
              <input v-model="newItem.topping" id="topping" required>
            </div>
            <div class="dialog-footer">
              <button type="submit" class="primary">Add</button>
              <button @click="dialog.close()" class="secondary" type="button">Close</button>
            </div>
          </form>
        </div>
      </dialog>
    </section>
  </main>
</template>
