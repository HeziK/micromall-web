<template>
  <div>
    <el-button type="danger" plain @click="batchDelete">批量删除</el-button>
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      ref="menuTree"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
            >新增</el-button
          >
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
            >删除</el-button
          >
          <el-button type="text" size="mini" @click="edit(data)"
            >修改</el-button
          >
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit"></el-input>
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
export default {
  data() {
    return {
      menus: [],
      title: "",
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: null,
        productUnit: null,
      },
      dialogVisible: false,
      dialogType: "",
      expandKey: [],
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
        //console.log("成功获取到菜单数据。。。", data.data),
          (this.menus = data.data);
      });
    },

    edit(data) {
      //console.log("要修改的数据：", data);
      this.dialogVisible = true;
      this.dialogType = "edit";
      this.title = "编辑分类菜单";

      //发送请求获取当前最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        //请求成功
        //console.log("获取的数据", data);
        this.category.parentCid = data.data.parentCid;
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.catLevel = data.data.catLevel;
        this.category.showStatus = data.data.showStatus;
        this.category.sort = data.data.sort;
      });
    },

    append(data) {
      //console.log("要添加的数据：", data);
      this.dialogVisible = true;
      this.dialogType = "add";
      this.title = "新增分类菜单";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;

      //修改完成后，弹出的新增对话框恢复原来状态
      this.category.name = "";
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.showStatus = 1;
      this.category.sort = 0;
    },

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】选项？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "删除成功 O.O",
              type: "success",
            });
            //刷新出新菜单
            this.getMenus();
            //设置刷新后默认自动展开的菜单
            this.expandKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
    },

    //添加菜单分类的方法
    addCategory() {
      //console.log("提交的数据：", this.category);
      //向后台发送保存数据请求
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        //保存成功弹出提示框
        this.$message({
          message: "保存成功 O.O",
          type: "success",
        });
        //保存后关闭对话框
        this.dialogVisible = false;
        //刷新出新菜单
        this.getMenus();
        //设置刷新后默认自动展开的菜单
        this.expandKey = [this.category.parentCid];
      });
    },

    //修改菜单分类的方法
    editCategory() {
      var { name, catId, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        //要发送的数据
        data: this.$http.adornData({ name, catId, icon, productUnit }, false),
      }).then(({ data }) => {
        this.$message({
          message: "修改成功 OvO",
          type: "success",
        });
        //保存后关闭对话框
        this.dialogVisible = false;
        //刷新出新菜单
        this.getMenus();
        //设置刷新后默认自动展开的菜单
        this.expandKey = [this.category.parentCid];
      });
    },

    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    //批量删除方法
    batchDelete(data) {
      let names = [];
      let catIds = [];
      let checkedNodes = this.$refs.menuTree.getCheckedNodes();
      //console.log("被选中的数据", checkedNodes);
      checkedNodes.forEach((element) => {
        names.push(element.name);
      });
      checkedNodes.forEach((element) => {
        catIds.push(element.catId);
      });
      this.$confirm(`是否批量删除【${names}】选项？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(() => {
            this.$message({
              message: "批量删除成功 OvO",
              type: "success",
            });
            this.getMenus();
          });
        })
        .catch(() => {});
    },
  },
  created() {
    this.getMenus();
  },
};
</script>

<style>
</style>