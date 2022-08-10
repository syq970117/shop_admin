<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>

      <!-- 表格 -->
      <tree-table
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        border
        :show-row-hover="false"
        class="treeTable"
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0">1级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1"
            >2级</el-tag
          >
          <el-tag type="warning" v-else>3级</el-tag>
        </template>
        <template slot="opt">
          <el-button size="mini" type="primary" icon="el-icon-edit"
            >编辑</el-button
          >
          <el-button size="mini" type="danger" icon="el-icon-delete"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryinfo.pagenum"
        :page-sizes="[10, 20, 30, 40]"
        :page-size="queryinfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- options 指定数据源 -->
          <!-- props 指定配置对象 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 请求参数
      queryinfo: {
        type: 3,
        pagenum: 1,
        pagesize: 10,
      },
      // 商品分类的数据列表，默认为空
      catelist: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        {
          label: "是否有效",
          // 表示将当前列定义为模板列
          type: "template",
          // 表示当前列使用的模板名称
          template: "isok",
        },
        {
          label: "排序",
          // 表示将当前列定义为模板列
          type: "template",
          // 表示当前列使用的模板名称
          template: "order",
        },
        {
          label: "操作",
          // 表示将当前列定义为模板列
          type: "template",
          // 表示当前列使用的模板名称
          template: "opt",
        },
      ],
      addCateDialogVisible: false,
      addCateForm: {
        // 将要添加的分类名称
        cat_name: "",
        // 父级分类的id
        cat_pid: 0,
        // 分类等级，默认为1级分类
        cat_level: 0,
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
        ],
      },
      // 父级分类数据
      parentCateList: [],
      cascaderProps: {
        expandTrigger: "hover",
        value: "cat_id",
        label: "cat_name",
        children: "children",
        checkStrictly: true,
      },
      selectedKeys: [],
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryinfo,
      });

      if (res.meta.status !== 200) {
        return this.$methods.error("获取分类失败！");
      }

      this.catelist = res.data.result;
      this.total = res.data.total;
    },
    handleSizeChange(newSize) {
      this.queryinfo.pagesize = newSize;
      this.getCateList();
    },
    handleCurrentChange(newSize) {
      this.queryinfo.pagenum = newSize;
      this.getCateList();
    },
    showAddCateDialog() {
      this.getParentCateList();
      this.addCateDialogVisible = true;
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 },
      });

      if (res.meta.status !== 200) {
        return this.$message.error("获取分类数据列表失败！");
      }

      this.parentCateList = res.data;
    },
    parentCateChanged() {
      // 如果 selectedKeys 长度 > 0，说明父级id不为0
      if (this.selectedKeys.length > 0) {
        // 父级分类id
        this.addCateForm.cat_pid =
          this.selectedKeys[this.selectedKeys.length - 1];

        // 当前分类等级
        this.addCateForm.cat_level = this.selectedKeys.length;

        return;
      } else {
        this.addCateForm.cat_pid = 0;
        this.addCateForm.cat_level = 0;
      }
    },
    addCate() {
      this.$refs.addCateFormRef.validate(async (valid) => {
        if (!valid) {
          return;
        }

        const { data: res } = await this.$http.post(
          "categories",
          this.addCateForm
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加分类失败！");
        }

        this.$message.success("添加分类成功！");
        this.getCateList();
        this.addCateDialogVisible = false;
      });
      console.log(this.addCateForm);
    },
    // 监听对话框的关闭事件，重置输入框
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields();
      this.selectedKeys = [];
      this.addCateForm.cat_pid = 0;
      this.addCateForm.cat_level = 0;
    },
  },
};
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}

.el-cascader {
  width: 100%;
}
</style>
