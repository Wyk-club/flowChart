<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <flowChart></flowChart>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import flowChart from '@/components/flowChart.vue'

export default {
  name: 'HomeView',
  components: {
    // HelloWorld
    flowChart
  }
}
</script>
