<script setup>
import CurrencyConverter from './components/CurrencyConverter.vue'
</script>

<template>
  <CurrencyConverter />
</template>

<style>
/* Style global : centrage de la carte sur la page */
body {
  margin: 0;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #eef2f5;
  font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
}
</style>
