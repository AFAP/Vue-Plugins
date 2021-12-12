<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <HandWriting  background="#b0b7f8" type="png" :oncomplete="handwritingComplete" />
    <!-- style="width: 300px;height: 200px;" -->
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
  color: #b0b7f8;
  height: 100vh;
}
</style>
