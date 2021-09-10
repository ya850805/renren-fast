<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
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
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分類名稱">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="圖標">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="計量單位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
/* eslint-disable */
export default {
  data() {
    return {
      menus: [],
      expandedKey: [],
      dialogVisible: false,
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
      dialogType: "", //add, edit
      title: "",
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    // handleNodeClick(data) {
    //   console.log(data);
    // },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("菜單數據：", data.data);
        this.menus = data.data;
      });
    },
    append(data) {
      this.dialogVisible = true;
      this.dialogType = "add";
      this.title = "添加分類";
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
    },

    remove(node, data) {
      console.log("remove", data);
      //刪除確定提示
      this.$confirm(`是否刪除【${data.name}】菜單?`, "提示", {
        confirmButtonText: "確定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          const ids = [data.catId]; // 設定為一個陣列
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            //刪除成功提示
            this.$message({
              message: "菜單刪除成功！",
              type: "success",
            });
            //重新發請求，刷新菜單數據
            this.getMenus();
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },
    addCategory() {
      //添加三級分類
      console.log(this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜單保存成功！",
          type: "success",
        });
        //關閉對話框
        this.dialogVisible = false;
        //刷新數據
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
    edit(data) {
      console.log("要修改的：", data);
      this.dialogType = "edit";
      this.title = "修改分類";
      this.dialogVisible = true;

      //發送請求獲取當前節點最新的數據
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
        data: this.$http.adornParams(this.category, false),
      }).then(({ data }) => {
        console.log(data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
    },
    //修改三級分類數據
    editCategory() {
      const { catId, name, icon, productUnit } = this.category;
      const updateData = { catId, name, icon, productUnit };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(updateData, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜單修改成功！",
          type: "success",
        });
        //關閉對話框
        this.dialogVisible = false;
        //刷新數據
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },
  },
  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>