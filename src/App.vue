<template>
  <div id="app">
    <threeDWorld></threeDWorld>
  </div>
</template>

<script>
import threeDWorld from "./components/threeDWorld.vue";

export default {
  name: "App",
  components: {
    threeDWorld,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  overflow: hidden;
}
</style>
