<script setup>
  import { ref } from 'vue'
  import MapView from './views/MapView.vue'
  import Navbar from './components/Navbar.vue'
  import Card from './components/Card.vue'
</script>

<template>
  <div id="app">
    <Navbar />
    <div class="Map-contanier">
      <Card />
      <MapView />

    </div>

  </div>

</template>

<style>
  #app {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }

  .Map-contanier {
    width: 100%;
    height: calc(100% - 60px);

  }
</style>