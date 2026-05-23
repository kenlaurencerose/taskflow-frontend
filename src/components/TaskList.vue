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
import { ref, onMounted } from 'vue'

const tasks = ref<any[]>([])

function loadThings() {
  const endpoint = import.meta.env.VITE_BACKEND_BASE_URL
  const requestOptions: RequestInit = {
    method: 'GET',
    redirect: 'follow'
  }

  fetch(endpoint, requestOptions)
    .then(response => response.json())
    .then((result: any[]) => result.forEach((task: any) => {
      tasks.value.push(task)
    }))
    .catch(error => console.log('error', error))
}

onMounted(() => {
  loadThings()

})
</script>
