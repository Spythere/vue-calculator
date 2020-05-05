<template>
  <div id="app">
    <Calculator />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Calculator from "./components/Calculator.vue";

@Component({
  components: {
    Calculator
  }
})
export default class App extends Vue {
  mounted() {}
}
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@500&display=swap");

body,
html {
  background: #3f3f3f;
  margin: 0;
  padding: 0;

  font-family: "Poppins", sans-serif;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

#app {
  min-height: 100vh;
}
</style>
