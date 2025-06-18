<template>
  <div id="app">
    <BlogPage />
  </div>
</template>

<script>
import BlogPage from './components/BlogPage.vue'

export default {
  name: 'App',
  components: {
    BlogPage
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  height: 100vh;
  overflow: hidden;
}

body {
  margin: 0;
  padding: 0;
}
</style>
