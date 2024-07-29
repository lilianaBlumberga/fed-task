<template>
  <div id="app">
    <MainPage />
  </div>
</template>

<script>
import MainPage from './components/MainPage.vue'

export default {
  name: 'App',
  components: {
    MainPage
  }
}
</script>

<style>

body {
    font-family: "Oswald", sans-serif;
    background-color: #F8F6F4;
    max-height: 100%;
}
</style>
