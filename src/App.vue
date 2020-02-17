<template>
  <div id="app">
    <ColorChord />
  </div>
</template>

<script>
import ColorChord from './components/ColorChord.vue'

export default {
  name: 'App',
  components: {
    ColorChord
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
