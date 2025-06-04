<template>
  <div class="theme-container">
    <NuxtRouteAnnouncer />
    <NuxtPage />
  </div>
</template>

<script setup>
// No additional script logic needed here
</script>

<style>
:root {
  --color-primary: #4A90E2;
  --color-secondary: #FFFFFF;
  --color-accent: #F5A623;
}
.theme-container {
  background-color: var(--color-secondary);
  min-height: 100vh;
  color: #333;
}
</style>
