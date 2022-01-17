<template>
  <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
  data() {
    return {
      menus: [],
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
    handleNodeClick(data, node, component) {
      // console.log('son-data', data, node, component)

      // 将子组件事件的数据传递给父组件
      this.$emit('handle-node-click', data, node, component)
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
    }
  }
}
</script>

<style>
</style>