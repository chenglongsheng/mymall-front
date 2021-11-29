<template>
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
</template>

<script>
export default {
  data() {
    return {
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
    append(data) {
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
          }),
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