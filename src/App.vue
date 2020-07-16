<template>
  <div id="app">
    <space></space>
  </div>
</template>

<script>
import space from './components/space.vue'

export default {
  name: 'App',
  components: {
    space
  }
}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
