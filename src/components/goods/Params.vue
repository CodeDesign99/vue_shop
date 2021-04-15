<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 权限列表卡片 -->
    <el-card>
      <!-- 提示 -->
      <!-- <div class="tips">
        <i class="el-icon-warning"> 注意：只允许为第三级分类设置相关参数</i>
      </div> -->
      <el-alert
        title="注意：只允许为第三级分类设置相关参数"
        type="warning"
        :closable="false"
        show-icon
      ></el-alert>

      <!-- 选择商品分类 -->
      <!-- <div class="select">
        <p class="text">选择商品分类：</p>
        <el-cascader
          v-model="selectedKeys"
          :options="parentCateList"
          :props="cascaderProps"
          @change="parentCateChanged"
          clearable
        ></el-cascader>
      </div> -->

      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab 页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数的面板 -->
        <el-tab-pane label="动态参数" name="many">
          <template>
            <el-button type="primary" :disabled="available" @click="addManyData"
              >添加参数</el-button
            >
            <!-- 参数列表 -->
            <el-table :data="ParamsList" border stripe>
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <!-- 循环渲染tag标签 -->
                  <el-tag
                    v-for="(item, i) in scope.row.attr_vals"
                    :key="i"
                    closable
                    @close="handleClose(i, scope.row)"
                    >{{ item }}</el-tag
                  >
                  <!-- 输入文本框 -->
                  <el-input
                    class="input-new-tag"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    ref="saveTagInput"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope.row)"
                    @blur="handleInputConfirm(scope.row)"
                  >
                  </el-input>
                  <!-- 添加按钮 -->
                  <el-button
                    v-else
                    class="button-new-tag"
                    size="small"
                    @click="showInput(scope.row)"
                    >+ New Tag</el-button
                  >
                </template>
              </el-table-column>
              <el-table-column type="index" label="序号" width="60px">
              </el-table-column>
              <el-table-column label="参数名称" prop="attr_name">
              </el-table-column>
              <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                  <el-button
                    type="primary"
                    icon="el-icon-edit"
                    size="mini"
                    @click="editParam(scope.row.attr_id)"
                    >编辑</el-button
                  >
                  <el-button
                    type="danger"
                    icon="el-icon-delete"
                    size="mini"
                    @click="removeParmeById(scope.row.attr_id)"
                    >删除</el-button
                  >
                </template>
              </el-table-column>
            </el-table>
            <!-- 添加动态参数的对话框 -->
            <el-dialog
              title="添加动态参数"
              :visible.sync="manyDialogVisible"
              width="50%"
              @close="addDialogClosed"
            >
              <el-form
                :model="addForm"
                :rules="FormRules"
                ref="addFormRef"
                label-width="100px"
              >
                <el-form-item label="动态参数" prop="attr_name">
                  <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
              </el-form>
              <span slot="footer" class="dialog-footer">
                <el-button @click="manyDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParam">确 定</el-button>
              </span>
            </el-dialog>
          </template>
        </el-tab-pane>
        <!-- 静态属性的面板 -->
        <el-tab-pane label="静态属性" name="only">
          <template>
            <el-button type="primary" :disabled="available" @click="addOnlyData"
              >添加属性</el-button
            >
            <!-- 属性列表 -->
            <el-table :data="ParamsList" border stripe>
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag
                    v-for="(item, i) in scope.row.attr_vals"
                    :key="i"
                    closable
                    @close="handleClose(i, scope.row)"
                    >{{ item }}</el-tag
                  >
                  <!-- 输入文本框 -->
                  <el-input
                    class="input-new-tag"
                    v-if="scope.row.inputVisible"
                    v-model="scope.row.inputValue"
                    ref="saveTagInput"
                    size="small"
                    @keyup.enter.native="handleInputConfirm(scope.row)"
                    @blur="handleInputConfirm(scope.row)"
                  >
                  </el-input>
                  <!-- 添加按钮 -->
                  <el-button
                    v-else
                    class="button-new-tag"
                    size="small"
                    @click="showInput(scope.row)"
                    >+ New Tag</el-button
                  >
                </template>
              </el-table-column>
              <el-table-column type="index" label="序号" width="60px">
              </el-table-column>
              <el-table-column label="属性名称" prop="attr_name">
              </el-table-column>
              <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                  <el-button
                    type="primary"
                    icon="el-icon-edit"
                    size="mini"
                    @click="editParam(scope.row.attr_id)"
                    >编辑</el-button
                  >
                  <el-button
                    type="danger"
                    icon="el-icon-delete"
                    size="mini"
                    @click="removeParmeById(scope.row.attr_id)"
                    >删除</el-button
                  >
                </template>
              </el-table-column>
            </el-table>
            <!-- 添加静态参数对话框 -->
            <el-dialog
              title="添加静态属性"
              :visible.sync="onlyDialogVisible"
              width="50%"
              @close="addDialogClosed"
            >
              <el-form
                :model="addForm"
                :rules="FormRules"
                ref="addFormRef"
                label-width="100px"
              >
                <el-form-item label="静态属性" prop="attr_name">
                  <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
              </el-form>
              <span slot="footer" class="dialog-footer">
                <el-button @click="onlyDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParam">确 定</el-button>
              </span>
            </el-dialog>
          </template>
        </el-tab-pane>
      </el-tabs>
      <!-- 修改参数 -->
      <el-dialog
        :title="'修改' + titleText"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClosed"
      >
        <el-form
          :model="editForm"
          :rules="FormRules"
          ref="editFormRef"
          label-width="100px"
        >
          <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="editForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editParamData">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedKeys: [],
      parentCateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectCateForm: {
        cat_id: 0
      },
      //被激活的页签名称
      activeName: 'many',
      available: true,
      ParamsList: [],
      manyDialogVisible: false,
      onlyDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      FormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }
        ]
      },
      editForm: {}
    }
  },
  created() {
    this.getParentCateList()
  },
  methods: {
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200)
        return this.$message.error('获取父级分类数据失败')
      this.parentCateList = res.data
    },
    parentCateChanged() {
      // 获得分类Id
      this.selectCateForm.cat_id = this.selectedKeys[
        this.selectedKeys.length - 1
      ]
      this.handleTabClick()
      if (this.selectedKeys.length > 0) {
        this.available = false
      } else {
        this.available = true
        this.ParamsList = []
      }
    },
    async handleTabClick() {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        return
      }
      if (this.selectedKeys.length > 0) {
        const { data: res } = await this.$http.get(
          'categories/' + this.selectCateForm.cat_id + '/attributes',
          {
            params: { sel: this.activeName }
          }
        )
        if (res.meta.status !== 200) return this.$message.error('获取参数失败')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
          // 控制文本框的显示与隐藏
          item.inputVisible = false
          // 文本框中输入的值
          item.inputValue = ''
        })
        this.ParamsList = res.data
      }
    },
    addManyData() {
      this.manyDialogVisible = true
    },
    addOnlyData() {
      this.onlyDialogVisible = true
    },
    addParam() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories/' + this.selectCateForm.cat_id + '/attributes',
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) return this.$message.error('添加参数失败')
        this.$message.success('添加参数成功')
        this.handleTabClick()
        if (this.activeName === 'many') {
          this.manyDialogVisible = false
        } else {
          this.onlyDialogVisible = false
        }
      })
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    async editParam(id) {
      const { data: res } = await this.$http.get(
        `categories/${this.selectCateForm.cat_id}/attributes/${id}`,
        {
          params: { attr_sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) return this.$message.error('查询失败')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editParamData() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.editForm.cat_id}/attributes/${this.editForm.attr_id}`,
          {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) return this.$message.error('修改参数失败')
        this.$message.success('修改参数成功')
        this.editDialogVisible = false
        this.handleTabClick()
      })
    },
    async removeParmeById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
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
      const { data: res } = await this.$http.delete(
        `categories/${this.selectCateForm.cat_id}/attributes/${id}`
      )
      if (res.meta.status !== 200) return this.$message.error('删除参数失败')
      this.$message.success('删除参数成功')
      this.handleTabClick()
    },
    showInput(row) {
      row.inputVisible = true
      // $nextTick 方法的作用是当页面上的元素重新渲染之后，才会执行回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputVisible = false
        row.inputValue = ''
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputVisible = false
      row.inputValue = ''
      this.saveAttrVals(row)
    },
    // 将对attr_vals的操作保存到数据库
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.selectCateForm.cat_id}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) return this.$message.error('修改参数项失败')
    },
    // 删除对应的参数可选项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  },
  components: {}
}
</script>

<style scoped>
/* .tips {
  height: 40px;
  background-color: #fcf6ea;
  color: #efa139;
  font-style: normal;
  line-height: 40px;
  padding-left: 15px;
  border-radius: 2px;
} */

/* .text {
  height: 40px;
  margin: 20px 0 0 0;
  float: left;
  line-height: 40px;
}
.el-cascader {
  margin-top: 15px;
} */
.cat_opt {
  margin: 15px 0;
}
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
