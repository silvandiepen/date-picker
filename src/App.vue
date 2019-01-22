<template>
  <div id="app">
    <div class="container">
      <div class="content">
      <h1>Date Picker</h1>
        
      </div>
      <div class="content">
        {{ currentDate }}
        <date-picker v-model="currentDate" />
      </div>
    </div>
    <sil-footer></sil-footer>
  </div>
</template>

<script>
import silFooter from "./components/footer.vue";
import datePicker from "./components/date-picker.vue";
// import Grid from "@sil/grid";
// const Row = Grid.row;
// const Column = Grid.column;

export default {
  name: "app",
  components: {
    // Column,
    // Row,
    datePicker,
    silFooter
  },
  data() {
    return {
      currentDate: 20190122
    };
  }
};
</script>

<style lang="scss">
@import "~@sil/base-style/src/scss/index.full";
</style>
