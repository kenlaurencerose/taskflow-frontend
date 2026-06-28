<template>
  <div class="app-background">
    <div class="task-card">
      <header class="app-header">
        <h1>TaskFlow</h1>
        <p class="subtitle">Organisiere deine Aufgaben effizient</p>
      </header>

      <div class="stats-section">
        <div class="stat-box">Gesamt: <span>{{ totalTasks }}</span></div>
        <div class="stat-box">Offen: <span>{{ openTasks }}</span></div>
        <div class="stat-box">Erledigt: <span>{{ completedTasks }}</span></div>
      </div>

      <div class="search-bar">
        <span class="search-icon">🔍</span>
        <input
          v-model="searchQuery"
          placeholder="Tasks nach Titel durchsuchen..."
        />
      </div>

      <form @submit.prevent="createTask" class="task-form">
        <h3>✨ Neuen Task hinzufügen</h3>
        <div class="input-group">
          <input v-model="newTask.title" placeholder="Was steht an?" required />
        </div>
        <div class="input-row">
          <input v-model="newTask.status" placeholder="Status (z.B. Offen)" />
          <input v-model="newTask.priority" placeholder="Priorität (z.B. Hoch)" />
        </div>
        <button type="submit" class="submit-btn">Hinzufügen</button>
      </form>

      <div class="list-section">
        <div class="list-header">
          <h3>📋 Deine Aufgaben</h3>
          <button
            v-if="tasks.length > 0"
            @click="clearAllTasks"
            class="clear-all-btn"
          >
            Alle löschen
          </button>
        </div>

        <ul class="task-list" v-if="filteredTasks.length > 0">
          <li v-for="task in filteredTasks" :key="task.id" class="task-item" :class="{ 'is-completed': task.status === 'Erledigt' }">
            <div class="task-left">
              <input
                type="checkbox"
                :checked="task.status === 'Erledigt'"
                @change="toggleTaskStatus(task)"
                class="status-checkbox"
              />
              <div class="task-details">
                <span class="task-title">{{ task.title }}</span>
                <div class="task-badges">
                  <span class="badge status">{{ task.status }}</span>
                  <span class="badge priority" :class="task.priority.toLowerCase()">{{ task.priority }}</span>
                </div>
              </div>
            </div>
            <button @click="deleteTask(task.id)" class="delete-btn" title="Task löschen">🗑️</button>
          </li>
        </ul>

        <div v-else class="empty-state">
          <p>Keine passenden Tasks gefunden.</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const tasks = ref<any[]>([])
const searchQuery = ref('')
const newTask = ref({ title: '', status: 'Offen', priority: 'Normal' })
const baseUrl = import.meta.env.VITE_BACKEND_BASE_URL

function loadTasks() {
  fetch(baseUrl)
    .then(response => response.json())
    .then((result: any[]) => { tasks.value = result })
    .catch(error => console.log('error', error))
}

function createTask() {
  const requestOptions: RequestInit = {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newTask.value)
  }
  fetch(`${baseUrl}tasks`, requestOptions)
    .then(response => response.json())
    .then(() => {
      loadTasks()
      newTask.value = { title: '', status: 'Offen', priority: 'Normal' }
    })
    .catch(error => console.log('error', error))
}

function deleteTask(id: number) {
  const requestOptions: RequestInit = {
    method: 'DELETE',
    redirect: 'follow'
  }
  fetch(`${baseUrl}tasks/${id}`, requestOptions)
    .then(response => {
      if (response.ok) {
        loadTasks()
      }
    })
    .catch(error => console.log('error', error))
}

function toggleTaskStatus(task: any) {
  const nextStatus = task.status === 'Erledigt' ? 'Offen' : 'Erledigt'
  const requestOptions: RequestInit = {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: nextStatus
  }
  fetch(`${baseUrl}tasks/${task.id}/status`, requestOptions)
    .then(response => {
      if (response.ok) {
        loadTasks()
      }
    })
    .catch(error => console.log('error', error))
}

function clearAllTasks() {
  if (confirm('Möchtest du wirklich alle Aufgaben unwiderruflich löschen?')) {
    const requestOptions: RequestInit = { method: 'DELETE' }
    fetch(`${baseUrl}tasks`, requestOptions)
      .then(response => {
        if (response.ok) {
          loadTasks()
        }
      })
      .catch(error => console.log('error', error))
  }
}

const totalTasks = computed(() => tasks.value.length)
const completedTasks = computed(() => tasks.value.filter(t => t.status === 'Erledigt').length)
const openTasks = computed(() => totalTasks.value - completedTasks.value)

const filteredTasks = computed(() => {
  return tasks.value.filter(task => task.title.toLowerCase().includes(searchQuery.value.toLowerCase()))
})

onMounted(() => { loadTasks() })
</script>

<style scoped>
.app-background {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 40px 20px;
  box-sizing: border-box;
  color: #f8fafc;
  font-family: 'Segoe UI', Roboto, sans-serif;
}

.task-card {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 16px;
  width: 100%;
  max-width: 550px;
  padding: 30px;
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.3), 0 8px 10px -6px rgba(0, 0, 0, 0.3);
}

.app-header { text-align: center; margin-bottom: 20px; }
.app-header h1 { margin: 0; font-size: 2rem; color: #38bdf8; font-weight: 700; }
.subtitle { margin: 5px 0 0; color: #94a3b8; font-size: 0.9rem; }

/* Statistik Leiste */
.stats-section {
  display: flex; justify-content: space-around; background: #0f172a;
  padding: 12px; border-radius: 10px; margin-bottom: 25px; border: 1px solid #334155;
}
.stat-box { font-size: 0.85rem; color: #94a3b8; }
.stat-box span { display: block; font-size: 1.2rem; font-weight: bold; color: #38bdf8; text-align: center; margin-top: 2px; }

/* Suchleiste */
.search-bar { position: relative; margin-bottom: 25px; }
.search-icon { position: absolute; left: 12px; top: 50%; transform: translateY(-50%); color: #64748b; }
.search-bar input {
  width: 100%; padding: 12px 12px 12px 40px; border-radius: 8px;
  border: 1px solid #334155; background: #0f172a; color: white;
  box-sizing: border-box; font-size: 0.95rem;
}
.search-bar input:focus { border-color: #38bdf8; outline: none; }

/* Formular */
.task-form { background: #0f172a; border: 1px solid #334155; padding: 20px; border-radius: 12px; margin-bottom: 30px; }
.task-form h3 { margin-top: 0; margin-bottom: 15px; font-size: 1.1rem; color: #e2e8f0; }
.input-group { margin-bottom: 12px; }
.input-row { display: flex; gap: 12px; margin-bottom: 15px; }
.task-form input {
  width: 100%; padding: 10px; border-radius: 6px; border: 1px solid #334155;
  background: #1e293b; color: white; box-sizing: border-box; font-size: 0.9rem;
}
.task-form input:focus { border-color: #38bdf8; outline: none; }
.submit-btn {
  width: 100%; padding: 12px; border: none; border-radius: 6px;
  background: #0284c7; color: white; font-weight: 600; cursor: pointer;
  font-size: 0.95rem;
}
.submit-btn:hover { background: #0369a1; }

/* Task-Liste Header & Button */
.list-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px; border-bottom: 1px solid #334155; padding-bottom: 8px; }
.list-header h3 { font-size: 1.1rem; margin: 0; color: #94a3b8; }
.clear-all-btn {
  background: transparent; border: 1px solid #ef4444; color: #ef4444;
  padding: 4px 10px; border-radius: 6px; cursor: pointer; font-size: 0.8rem;
  transition: all 0.2s;
}
.clear-all-btn:hover { background: #ef4444; color: white; }

.task-list { list-style: none; padding: 0; margin: 0; display: flex; flex-direction: column; gap: 10px; }
.task-item { display: flex; justify-content: space-between; align-items: center; background: #0f172a; border: 1px solid #334155; padding: 14px; border-radius: 8px; }
.task-left { display: flex; align-items: center; gap: 14px; }
.status-checkbox { width: 18px; height: 18px; cursor: pointer; accent-color: #38bdf8; }
.task-title { font-weight: 600; color: #f1f5f9; display: block; margin-bottom: 4px; }
.task-badges { display: flex; gap: 8px; }
.badge { font-size: 0.75rem; padding: 2px 8px; border-radius: 4px; font-weight: 500; }
.badge.status { background: #1e293b; color: #94a3b8; border: 1px solid #334155; }
.badge.priority { background: #16a34a; color: white; }
.badge.priority.hoch { background: #dc2626; }

.task-item.is-completed { opacity: 0.5; }
.task-item.is-completed .task-title { text-decoration: line-through; color: #64748b; }

.delete-btn { background: transparent; border: none; cursor: pointer; font-size: 1.2rem; padding: 5px; border-radius: 6px; }
.delete-btn:hover { background: #27272a; }
.empty-state { text-align: center; color: #64748b; padding: 20px 0; }
</style>
