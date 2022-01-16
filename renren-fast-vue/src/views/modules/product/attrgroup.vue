<template>
  <el-row :gutter="20">
    <el-col :span="6">
      <category></category>
    </el-col>
    <el-col :span="18">asdada</el-col>
  </el-row>
</template>

<script>
import Category from '../common/category.vue'
export default {
  components: { Category },
  data() {
    return {}
  },
  created() {},
  methods: {}
}
</script>