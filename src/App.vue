<template>
  <div id="app">
    <Date v-model="now"></Date>
  </div>
</template>

<script>
import Date from "./components/Date.vue"

export default {
  name: "App",
  data(){
    return {
      now: new Date()
    }
  },
  components: {
    Date
  }
}
</script>

<style></style>
