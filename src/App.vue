<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <HandWriting msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
import HandWriting from './components/HandWriting.vue'

export default {
  name: 'App',
  components: {
    HandWriting
  }
}
</script>

<style>
body {
  padding: 0px;
  margin: 0px;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
