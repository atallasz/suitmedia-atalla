<script setup lang="ts">
import { RouterView } from 'vue-router'
</script>

<template>
  <RouterView name="header" />
  <RouterView name="banner" />
  <RouterView />
</template>

<style>
.main-content {
  min-height: 100vh;
}
</style>

