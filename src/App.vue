<template>
  <div id="app">
    <img alt="SpeechKit logo" src="https://github.com/mastashake08/speech-kit/blob/main/cover.png?raw=true" width="100%" height="500">
    <HelloWorld msg="SpeechKit Demo"/>
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
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 0px;
}
</style>
