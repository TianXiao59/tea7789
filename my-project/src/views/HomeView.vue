<script>
import 'jsmind/style/jsmind.css'
import jsMind from 'jsmind/js/jsmind.js'
window.jsMind = jsMind

require('jsmind/js/jsmind.draggable.js')
require('jsmind/js/jsmind.screenshot.js')
export default {
  props: {
    showBar: { // 是否显示工具栏，显示启用编辑
      type: Boolean,
      default: true
    },
    theme: { // 主题
      type: String,
      default: 'info'
    },
    lineColor: { // 线条颜色
      type: String,
      default: 'skyblue'
    }
  },
  data() {
    return {
      mind: {},
      jm: null,
      isZoomIn: false,
      isZoomOut: false,
      level: 0,
      nodeOptions: [
        { value: 1, label: '展开到一级节点' },
        { value: 2, label: '展开到二级节点' },
        { value: 3, label: '展开到三级节点' },
        { value: 0, label: '展开全部节点' },
        { value: -1, label: '隐藏全部节点' }
      ],
      themeOptions: [
        { value: 'default', label: 'default' },
        { value: 'primary', label: 'primary' },
        { value: 'warning', label: 'warning' },
        { value: 'danger', label: 'danger' },
        { value: 'success', label: 'success' },
        { value: 'info', label: 'info' },
        { value: 'greensea', label: 'greensea' },
        { value: 'nephrite', label: 'nephrite' },
        { value: 'belizehole', label: 'belizehole' },
        { value: 'wisteria', label: 'wisteria' },
        { value: 'asphalt', label: 'asphalt' },
        { value: 'orange', label: 'orange' },
        { value: 'pumpkin', label: 'pumpkin' },
        { value: 'pomegranate', label: 'pomegranate' },
        { value: 'clouds', label: 'clouds' },
        { value: 'asbestos', label: 'asbestos' }
      ],
      localTheme: this.theme,
      dialogVisible: false,
      nodeOption: {
        content: '',
        bgColor: '',
        fontColor: '',
        fontSize: '',
        fontWeight: '',
        fontStyle: ''
      }
    }
  },
  created() {
  },
  mounted() {
    this.getData()
    this.mouseWheel()
  },
  methods: {
    beforeUpload (file) { // 上传文件之前钩子
      if (file) {
        jsMind.util.file.read(file, (jsmindData) => {
          const mind = jsMind.util.json.string2json(jsmindData)
          if (mind) {
            this.jm.show(mind)
            this.$message({ type: 'success', message: '打开成功' })
          } else {
            this.prompt_info('不能打开mindmap文件')
          }
        })
      } else {
        this.prompt_info('请先选择文件')
        return false
      }
    },
    upload() {},
    getData() {
      this.$API({
        name: 'getMind'
      }).then(res => {
        this.mind = res.data
        this.open_empty()
      }).catch(error => {
        this.$message.error(error)
      })
    },
    open_empty() {
      const options = {
        container: 'jsmind_container', // 必选，容器ID
        editable: this.showBar, // 可选，是否启用编辑
        theme: this.localTheme, // 可选，主题
        view: {
          line_width: 2, // 思维导图线条的粗细
          line_color: this.lineColor // 思维导图线条的颜色
        },
        shortcut: {
          enable: true // 禁用快捷键
        },
        layout: {
          hspace: 50, // 节点之间的水平间距
          vspace: 20, // 节点之间的垂直间距
          pspace: 13 // 节点与连接线之间的水平间距（用于容纳节点收缩/展开控制器）
        },
        mode: 'side' // 显示模式，子节点只分布在根节点右侧
      }
      this.jm = jsMind.show(options, this.mind)
      // 改变窗口大小重置画布
      window.onresize = () => {
        this.jm.resize()
      }
    },
    save_nodearray_file() {
      const mindData = this.jm.get_data('node_array')
      const mindName = mindData.meta.name
      const mindStr = jsMind.util.json.json2string(mindData)
      jsMind.util.file.save(mindStr, 'text/jsmind', mindName + '.jm')
    },
    screen_shot() {
      this.jm.screenshot.shootDownload()
    },
    expand_all() {
      this.jm.expand_all()
    },
    collapse_all() {
      this.jm.collapse_all()
    },
    expand_to_level(num) {
      switch (num) {
        case -1:
          this.collapse_all()
          break
        case 0:
          this.expand_all()
          break
        default:
          this.jm.expand_to_depth(num)
          break
      }
    },
    zoomIn() {
      if (this.jm.view.zoomIn()) {
        this.isZoomOut = false
      } else {
        this.isZoomIn = true
      }
    },
    zoomOut() {
      if (this.jm.view.zoomOut()) {
        this.isZoomIn = false
      } else {
        this.isZoomOut = true
      }
    },
    prompt_info(msg) {
      this.$message({ type: 'warning', message: msg})
    },
    get_nodearray_data() {
      const mindData = this.jm.get_data('node_array')
      const mindString = jsMind.util.json.json2string(mindData)
      this.$message({ type: 'info', message: mindString})
    },
    set_theme(themeName) {
      this.jm.set_theme(themeName)
    },
    scrollFunc(e) {
      e = e || window.event
      if (e.wheelDelta) {
        if (e.wheelDelta > 0) {
          this.zoomIn()
        } else {
          this.zoomOut()
        }
      } else if (e.detail) {
        if (e.detail > 0) {
          this.zoomIn()
        } else {
          this.zoomOut()
        }
      }
      this.jm.resize()
    },
    // 鼠标滚轮放大缩小
    mouseWheel() {
      if (document.addEventListener) {
        document.addEventListener('domMouseScroll', this.scrollFunc, false)
      }
      this.$refs.container.onmousewheel = this.scrollFunc
    },
    // 新增节点
    addNode() {
      let selectedNode = this.jm.get_selected_node()
      if (!selectedNode) {
        this.$message({ type: 'warning', message: '请先选择一个节点!'})
        return
      }
      let nodeid = jsMind.util.uuid.newid()
      let topic = 'new Node'
      let newNode = this.jm.add_node(selectedNode, nodeid, topic)
      if (newNode) {
        this.jm.select_node(nodeid)
        this.jm.begin_edit(nodeid)
      }
    },
    // 新增兄弟节点
    addBrotherNode() {
      let selectedNode = this.jm.get_selected_node()
      if (!selectedNode) {
        this.$message({ type: 'warning', message: '请先选择一个节点!'})
        return
      } else if (selectedNode.isroot) {
        this.$message({ type: 'warning', message: '不能在根节点添加，请重新选择节点!'})
        return
      }
      let nodeid = jsMind.util.uuid.newid()
      let topic = 'new Node'
      let newNode = this.jm.insert_node_after(selectedNode, nodeid, topic)
      if (newNode) {
        this.jm.select_node(nodeid)
        this.jm.begin_edit(nodeid)
      }
    },
    // 获取选中标签的 ID
    get_selected_nodeid () {
      let selectedNode = this.jm.get_selected_node()
      if (selectedNode) {
        return selectedNode.id
      } else {
        return null
      }
    },
    // 删除节点
    removeNode() {
      let selectedId = this.get_selected_nodeid()
      if (!selectedId) {
        this.$message({
          type: 'warning',
          message: '请先选择一个节点!'
        })
        return
      }
      this.jm.remove_node(selectedId)
    },
    // 编辑节点
    editNode () {
      let selectedId = this.get_selected_nodeid()
      if (!selectedId) {
        this.$message({ type: 'warning', message: '请先选择一个节点!'})
        return
      }
      let nodeObj = this.jm.get_node(selectedId)
      this.nodeOption.content = nodeObj.topic
      this.nodeOption.bgColor = nodeObj.data['background-color']
      this.nodeOption.fontColor = nodeObj.data['foreground-color']
      this.nodeOption.fontSize = nodeObj.data['font-size']
      this.nodeOption.fontWeight = nodeObj.data['font-weight']
      this.nodeOption.fontStyle = nodeObj.data['font-style']
      this.dialogVisible = true
    },
    sureEditNode () {
      let selectedId = this.get_selected_nodeid()
      this.jm.update_node(selectedId, this.nodeOption.content)
      this.jm.set_node_font_style(selectedId, this.nodeOption.fontSize, this.nodeOption.fontWeight, this.nodeOption.fontStyle)
      this.jm.set_node_color(selectedId, this.nodeOption.bgColor, this.nodeOption.fontColor)
      this.nodeOption = {
        content: '',
        bgColor: '',
        fontColor: '',
        fontSize: '',
        fontWeight: '',
        fontStyle: ''
      }
      this.dialogVisible = false
    }
  },
  beforeDestroy() {
    document.removeEventListener('domMouseScroll', this.scrollFunc, false)
  }
}
</script>


<style lang="less" scoped>
.jsmind_layout {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: calc(100% - 40px);
  overflow: hidden;
  .jsmind_toolbar {
    width: 100%;
    padding: 0 10px 10px 10px;
    height: auto;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    background-color: #f8f9fa;
    box-shadow: 0 0 4px #b8b8b8;
  }
  /deep/ .el-button--medium, /deep/ .el-input--medium {
    margin-top: 10px;
  }
  #jsmind_container {
    flex: 1 1 auto;
  }
  /deep/.jsmind-inner {
    overflow: hidden auto !important;
  }
  /deep/.el-upload-list {
    display: none !important;
  }
  /* 隐藏滚动条 */
  .jsmind-inner::-webkit-scrollbar {
    display: none;
  }
  .pad {
    margin-right: 10px;
  }
  .pad-left {
    margin-left: 10px;
  }
  /deep/ jmnode.selected {
    background-color: #b9b9b9;
    color: #fff;
    box-shadow: 2px 2px 8px #777;
  }
  /deep/ jmnode:hover {
    box-shadow: 2px 2px 8px #777;
  }
  .form-con {
    padding-top:20px;
  }
  .ele-width {
    width: 96%;
  }
}
</style>


