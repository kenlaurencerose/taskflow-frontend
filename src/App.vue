<template>
  <main>
    <TaskList />
  </main>
</template>

<script setup lang="ts">
import TaskList from './components/TaskList.vue' // Pfad zu deiner TaskList
</script>

<style>
/* Globales Zurücksetzen, damit das Design sauber auf der ganzen Seite liegt */
body {
  margin: 0;
  padding: 0;
  background-color: #0f172a; /* Dunkler Hintergrund */
}
</style>
