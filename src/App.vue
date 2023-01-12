<template>
  <div id="app">
    <RedactorX ref="redactorx" v-model="content" :config="configOptions"></RedactorX>
    <!-- {{ content }} -->
  </div>
</template>

<script>
import RedactorX from '@/RedactorX.vue'
export default {
  name: 'App',
  components:{RedactorX},
  data() {
    return {
      content:'',
      configOptions: {}
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
