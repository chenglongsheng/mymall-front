<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    @node-click="handleNodeClick"
  ></el-tree>
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
    handleNodeClick(data) {
      console.log(data)
    }
  }
}
</script>

<style>
</style>