<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <HandWriting style="width: 300px;height: 200px;" :oncomplete="handwritingComplete" />
  </div>
</template>

<script>
import HandWriting from './components/HandWriting.vue'

export default {
  name: 'App',
  components: {
    HandWriting
  },
  methods: {
    handwritingComplete(base64Data) {
      console.log('APP', base64Data)
    }
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}
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
  height: 100vh;
}
</style>
