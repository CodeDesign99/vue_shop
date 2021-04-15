<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 权限列表卡片 -->
    <el-card>
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      <!-- 用户列表 -->
      <!-- <el-table :data="CategoryList" border="">
        <el-table-column type="index"></el-table-column>
        <el-table-column label="分类名称" prop="cat_name"></el-table-column>
        <el-table-column label="是否有效" prop="cat_deleted">
          <template slot-scope="scope">
            <i v-if="scope.row.cat_deleted == true" class="el-icon-success"></i>
            <i v-else class="el-icon-error"></i>
          </template>
        </el-table-column>
        <el-table-column label="排序" prop="cat_level">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag v-else-if="scope.row.cat_level === 1" type="success"
              >二级</el-tag
            >
            <el-tag v-else type="warning">三级</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="300px">
          <template>
            <el-button size="mini" class="el-icon-edit" type="primary"
              >编辑</el-button
            >
            <el-button size="mini" class="el-icon-delete" type="danger"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table> -->

      <!-- 表格 -->
      <tree-table
        :data="CategoryList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text=""
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="deleted" slot-scope="scope">
          <i
            style="color:#F56C6C"
            v-if="scope.row.cat_deleted == true"
            class="el-icon-error"
          ></i>
          <i style="color:#67C23A" v-else class="el-icon-success"></i>
        </template>
        <!-- 排序 -->
        <template slot="level" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag
            v-else-if="scope.row.cat_level === 1"
            type="success"
            size="mini"
            >二级</el-tag
          >
          <el-tag v-else type="warning" size="mini">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="operation" slot-scope="scope">
          <el-button
            size="mini"
            class="el-icon-edit"
            type="primary"
            @click="showEditCateDialog(scope.row.cat_id)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            class="el-icon-delete"
            type="danger"
            @click="removeCateById(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="CategoryInfo.pagenum"
        :page-sizes="[2, 3, 5, 10]"
        :page-size="CategoryInfo.pagesize"
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
        ref="addFormRef"
        :rules="addCateFormRules"
        :model="addCateForm"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改分类对话框 -->
    <el-dialog
      title="修改分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
    >
      <el-form
        ref="aditFormRef"
        :rules="editCateFormRules"
        :model="editCateForm"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //商品分类的数据列表
      CategoryList: [],
      CategoryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      //   为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'deleted'
        },
        {
          label: '排序',
          type: 'template',
          template: 'level'
        },
        {
          label: '操作',
          type: 'template',
          template: 'operation'
        }
      ],
      addCateDialogVisible: false,
      editCateDialogVisible: false,
      addCateForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      editCateForm: {},
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      editCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      selectedKeys: []
    }
  },
  created() {
    this.getCategoryList()
  },
  methods: {
    async getCategoryList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.CategoryInfo
      })
      if (res.meta.status !== 200)
        return this.$message.error('获取商品分类列表失败')
      this.CategoryList = res.data.result
      this.total = res.data.total
      //   console.log(res.data)
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
      //   console.log(newSize)
      this.CategoryInfo.pagesize = newSize
      this.getCategoryList()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange(newPage) {
      //   console.log(newPage)
      this.CategoryInfo.pagenum = newPage
      this.getCategoryList()
    },
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200)
        return this.$message.error('获取父级分类数据失败')
      this.parentCateList = res.data
      //   console.log(this.parentCateList)
    },
    addCateDialogClosed() {
      this.$refs.addFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    addCate() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) return this.$message.error('添加分类失败')
        this.$message.success('添加分类成功')

        this.getCategoryList()
        // console.log(this.addCateForm)
        this.addCateDialogVisible = false
      })
    },
    parentCateChanged() {
      // console.log(this.selectedKeys)
      // 如果 selectedKeys 数组中的length大于0，证明选择的父级分类
      // 反之，就是说没有选择任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类Id
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        // 父级分类Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
        return
      }
    },
    //修改编辑分类按钮事件
    async showEditCateDialog(id) {
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error('查询分类失败')
      this.editCateForm = res.data
      this.editCateDialogVisible = true
    },
    //修改编辑确认按钮事件
    async editCate() {
      const { data: res } = await this.$http.put(
        'categories/' + this.editCateForm.cat_id,
        {
          cat_name: this.editCateForm.cat_name
        }
      )
      if (res.meta.status !== 200) return this.$message.error('更新分类失败')
      this.$message.success('更新分类成功')
      this.getCategoryList()
      this.editCateDialogVisible = false
    },
    async removeCateById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除分类失败')
      this.$message.success('删除分类成功')
      this.getCategoryList()
    }
  },
  components: {}
}
</script>

<style scoped>
.zk-table {
  margin-top: 20px;
}
.el-cascader {
  width: 100%;
}
</style>
