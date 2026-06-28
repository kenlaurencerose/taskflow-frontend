<template>
  <div class="task-container">
    <h1>Task Liste</h1>

    <div class="filter-section">
      <input
        v-model="searchQuery"
        placeholder="Tasks nach Titel filtern..."
        class="filter-input"
      />
    </div>

    <form @submit.prevent="createTask" class="task-form">
      <h3>Neuen Task hinzufügen</h3>
      <input v-model="newTask.title" placeholder="Titel" required />
      <input v-model="newTask.status" placeholder="Status (z.B. Offen)" />
      <input v-model="newTask.priority" placeholder="Priorität (z.B. Hoch)" />
      <button type="submit">Hinzufügen</button>
    </form>

    <ul class="task-list">
      <li v-for="task in filteredTasks" :key="task.id" class="task-item">
        <span class="task-info">
          <strong>{{ task.title }}</strong> - {{ task.status }} - [{{ task.priority }}]
        </span>

        <button @click="deleteTask(task.id)" class="delete-btn">Löschen</button>
      </li>
    </ul>

    <p v-if="filteredTasks.length === 0">Keine Tasks gefunden.</p>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

// Reaktive Variablen
const tasks = ref<any[]>([])
const searchQuery = ref('') // Für den Filter

// Die Maske/Formular-Daten
const newTask = ref({
  title: '',
  status: 'Offen',
  priority: 'Normal'
})

const baseUrl = import.meta.env.VITE_BACKEND_BASE_URL

// ================= USE CASE: ALLE TASKS LADEN (GET /) =================
function loadTasks() {
  fetch(baseUrl)
    .then(response => response.json())
    .then((result: any[]) => {
      tasks.value = result // Überschreibt die Liste sauber ohne Duplikate
    })
    .catch(error => console.log('error', error))
}

// ================= USE CASE: TASK ERSTELLEN (POST /tasks) =================
function createTask() {
  fetch(`${baseUrl}tasks`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newTask.value)
  })
    .then(response => response.json())
    .then(() => {
      loadTasks() // Liste neu laden, damit der neue Task erscheint
      // Maske zurücksetzen
      newTask.value = { title: '', status: 'Offen', priority: 'Normal' }
    })
    .catch(error => console.log('error', error))
}

// ================= USE CASE: TASK LÖSCHEN (Vorbereitung für Backend) =================
function deleteTask(id: number) {
  console.log('Lösche Task mit ID:', id)
  // Hier kommt gleich das FETCH für DELETE hin, sobald das Backend bereit ist!
}

// ================= ZUSATZPUNKTE: FILTER-LOGIK =================
// "computed" überwacht die searchQuery und filtert die Liste live im Browser
const filteredTasks = computed(() => {
  return tasks.value.filter(task => {
    return task.title.toLowerCase().includes(searchQuery.value.toLowerCase())
  })
})

onMounted(() => {
  loadTasks()
})
</script>

<style scoped>
/* Ein bisschen Styling, damit es ordentlich aussieht */
.task-container { max-width: 500px; margin: 0 auto; font-family: sans-serif; }
.filter-input, .task-form input { display: block; width: 100%; margin-bottom: 10px; padding: 8px; box-sizing: border-box; }
.task-form { background: #f4f4f4; padding: 15px; border-radius: 5px; margin-bottom: 20px; color: #333; }
.task-item { display: flex; justify-content: space-between; padding: 10px; border-bottom: 1px solid #ccc; align-items: center; }
.delete-btn { background: #ff4d4d; color: white; border: none; padding: 5px 10px; cursor: pointer; border-radius: 3px; }
</style>
