<template>
  <LayOut></LayOut>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import LayOut from "./components/layOut/LayOut.vue";

export default defineComponent({
  name: "App",
  components: {
    LayOut,
  },
});
</script>

<style>
* {
  font-family: "Montserrat";
}
</style>
