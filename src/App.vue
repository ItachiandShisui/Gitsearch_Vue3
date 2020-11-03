<template>
  <div id="App">
    <Search />
  </div>
</template>

<script>
import Search from "@/components/Search";

export default {
  name: "App",
  components: { Search },
};
</script>

<style>
#app {
  font-family: "Ubuntu", sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin: 5% auto 0 auto;
  width: 90%;
  max-width: 1125px;
}
</style>
