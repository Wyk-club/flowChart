<template>
  <div class="flow_chart">
    <!-- <div id="container" style="width: 600px; height: 400px"></div> -->
    <router-link to="/pages/01">快速上手</router-link>
    <router-view></router-view>
  </div>
</template>

<script>
import { Graph } from '@antv/x6'
export default {
  name: 'flowChart',
  props: {

  },
  mounted () {
    // this.init()
  },
  methods: {
    init () {
      const graph = new Graph({
        container: document.getElementById('container'),
        grid: true
      })
      const source = graph.addNode({
        x: 300,
        y: 40,
        width: 80,
        height: 40,
        label: 'Hello'
      })

      const target = graph.addNode({
        x: 420,
        y: 180,
        width: 80,
        height: 40,
        label: 'World'
      })

      graph.addEdge({
        source,
        target
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
