<template>
  <div>
    <el-switch v-model="allowDraggable" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button
      @click="batchUpdate"
      v-if="allowDraggable"
      size="mini"
      round
      type="primary"
      style="padding-left: 15px;"
    >批量修改</el-button>
    <el-button @click="batchDelete" type="danger" round size="mini">批量删除</el-button>
    <el-tree
      node-key="catId"
      show-checkbox
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedIds"
      :draggable="allowDraggable"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="tree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >Append</el-button>
          <el-button type="text" size="mini" @click="() => modify(data)">Modify</el-button>
          <el-button
            v-if="node.isLeaf"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >Delete</el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="closeOnClickModal"
      :show-close="false"
      :center="true"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" @click="sumbmitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      Cid: [],
      allowDraggable: false,
      updateNodes: [],
      maxLevel: 1,
      baseLevel: 0,
      closeOnClickModal: false,
      title: '',
      category: {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: '',
        productCount: 0
      },
      dialogVisible: false,
      menus: [],
      expandedIds: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  created() {
    this.getCategory()
  },
  methods: {
    batchUpdate() {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(() => {
        this.getCategory()
        this.expandedIds = this.Cid
        this.$message({
          type: 'success',
          message: `移动成功!`
        })
      })
      // console.log('updateNodes:', this.updateNodes)
      this.updateNodes = []
      this.Cid = []
    },
    batchDelete() {
      let CheckedNodes = this.$refs.tree.getCheckedNodes()
      console.log('CheckedNodes', CheckedNodes)
      let ids = []
      CheckedNodes.forEach(e => {
        ids.push(e.catId)
      })
      this.$confirm(
        `此操作将删除【${ids}】菜单, 是否继续?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.getCategory()
            this.expandedIds = ids
          })
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 移动成功后交换数据
      console.log('handleDrop: ', draggingNode, dropNode, dropType)

      // 改变三个属性，parent_cid， cat_level， sort
      // parent_cid
      let Cid = 0
      let siblings = null
      if (dropType === 'before' || dropType === 'after') {
        Cid = dropNode.data.parentCid
        siblings = dropNode.parent.childNodes
      } else {
        Cid = dropNode.data.catId
        siblings = dropNode.childNodes
      }

      this.Cid.push(Cid)

      // sort
      for (let i in siblings) {
        this.updateNodes.push({
          catId: siblings[i].data.catId,
          parentCid: Cid,
          sort: i,
          catLevel: siblings[i].level
        })
        if (siblings[i].data.catLevel != siblings[i].level) {
          this.updateChildNodesLevel(siblings[i])
        }
      }
    },
    updateChildNodesLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i in node.childNodes) {
          this.updateNodes.push({
            catId: node.childNodes[i].data.catId,
            catLevel: node.childNodes[i].level
          })
          this.updateChildNodesLevel(node.childNodes[i])
        }
      }
    },
    // 判断节点是否拖拽
    allowDrop(draggingNode, dropNode, type) {
      // console.log('allowDrop:', draggingNode, dropNode, type)
      // 记录基准节点
      this.baseLevel = draggingNode.level
      var deep = this.countLevel(draggingNode)
      // console.log('深度：', deep)
      if (type === 'inner') {
        return deep + dropNode.level <= 3
      } else {
        return deep + dropNode.parent.level <= 3
      }
    },
    //计算当前节点的最大深度
    countLevel(node) {
      var maxLevel = 1
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.baseLevel) {
            this.maxLevel = node.childNodes[i].level - this.baseLevel + 1
          }
          this.countLevel(node.childNodes[i])
        }
        return this.maxLevel
      } else {
        return maxLevel
      }
    },
    getCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        // 对象解构
        // console.log('success', data.data)
        this.menus = data.data
      })
    },
    addMenus() {
      if (this.category.name != '') {
        this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
        }).then(({ data }) => {
          console.log(data)
          this.getCategory()
          this.expandedIds = [this.category.parentCid]
          this.$message({
            type: 'success',
            message: `菜单添加成功!`
          })
        })
        this.dialogVisible = false
        console.log('add', this.category)
        this.category.name = ''
      } else {
        this.$message({
          type: 'info',
          message: '名称不能为空'
        })
      }
    },
    modifyMenus() {
      var { catId, name, icon, productUnit, parentCid } = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(() => {
        this.dialogVisible = false
        this.getCategory()
        this.expandedIds = [parentCid]
        this.$message({
          type: 'success',
          message: `菜单修改成功!`
        })
        this.category.catId = null
        this.category.name = ''
        this.category.icon = ''
        this.category.parentCid = 0
        this.category.productUnit = ''
      })
    },
    sumbmitData() {
      if (this.title === '添加分类') {
        this.addMenus()
      }
      if (this.title === '修改分类') {
        this.modifyMenus()
      }
    },
    cancel() {
      this.dialogVisible = false
      this.category.catId = null
      this.category.name = ''
      this.category.icon = ''
      this.category.parentCid = 0
      this.category.productUnit = ''
    },
    append(data) {
      this.dialogVisible = true
      this.title = '添加分类'
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      console.log('append', data)
    },
    modify(data) {
      this.dialogVisible = true
      this.title = '修改分类'
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        this.category.catId = data.data.catId
        this.category.name = data.data.name
        this.category.icon = data.data.icon
        this.category.parentCid = data.data.parentCid
        this.category.productUnit = data.data.productUnit
      })
      console.log('modify', data)
    },
    remove(node, data) {
      var ids = [data.catId]
      this.$confirm(
        `此操作将删除【${node.data.name}】菜单, 是否继续?`,
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      )
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.getCategory()
            this.expandedIds = [node.data.parentCid]
          })
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      console.log('remove', node, data)
    }
  }
}
</script>

<style>
</style>