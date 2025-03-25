<script setup>
import star from './components/star UI.vue'
import DownloadExcel from './components/download excel.vue'
</script>

<template>
  <star msg="小星星小工具" />
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
