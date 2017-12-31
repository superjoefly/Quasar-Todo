<template>
  <!-- Don't drop "q-app" class -->
  <div id="q-app">
    <!-- <router-view /> -->
    <app-layout></app-layout>
  </div>
</template>

<script>
import Layout from './components/Layout'

export default {
  components: {
    'app-layout': Layout
  }
}
</script>

<style></style>
