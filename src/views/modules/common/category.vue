<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    ref="menuTree"
    @node-click="nodeClick"
  >
  </el-tree>
</template>

<style>
</style>

<script>
export default {
  data() {
    return {
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("菜單數據：", data.data);
        this.menus = data.data;
      });
    },
    nodeClick(data, node, component) {
        console.log("子組件category的節點被點擊", data, node, component)
        //向父組件發送事件
        this.$emit("tree-node-click", data, node, component);
    },
  },
  created() {
    this.getMenus();
  },
};
</script>
