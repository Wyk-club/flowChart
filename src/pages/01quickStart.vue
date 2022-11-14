<template>
  <div class="container">
    <el-button type="primary" @click="graphZoom('+')">缩放+0.2</el-button>
    <el-button type="primary" @click="graphZoom('-')">缩放-0.2</el-button>
    <el-button type="primary" @click="graphCenter">居中</el-button>
    <el-button type="primary" @click="graphToSVG">导出SVG</el-button>
    <el-button type="primary" @click="graphToPNG">导出PNG</el-button>
    <el-button type="primary" @click="graphToJPEG">导出JPEG</el-button>
    <el-button type="primary" @click="graphDispose">销毁画布</el-button>
    <div id="container"></div>
  </div>
</template>

<script>
import { Graph, Shape, DataUri } from '@antv/x6'
export default {
  name: 'flowChart',
  props: {

  },
  data () {
    return {
      graph: null
    }
  },
  mounted () {
    this.init()
    this.createRect()
  },
  methods: {
    // 初始化画布
    init () {
      const data = {
        // 节点
        nodes: [
          {
            id: 'node1', // String，可选，节点的唯一标识
            x: 40, // Number，必选，节点位置的 x 值
            y: 40, // Number，必选，节点位置的 y 值
            width: 80, // Number，可选，节点大小的 width 值
            height: 40, // Number，可选，节点大小的 height 值
            label: 'hello', // String，节点标签
            attrs: {
              rect: {
                fill: '#2ECC71',
                stroke: '#000',
                strokeDasharray: '10,2'
              },
              text: {
                text: 'Hello',
                fill: '#333',
                fontSize: 13
              }
            }
          },
          {
            id: 'node2', // String，节点的唯一标识
            shape: 'ellipse',
            x: 160, // Number，必选，节点位置的 x 值
            y: 180, // Number，必选，节点位置的 y 值
            width: 80, // Number，可选，节点大小的 width 值
            height: 40, // Number，可选，节点大小的 height 值
            label: 'world', // String，节点标签
            attrs: {
              ellipse: {
                fill: '#F39C12',
                stroke: '#000',
                rx: 16,
                ry: 16
              },
              label: {
                text: 'World',
                fill: '#333',
                fontSize: 18,
                fontWeight: 'bold',
                fontVariant: 'small-caps'
              }
            }
          }
        ],
        // 边
        edges: [
          {
            source: 'node1', // String，必须，起始节点 id
            target: 'node2' // String，必须，目标节点 id
          }
        ]
      }
      this.graph = new Graph({
        container: document.getElementById('container'),
        width: 800,
        height: 600,
        panning: { // 画布拖拽
          enabled: true,
          modifiers: 'shift'
        },
        background: {
          color: '#fffbe6' // 设置画布背景颜色
        },
        grid: {
          type: 'mesh',
          size: 10, // 网格大小 10px
          visible: true // 渲染网格背景
        }
      })
      console.log('graph', this.graph)
      this.graph.fromJSON(data)
    },
    // 创建Rect节点
    createRect () {
      const rect = new Shape.Rect({
        x: 200,
        y: 40,
        width: 100,
        height: 40,
        markup: [
          {
            tagName: 'rect',
            selector: 'body',
            attrs: {
              fill: '#fff',
              stroke: '#000',
              strokeWidth: 2
            }
          },
          {
            tagName: 'text',
            selector: 'label',
            attrs: {
              fill: '#333',
              textAnchor: 'middle',
              textVerticalAnchor: 'middle'
            }
          }
        ],
        attrs: { // 节点样式
          rect: { // 使用 rect css 选择器替代预定义的 body 选择器
            fill: '#2ECC71',
            stroke: '#000'
          },
          text: { // 使用 text css 选择器替代预定义的 label 选择器
            text: 'rect',
            fill: '#333',
            fontSize: 13
          }
        }
      })
      this.graph.addNode(rect)
    },

    // 【画布】缩放
    graphZoom (type) {
      switch (type) {
        case '+':
          return this.graph.zoom(0.2)
        case '-':
          return this.graph.zoom(-0.2)
        default:
          return null
      }
    },
    // 【画布】居中
    graphCenter () {
      this.graph.centerContent()
    },
    // 【画布】导出SVG
    graphToSVG () {
      this.graph.toSVG((dataUri) => {
        // 下载
        DataUri.downloadDataUri(DataUri.svgToDataUrl(dataUri), 'chart.svg')
      }, {
        preserveDimensions: {
          width: 100,
          height: 100
        },
        copyStyles: false, // 防止导出时页面闪动
        stylesheet: `
          rect {
            fill: red;
          }
        `
      })
    },
    // 【画布】导出PNG
    graphToPNG () {
      this.graph.toPNG((dataUri) => {
        // 下载
        DataUri.downloadDataUri(dataUri, 'chart.png')
      }, {
        copyStyles: false, // 防止导出时页面闪动
        padding: {
          top: 20,
          right: 30,
          bottom: 40,
          left: 50
        }
      })
    },
    // 【画布】导出JPEG
    graphToJPEG () {
      this.graph.toPNG((dataUri) => {
        // 下载
        DataUri.downloadDataUri(dataUri, 'chart.jpeg')
      }, {
        copyStyles: false, // 防止导出时页面闪动
        padding: {
          top: 20,
          right: 30,
          bottom: 40,
          left: 50
        }
      })
    },
    // 【画布】销毁画布
    graphDispose () {
      this.graph.dispose()
    }
  }
}
</script>

<style lang="scss" scoped>
.x6-cell rect {
  fill: red;
  stroke: #000;
}
</style>
