<template>
  <div>
    <el-tree
      node-key="catId"
      :data="menus"
      :props="defaultProps"
      show-checkbox
      :expand-on-click-node="false"
      :default-expanded-keys="expandedIds"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.isLeaf"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span></el-tree
    >
    <el-dialog title="添加菜单" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addMenus">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0
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
    append(data) {
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      console.log('append', data)
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