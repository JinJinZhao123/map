<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
    <HelloWorld />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
*{
  box-sizing: border-box;
}
body{
   background-color:#10aeb5;
}
#app {
  width: 780px;
  margin: 0 auto;
  background-color:#10aeb5;
}
</style>
