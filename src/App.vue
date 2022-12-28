<template>
  <div id="app">
    <div class="container-fluid">
      <Table />
    </div>
  </div>
</template>

<script>
import Table from "./components/Table.vue";

export default {
  name: "App",
  components: {
    Table,
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
  /* margin-top: 60px; */
}
</style>
