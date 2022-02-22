<template>
  <div class="container_warp">
    <div id="containerChart"></div>
    <RightDrawer
      class="right_drawer"
      :drawerType="type"
      :selectCell="selectCell"
      :graph="graph"
      :grid="grid"
      @deleteNode="deleteNode"
    ></RightDrawer>
    <div class="operating">
      <div class="btn-group">
        <el-tooltip
          class="item"
          effect="dark"
          content="圆形"
          placement="bottom-start"
        >
          <div class="btn" @mousedown="startDrag('Circle', $event)">
            <el-button
              size="mini"
              class="iconfont icon-yuanxingweixuanzhong"
            ></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="矩形"
          placement="bottom-start"
        >
          <div class="btn" @mousedown="startDrag('Rect', $event)">
            <el-button
              size="mini"
              class="iconfont icon-24gl-square"
            ></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="条件节点"
          placement="bottom-start"
        >
          <div class="btn" @mousedown="startDrag('polygon', $event)">
            <el-button size="mini" class="iconfont icon-lingxing"></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="文本节点"
          placement="bottom-start"
        >
          <div class="btn" @mousedown="startDrag('TextBlock', $event)">
            <el-button size="mini" class="iconfont icon-wenben"></el-button>
          </div>
        </el-tooltip>
        <div class="btn-group_tips" v-if="showTips">
          拖拽生成<br />资产拓扑图形
        </div>
      </div>
      <div class="btn-group">
        <!-- 使用element ui -->
        <el-tooltip
          class="item"
          effect="dark"
          content="直线箭头"
          placement="bottom-start"
        >
          <div
            :class="['btn', currentArrow === 1 ? 'currentArrow' : '']"
            @click="changeEdgeType('normal')"
          >
            <el-button
              size="mini"
              class="iconfont icon-jiantou_zuoxia_o"
            ></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="曲线箭头"
          placement="bottom-start"
        >
          <div
            :class="['btn', currentArrow === 2 ? 'currentArrow' : '']"
            @click="changeEdgeType('smooth')"
          >
            <el-button size="mini" class="iconfont icon-Down-Right"></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="直角箭头"
          placement="bottom-start"
        >
          <div
            :class="['btn', currentArrow === 3 ? 'currentArrow' : '']"
            @click="changeEdgeType('rightAngle')"
          >
            <el-button
              size="mini"
              class="iconfont icon-subdirectory_arrow_right"
            ></el-button>
          </div>
        </el-tooltip>
        <el-tooltip
          class="item"
          effect="dark"
          content="虚线箭头"
          placement="bottom-start"
        >
          <div
            :class="['btn', currentArrow === 4 ? 'currentArrow' : '']"
            @click="changeEdgeType('dotted')"
          >
            <el-button
              size="mini"
              class="iconfont icon-xuxianjiantou"
            ></el-button>
          </div>
        </el-tooltip>
      </div>
      <!-- <div class="btn-group">
          <div class="btn" @click="changeMode('edit')" title="选择模式">
            <i class="iconfont icon-mousepointershubiao"></i>
          </div>
          <div class="btn" @click="changeMode('drag')" title="拖拽模式">
            <i class="iconfont icon-tuozhuai"></i>
          </div>
        </div> -->
      <div class="btn-group">
        <!-- 使用element ui -->
        <el-tooltip
          class="item"
          effect="dark"
          content="删除"
          placement="bottom-start"
        >
          <div class="btn" @click="deleteNode()" style="margin-top: 5px;">
            <el-button size="mini" class="iconfont icon-delete"></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="保存PNG"
          placement="bottom-start"
        >
          <div class="btn" @click="saveToPNG()">
            <el-button
              size="mini"
              class="iconfont icon-baocuntupian"
            ></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="撤销"
          placement="bottom-start"
        >
          <div class="btn" @click="undo()">
            <el-button size="mini" class="iconfont icon-chexiao"></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="回退"
          placement="bottom-start"
        >
          <div class="btn" @click="redo()">
            <el-button size="mini" class="iconfont icon-huitui"></el-button>
          </div>
        </el-tooltip>

        <el-tooltip
          class="item"
          effect="dark"
          content="展示数据"
          placement="bottom-start"
        >
          <div class="btn" @click="showData()">
            <el-button size="mini" class="iconfont icon-huitui"></el-button>
          </div>
        </el-tooltip>
        <el-tooltip
          class="item"
          effect="dark"
          content="加载数据"
          placement="bottom-start"
        >
          <div class="btn" @click="loadData()">
            <el-button size="mini" class="iconfont icon-huitui"></el-button>
          </div>
        </el-tooltip>
      </div>
    </div>
  </div>
</template>
<script>
import "@antv/x6-vue-shape";
import {
  Graph,
  Shape,
  Addon,
  FunctionExt,
  DataUri,
  Edge,
  cell
} from "@antv/x6";
import RightDrawer from "./components/RightDrawer";
import insertCss from "insert-css";
import { startDragToGraph } from "./Graph/methods.js";
import loadData from "../data/data.js";
const data = {};
export default {
  data() {
    return {
      graph: "",
      value1: true,
      type: "grid",
      selectCell: "",
      connectEdgeType: {
        //连线方式
        connector: "normal",
        router: {
          name: ""
        }
      },
      strokeDasharray: "0",
      showTips: false,
      currentArrow: 1,
      grid: {
        // 网格设置
        size: 20, // 网格大小 10px
        visible: true, // 渲染网格背景
        type: "mesh",
        args: {
          color: "#D0D0D0",
          thickness: 1, // 网格线宽度/网格点大小
          factor: 10
        }
      },
      loadDataFlag: true
    };
  },
  components: {
    RightDrawer
  },
  methods: {
    initX6() {
      var _that = this;
      this.graph = new Graph({
        container: document.getElementById("containerChart"),
        width: 1700,
        height: "100%",
        grid: _that.grid,
        resizing: {
          //调整节点宽高
          enabled: _that.loadDataFlag,
          orthogonal: false
        },
        selecting: {
          enabled: true,
          rubberband: true,
          showNodeSelectionBox: true,
          showEdgeSelectionBox: true,
          movable:_that.loadDataFlag
        },
        snapline: true,
        interacting: {
          edgeMovable: _that.loadDataFlag,
          edgeLabelMovable: _that.loadDataFlag,
          arrowheadMovable: _that.loadDataFlag,
          vertexMovable: _that.loadDataFlag,
          vertexAddable: _that.loadDataFlag,
          vertexDeletable: _that.loadDataFlag,
          useEdgeTools: _that.loadDataFlag,
          nodeMovable: _that.loadDataFlag,
          magnetConnectable: _that.loadDataFlag,
          stopDelegateOnDragging: _that.loadDataFlag
        },
        connecting: {
          // 节点连接
          anchor: "center",
          connectionPoint: "anchor",
          allowBlank: true,
          snap: true,
          createEdge() {
            return new Shape.Edge({
              attrs: {
                line: {
                  stroke: "#1890ff",
                  strokeWidth: 1,
                  targetMarker: {
                    name: "classic",
                    size: 8
                  },
                  strokeDasharray: _that.strokeDasharray //虚线
                  // style: {
                  //   animation: "ant-line 30s infinite linear"
                  // }
                }
              },
              label: {
                text: ""
              },
              connector: _that.connectEdgeType.connector,
              router: {
                name: _that.connectEdgeType.router.name || ""
              },
              zIndex: 0
            });
          }
        },
        highlighting: {
          magnetAvailable: {
            name: "stroke",
            args: {
              padding: 4,
              attrs: {
                strokeWidth: 4,
                stroke: "#6a6c8a"
              }
            }
          }
        },
        history: true,
        keyboard: {
          enabled: true
        }
      });
      insertCss(`
              @keyframes ant-line {
                to {
                    stroke-dashoffset: -1000
                }
              }
            `);
      this.graph.history.redo();
      this.graph.history.undo();
      this.graph.bindKey("delete", e => {
        if (_that.loadDataFlag === true) {
          this.deleteNode();
        }
      });
      // 鼠标移入移出节点
      this.graph.on(
        "node:mouseenter",
        FunctionExt.debounce(() => {
          if (_that.loadDataFlag === true) {
            const container = document.getElementById("containerChart");
            const ports = container.querySelectorAll(".x6-port-body");
            this.showPorts(ports, true);
          }
        }),
        500
      );
      this.graph.on("node:mouseleave", () => {
        if (_that.loadDataFlag === true) {
          const container = document.getElementById("containerChart");
          const ports = container.querySelectorAll(".x6-port-body");
          this.showPorts(ports, false);
        }
      });
      // blank 画布空白区域
      this.graph.on("blank:click", () => {
        this.type = "grid";
      });
      // cell 节点/边
      this.graph.on("cell:click", ({e, cell,view  }) => {
        this.type = cell.isNode() ? "node" : "edge";
        // console.log(e);
      });
      // 选中连线后，移动node出现选中的bug
      this.graph.on("node:moving", ({ e, x, y, node, view }) => {
        this.graph.cleanSelection();
      });
      this.graph.on("selection:changed", args => {
        args.added.forEach(cell => {
          console.log(cell)
          this.selectCell = cell;
          if (cell.isEdge()) {
            // cell.isEdge() && cell.attr("line/strokeDasharray", 5); //虚线蚂蚁线
            cell.isEdge() && cell.attr("line/strokeDasharray", cell.store.data.attrs.line.strokeDasharray);
            cell.addTools([
              {
                name: "vertices",
                args: {
                  padding: 4,
                  attrs: {
                    strokeWidth: 0.1,
                    stroke: "#2d8cf0",
                    fill: "#ffffff"
                  }
                }
              }
            ]);
          }
        });
        args.removed.forEach(cell => {
          // cell.isEdge() && cell.attr("line/strokeDasharray", 0); //正常线
          cell.isEdge() && cell.attr("line/strokeDasharray", cell.store.data.attrs.line.strokeDasharray); //正常线
          cell.removeTools();
        });
      });
    },
    showPorts(ports, show) {
      for (let i = 0, len = ports.length; i < len; i = i + 1) {
        ports[i].style.visibility = show ? "visible" : "hidden";
      }
    },
    // 选择模式或者拖拽模式
    // edit选择模式 drag拖拽模式
    // changeMode(mode){
    //   console.log(mode)
    //   if(mode === 'edit'){
    //     this.graph.enableRubberband();
    //   }else if(mode === 'drag'){
    //     this.graph.disableRubberband();
    //   }
    // },
    showData() {
      console.log(this.graph.toJSON());
      console.log(JSON.stringify(this.graph.toJSON()));
    },
    loadData() {
      this.loadDataFlag = false;
      this.initX6();
      setTimeout(() => {
        this.graph.fromJSON(loadData);
      }, 0);
      setTimeout(() => {
        const container = document.getElementById("containerChart");
        const ports = container.querySelectorAll(".x6-port-body");
        this.showPorts(ports, false);
      }, 0);
    },
    // 拖拽生成正方形或者圆形
    startDrag(type, e) {
      startDragToGraph(this.graph, type, e);
    },
    // 删除节点
    deleteNode() {
      const cell = this.graph.getSelectedCells();
      this.graph.removeCells(cell);
      this.type = "grid";
    },
    // 保存png
    saveToPNG() {
      this.$nextTick(() => {
        this.graph.toPNG(
          dataUri => {
            // 下载
            DataUri.downloadDataUri(dataUri, "资产拓扑图.png");
          },
          {
            backgroundColor: "white",
            padding: {
              top: 50,
              right: 50,
              bottom: 50,
              left: 50
            },
            quality: 1,
            copyStyles: false
          }
        );
      });
    },
    undo() {
      this.graph.history.undo();
    },
    redo() {
      this.graph.history.redo();
    },
    // 改变边形状
    changeEdgeType(e) {
      if (e === "normal") {
        this.connectEdgeType = {
          connector: "normal",
          router: { name: "" }
        };
        this.strokeDasharray = 0;
        this.currentArrow = 1;
      } else if (e === "smooth") {
        this.connectEdgeType = {
          connector: "smooth",
          router: { name: "" }
        };
        this.strokeDasharray = 0;
        this.currentArrow = 2;
      } else if (e === "rightAngle") {
        this.connectEdgeType = {
          connector: "normal",
          router: { name: "manhattan" }
        };
        this.strokeDasharray = 0;
        this.currentArrow = 3;
      } else {
        this.connectEdgeType = {
          connector: "smooth",
          router: { name: "" }
        };
        this.strokeDasharray = "5,5";
        this.currentArrow = 4;
      }
    }
  },
  mounted() {
    this.initX6();
    setTimeout(() => {
      this.showTips = true;
    }, 1000);
    setTimeout(() => {
      this.showTips = false;
    }, 5000);
  }
};
</script>
<style lang="less" scoped>
@import "./index.less";
.iconfont {
  font-size: 20px;
}
</style>
