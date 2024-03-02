<template>
  <div id="app">
    <LabelMain msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
import LabelMain from './components/LabelMain.vue'

export default {
  name: 'app',
  components: {
    LabelMain
  }
}
</script>

<style>
html {
  overflow: hidden;
  height: 100%;
  width: 100%;
}

body {
  margin: 0px;
  padding: 0px;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  /* color: #2c3e50; */
  margin: 0px;
}
</style>
