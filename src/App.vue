<template>
  <div id="app">
    <HomePage msg="Welcome to Your Vue.js App" class="full-height"/>
    <ListPage class="auto-height"/>
  </div>
</template>

<script>
import HomePage from './components/HomePage.vue'
import ListPage from './components/ListPage.vue'

export default {
  name: 'App',
  components: {
    HomePage,
    ListPage
  }
}
</script>

<style>
#app {
  display: flex;
  flex-direction: column; /* Stack components vertically */
  min-height: 100vh; /* Minimum height to viewport height */
  overflow: auto; /* Enable scrolling */
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.full-height {
  min-height: 100vh; /* Ensure minimum viewport height */
  width: 100%; /* Ensure full width */
}

.auto-height {
  flex: 1; /* Allow to grow with content */
  min-height: 100vh; /* Ensure minimum viewport height */
  width: 100%; /* Ensure full width */
}
</style>
