<template>
  <div id="app">
    <Blackjack />
  </div>
</template>

<script>
import Blackjack from './components/Blackjack.vue'

export default {
  name: 'App',
  components: {
    Blackjack
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
