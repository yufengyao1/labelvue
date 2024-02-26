<template>
  <div id="app">
    <LabelMain msg="Welcome to Your Vue.js App"/>
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
body{
  margin:0;
}
/* #app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */
</style>
