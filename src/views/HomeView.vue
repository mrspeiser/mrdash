<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <ChartWrapper msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import ChartWrapper from '@/components/ChartWrapper.vue'

export default {
  name: 'HomeView',
  components: {
    ChartWrapper
  }
}
</script>
