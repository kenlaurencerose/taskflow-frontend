<template>
  <div>
    <h1>Task Liste</h1>

    <ul>
      <li v-for="task in tasks" :key="task.id">
        {{ task.title }} - {{ task.status }} - {{ task.priority }}
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const tasks = ref([
  {
    id: 1, title: "Projekt anlegen", status: "Open", priority: "High"
  },
  {
    id: 2, title: "Vue bauen", status: "In Progress", priority: "Medium"
  },
  {
    id: 3, title: "Repo pushen", status: "Done", priority: "High"
  }
])
</script>
