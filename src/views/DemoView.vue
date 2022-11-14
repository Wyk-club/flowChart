<template>
  <div class="demo_view">
    <!-- <div id="graph-container">
    </div>
    <div id="stencil"></div> -->
    <div id="container">
      <div class="header">
        <div class="select_wrapper">
          <h2 class="title">元素</h2>
          <el-select v-model="optionValue" placeholder="请选择">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
          <el-button @click="undo">撤销</el-button>
          <el-button v-popover:popover>激活</el-button>
          <el-button @click="save">保存画布</el-button>
        </div>
        <div id="flow_item"></div>
      </div>
      <div id="content"></div>

      <el-popover
        ref="popover"
        placement="right"
        width="400"
        v-model="showPoper"
        trigger="click">
        <el-table :data="gridData">
          <el-table-column width="150" property="date" label="日期"></el-table-column>
          <el-table-column width="100" property="name" label="姓名"></el-table-column>
          <el-table-column width="300" property="address" label="地址"></el-table-column>
        </el-table>
      </el-popover>

      <el-dialog title="收货地址" :visible.sync="dialogTableVisible">
        1
      </el-dialog>

    </div>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import { Graph, Shape, Addon } from '@antv/x6'
import insertCss from 'insert-css'
export default {
  name: 'DemoView',
  components: {
  },
  data () {
    return {
      options: [{
        value: '选项1',
        label: '黄金糕'
      }, {
        value: '选项2',
        label: '双皮奶'
      }, {
        value: '选项3',
        label: '蚵仔煎'
      }, {
        value: '选项4',
        label: '龙须面'
      }, {
        value: '选项5',
        label: '北京烤鸭'
      }],
      optionValue: '选项1',
      gridData: [{
        date: '2016-05-02',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }, {
        date: '2016-05-04',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }, {
        date: '2016-05-01',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }, {
        date: '2016-05-03',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }],
      dialogTableVisible: false,
      showPoper: false,
      graphData: {}
    }
  },
  mounted () {
    this.preWork()
    this.init()
    this.eventListening()
    // window.addEventListener('resize', function () {
    //   this.graph.resizeGraph()
    // })
  },
  methods: {
    // 初始化画布
    init () {
      const graph = new Graph({
        container: document.getElementById('content'),
        grid: {
          visible: true,
          type: 'mesh'
        },
        // autoResize: true,
        panning: { // 拖拽平移 not
          enabled: true,
          modifiers: 'shift'
        },
        // scroller: { // not
        // enabled: true,
        // padding: 0
        // pannable: true,
        // pageVisible: true,
        // pageBreak: false,
        // autoResize: true,
        // },
        history: { // 撤销/重做
          enabled: true
        },
        mousewheel: { // not
          enabled: true,
          zoomAtMousePosition: true,
          modifiers: 'ctrl',
          minScale: 0.5,
          maxScale: 3
        },
        connecting: { // not
          router: {
            name: 'manhattan',
            args: {
              padding: 1
            }
          },
          connector: {
            name: 'rounded',
            args: {
              radius: 8
            }
          },
          anchor: 'center',
          connectionPoint: 'anchor',
          allowBlank: false,
          snap: {
            radius: 20
          },
          createEdge () {
            return new Shape.Edge({
              attrs: {
                line: {
                  stroke: '#A2B1C3',
                  strokeWidth: 2,
                  targetMarker: {
                    name: 'block',
                    width: 12,
                    height: 8
                  }
                }
              },
              zIndex: 0
            })
          },
          validateConnection ({ targetMagnet }) {
            return !!targetMagnet
          }
        },
        highlighting: { // not
          magnetAdsorbed: {
            name: 'stroke',
            args: {
              attrs: {
                fill: '#5F95FF',
                stroke: '#5F95FF'
              }
            }
          }
        },
        resizing: true, // not
        rotating: true, // not
        selecting: { // 点选/框选 not
          enabled: true,
          rubberband: true,
          showNodeSelectionBox: true,
          className: 'my-selecting'
        },
        // 下面都是 not
        snapline: true,
        keyboard: true,
        clipboard: true
      })
      // #endregion

      // #region 初始化 stencil
      const stencil = new Addon.Stencil({
        // dndContainer: document.getElementById('flow_item'),
        title: '',
        target: graph,
        stencilGraphWidth: 800,
        stencilGraphHeight: 180,
        collapsable: true,
        // groups: [
        //   {
        //     title: '基础流程图',
        //     name: 'group1'
        //   }
        //   {
        //     title: '系统设计图',
        //     name: 'group2',
        //     graphHeight: 250,
        //     layoutOptions: {
        //       rowHeight: 70
        //     }
        //   }
        // ],
        layoutOptions: {
          columns: 8,
          columnWidth: 80,
          rowHeight: 55
        }
      })
      document.getElementById('flow_item').appendChild(stencil.container)
      // #endregion

      // #region 快捷键与事件
      // copy cut paste
      graph.bindKey(['meta+c', 'ctrl+c'], () => {
        const cells = graph.getSelectedCells()
        if (cells.length) {
          graph.copy(cells)
        }
        return false
      })
      graph.bindKey(['meta+x', 'ctrl+x'], () => {
        const cells = graph.getSelectedCells()
        if (cells.length) {
          graph.cut(cells)
        }
        return false
      })
      graph.bindKey(['meta+v', 'ctrl+v'], () => {
        if (!graph.isClipboardEmpty()) {
          const cells = graph.paste({ offset: 32 })
          graph.cleanSelection()
          graph.select(cells)
        }
        return false
      })

      // undo redo
      graph.bindKey(['meta+z', 'ctrl+z'], () => {
        if (graph.history.canUndo()) {
          graph.history.undo()
        }
        return false
      })
      graph.bindKey(['meta+shift+z', 'ctrl+shift+z'], () => {
        if (graph.history.canRedo()) {
          graph.history.redo()
        }
        return false
      })

      // select all
      graph.bindKey(['meta+a', 'ctrl+a'], () => {
        const nodes = graph.getNodes()
        if (nodes) {
          graph.select(nodes)
        }
      })

      // delete
      graph.bindKey('backspace', () => {
        const cells = graph.getSelectedCells()
        if (cells.length) {
          graph.removeCells(cells)
        }
      })

      // zoom
      graph.bindKey(['ctrl+1', 'meta+1'], () => {
        const zoom = graph.zoom()
        if (zoom < 1.5) {
          graph.zoom(0.1)
        }
      })
      graph.bindKey(['ctrl+2', 'meta+2'], () => {
        const zoom = graph.zoom()
        if (zoom > 0.5) {
          graph.zoom(-0.1)
        }
      })

      // 控制连接桩显示/隐藏
      const showPorts = (ports, show) => {
        for (let i = 0, len = ports.length; i < len; i = i + 1) {
          ports[i].style.visibility = show ? 'visible' : 'hidden'
        }
      }

      // #region 初始化图形
      const ports = {
        groups: {
          top: {
            position: 'top',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          },
          right: {
            position: 'right',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          },
          bottom: {
            position: 'bottom',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          },
          left: {
            position: 'left',
            attrs: {
              circle: {
                r: 4,
                magnet: true,
                stroke: '#5F95FF',
                strokeWidth: 1,
                fill: '#fff',
                style: {
                  visibility: 'hidden'
                }
              }
            }
          }
        },
        items: [
          {
            group: 'top'
          },
          {
            group: 'right'
          },
          {
            group: 'bottom'
          },
          {
            group: 'left'
          }
        ]
      }

      const customRect = Graph.registerNode(
        'custom-rect',
        {
          inherit: 'rect',
          width: 66,
          height: 36,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626'
            }
          },
          tools: [
            {
              name: 'button-remove',
              args: {
                x: '100%',
                y: 0,
                offset: { x: -10, y: 10 }
              }
            }
          ],
          ports: { ...ports }
        },
        true
      )

      const customPolygon = Graph.registerNode(
        'custom-polygon',
        {
          inherit: 'polygon',
          width: 66,
          height: 36,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626'
            }
          },
          ports: {
            ...ports,
            items: [
              {
                group: 'top'
              },
              {
                group: 'bottom'
              }
            ]
          }
        },
        true
      )

      const customCircle = Graph.registerNode(
        'custom-circle',
        {
          inherit: 'circle',
          width: 45,
          height: 45,
          attrs: {
            body: {
              strokeWidth: 1,
              stroke: '#5F95FF',
              fill: '#EFF4FF'
            },
            text: {
              fontSize: 12,
              fill: '#262626'
            }
          },
          ports: { ...ports }
        },
        true
      )

      const customImage = Graph.registerNode(
        'custom-image',
        {
          inherit: 'rect',
          width: 52,
          height: 52,
          markup: [
            {
              tagName: 'rect',
              selector: 'body'
            },
            {
              tagName: 'image'
            },
            {
              tagName: 'text',
              selector: 'label'
            }
          ],
          attrs: {
            body: {
              stroke: '#5F95FF',
              fill: '#5F95FF'
            },
            image: {
              width: 26,
              height: 26,
              refX: 13,
              refY: 16
            },
            label: {
              refX: 3,
              refY: 2,
              textAnchor: 'left',
              textVerticalAnchor: 'top',
              fontSize: 12,
              fill: '#fff'
            }
          },
          ports: { ...ports }
        },
        true
      )

      const r1 = graph.createNode({
        shape: 'custom-rect',
        label: '开始',
        attrs: {
          body: {
            rx: 20,
            ry: 26
          }
        }
      })
      const r2 = graph.createNode({
        shape: 'custom-rect',
        label: '过程'
      })
      const r3 = graph.createNode({
        shape: 'custom-rect',
        attrs: {
          body: {
            rx: 6,
            ry: 6
          }
        },
        label: '可选过程'
      })
      const r4 = graph.createNode({
        shape: 'custom-polygon',
        attrs: {
          body: {
            refPoints: '0,10 10,0 20,10 10,20'
          }
        },
        label: '决策'
      })
      const r5 = graph.createNode({
        shape: 'custom-polygon',
        attrs: {
          body: {
            refPoints: '10,0 40,0 30,20 0,20'
          }
        },
        label: '数据'
      })
      const r6 = graph.createNode({
        shape: 'custom-circle',
        label: '连接'
      })
      // stencil.load([r1, r2, r3, r4, r5, r6], 'group1')
      stencil.load([r1, r2, r3, r4, r5, r6])

      const targetList = [r1, r2, r3, r4, r5, r6]
      graph.on('node:mouseenter', ({ node }) => {
        const container = document.getElementById('content')
        const ports = container.querySelectorAll(
          '.x6-port-body'
        )
        showPorts(ports, true)

        console.log(targetList, ' targetList')
        console.log('node', node)
        if (targetList.indexOf(node) > 0) {
          node.addTools({
            name: 'button-remove',
            args: {
              x: '100%',
              y: 0,
              offset: { x: -10, y: 10 }
            }
          })
        }
      })
      graph.on('node:mouseleave', ({ node }) => {
        const container = document.getElementById('content')
        const ports = container.querySelectorAll(
          '.x6-port-body'
        )
        showPorts(ports, false)

        if (targetList.indexOf(node) > 0) {
          node.removeTools()
        }
      })

      let nodes = []
      let edges = []
      if (sessionStorage.getItem('nodes')) {
        // 拿到之前画布保存时保存在缓存中的节点数据
        nodes = JSON.parse(sessionStorage.getItem('nodes'))
      }
      if (sessionStorage.getItem('edges')) {
        // 拿到之前画布保存时保存在缓存中的边数据
        edges = JSON.parse(sessionStorage.getItem('edges'))
      }
      graph.fromJSON({
        nodes: nodes,
        edges: edges
      })
      this.graph = graph
    },
    // 准备工作
    preWork () {
      // 这里协助演示的代码，在实际项目中根据实际情况进行调整
      // const container = document.getElementById('container')
      // const stencilContainer = document.createElement('div')
      // stencilContainer.id = 'stencil'
      // const graphContainer = document.createElement('div')
      // graphContainer.id = 'graph-container'
      // container.appendChild(stencilContainer)
      // container.appendChild(graphContainer)

      insertCss(`
        #container {
        }
        #stencil {
          height: 180px;
          position: relative;
          border-right: 1px solid #dfe3e8;
        }
        #graph-container {
          width: 100%;
          height: calc(100% - 100px);
          border: 1px solid #eee;
        }
        .x6-widget-stencil  {
          background-color: #fff;
        }
        .x6-widget-stencil-title {
          background-color: #fff;
        }
        .x6-widget-stencil-group-title {
          background-color: #fff !important;
        }
        .x6-widget-transform {
          margin: -1px 0 0 -1px;
          padding: 0px;
          border: 1px solid #239edd;
        }
        .x6-widget-transform > div {
          border: 1px solid #239edd;
        }
        .x6-widget-transform > div:hover {
          background-color: #3dafe4;
        }
        .x6-widget-transform-active-handle {
          background-color: #3dafe4;
        }
        .x6-widget-transform-resize {
          border-radius: 0;
        }
        .x6-widget-selection-inner {
          border: 1px solid #239edd;
        }
        .x6-widget-selection-box {
          opacity: 0;
        }
      `)
    },
    // 【画布】撤销
    undo () {
      this.graph.undo()
    },
    // 【画布】事件监听
    eventListening () {
      // 点击
      this.graph.on('cell:click', ({ e, x, y, cell, view }) => {
        console.log('点击事件，节点信息：')
        console.log('e：', e)
        console.log('x：', x)
        console.log('y：', y)
        console.log('cell：', cell)
        console.log('view：', view)
        console.log('------------------------------')
        console.log(this.$refs.popover.doShow)
        this.$refs.popover.doShow()
        // this.dialogTableVisible = true
      })

      // 点/选事件
      // this.graph.on('node:selected', ({ cell, node, options }) => {
      //   console.log('选中', {
      //     cell, node, options
      //   })
      // })
    },
    // 【画布】保存 写入到session中
    save () {
      window.sessionStorage.setItem('nodes', JSON.stringify(this.graph.getNodes())) // 保存节点
      window.sessionStorage.setItem('edges', JSON.stringify(this.graph.getEdges())) // 保存边
      this.$message({
        message: '保存成功',
        type: 'success'
      })
    }
  }
}
</script>

<style lang="scss" scoped>
.demo_view {
  height: 100%;
  #container {
    position: relative;
    width: 100%;
    height: 100%;
    .header {
      display: flex;
      height: 100px;
      // border: 1px solid red;
      .select_wrapper {
        display: flex;
        align-items: center;
        // justify-content: space-between;
        padding: 0 10px 0;
        width: 400px;
        height: 100%;
        // border: 1px solid red;
        border-right: 1px solid #eee;
        box-sizing: border-box;
        .title {
          margin-right: 10px;
        }
      }
      #flow_item {
        width: calc(100% - 400px);
      }
      ::v-deep .x6-widget-stencil {
        position: static;
        .x6-widget-stencil-title {
          height: 15px;
        }
      }
      ::v-deep .x6-widget-stencil-content {
        position: static;
      }
    }
    #content {
      // position: absolute;
      // top: 100px;
      // left: 0;
      // right: 0;
      // bottom: 0;
      // height: calc(100% - 100px);

      width: 100%;
      height: 100%;
    }
  }
  /*尝试修改多选样式 无效*/
  ::v-deep .my-selecting {
    color: red;
    .x6-widget-selection-box {
      border: 1px dashed red;
    }
  }
  /*尝试修改scroller蒙层样式 无效*/
  ::v-deep .x6-graph-scroller-content {
    height: calc(100% - 100px);
  }
}
</style>
