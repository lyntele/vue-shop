<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert closable title="注意！只允许为第三级分类设置相关参数" type="warning" show-icon></el-alert>
      <!-- 选择商品分类区域 -->
      <div class="cat_top">选择商品分类：
        <el-cascader
          v-model="selectedCatKeys"
          :options="catelist"
          :props="cateProps"
          @change="handleChange">
        </el-cascader>
      </div>
      <!-- 分页区域 -->
       <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled='this.isChecked' @click="showAddDialog">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="this.manyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)">
                  {{item}}
                </el-tag>
                <!-- 动态添加tag -->
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
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" @click="showeditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" type="danger">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled='this.isChecked' @click="showAddDialog">添加属性</el-button>
          <!-- 静态参数表格 -->
          <el-table :data="this.onlyTableData" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- tag标签 -->
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)" >
                  {{item}}
                </el-tag>
                <!-- 动态添加tag -->
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
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引列 -->
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" @click="showeditDialog(scope.row.attr_id)">编辑</el-button>
                <el-button size="mini" type="danger" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
      <!-- 添加参数对话框 -->
      <el-dialog :title="'添加' + this.titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
        <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
          <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="addForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 修改参数对话框 -->
      <el-dialog :title="'修改' + this.titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
          <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="editForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //  商品分类列表
      catelist: [],
      //  级联选择框的配置对象
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //  级联选择框双向绑定的数组
      selectedCatKeys: [],
      //  tab默认激活的标签页
      activeName: 'many',
      //  添加参数和添加属性按钮是否可选用
      isChecked: true,
      //  动态表格数据
      manyTableData: [],
      //  静态表格数据
      onlyTableData: [],
      //  添加对话框的显示与隐藏
      addDialogVisible: false,
      //  这是添加参数的表单数据对象
      addForm: {
        attr_name: ''
      },
      //  添加表单数据的验证对象
      addFormRules: {
        attr_name: { required: true, message: '请输入参数名称', trigger: 'blur' }
      },
      //  修改参数对话框的显示与隐藏
      editDialogVisible: false,
      //  这是修改参数的表单数据对象
      editForm: {
        attr_name: ''
      },
      //  添加表单数据的验证对象
      editFormRules: {
        attr_name: { required: true, message: '请输入参数名称', trigger: 'blur' }
      },
      //  控制按钮与文本框的切换
      inputVisible: false,
      //  文本框中输入的内容
      inputValue: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //  获取所有商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    //  级联选择框中的项变化会触发这个数组
    handleChange() {
      this.getParamsData()
    },
    //  Tab点击事件的处理函数
    handleTabClick() {
      console.log(this.activeName)
      this.getParamsData()
    },
    //  获取商品分类数据
    async getParamsData() {
      //  根据所选分类的Id和所取参数的面板来获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取参数列表失败')
      this.isChecked = false
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
        //  控制文本框的显示与隐藏
        item.inputVisible = false
        //  文本框中输入的值
        item.inputValue = ''
      })
      console.log(res.data)
      if (this.activeName === 'many') this.manyTableData = res.data
      else this.onlyTableData = res.data
    },
    //  显示添加动态属性/静态参数的对话框
    showAddDialog() {
      this.addDialogVisible = true
      console.log(this.textTitle)
    },
    //  监听对话框关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //  添加分类
    async addParams() {
      const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
        attr_name: this.addForm.attr_name,
        attr_sel: this.activeName
      })
      if (res.meta.status !== 201) return this.$message.error('添加参数失败')
      this.getParamsData()
      this.addDialogVisible = false
      return this.$message.success('添加成功')
    },
    //  显示修改动态属性/静态参数的对话框
    async showeditDialog(attrId) {
      this.editDialogVisible = true
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attrId}`, {
        params: { attr_sel: this.activeName }
      })
      if (res.meta.status !== 200) return this.$message.error('获取参数信息失败')
      this.editForm = res.data
    },
    //  监听修改对话框关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //  修改参数
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 200) return this.$message.error('修改参数失败')
        this.getParamsData()
      })
      this.$message.success('修改成功')
      this.editDialogVisible = false
    },
    //  删除参数
    async removeParams(attrId) {
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrId}`)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getParamsData()
    },
    //  点击按钮展示文本输入框
    showInput(row) {
      row.inputVisible = true
      //  让文本框自动获得焦点
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    //  失去焦点时触发
    async handleInputConfirm(row) {
      //  如果输入的都是空格，则非法，直接return
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return false
      }
      //  没问题的话进行后续tag添加
      row.attr_vals.push(row.inputValue.trim())
      row.inputVisible = false
      row.inputValue = ''
      this.saveAttrVals(row)
    },
    //  保存对attr_vals的操作
    async saveAttrVals(row) {
      //  需要发起请求保存这一次操作
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(',')
      })
      if (res.meta.status !== 200) return this.$message.error('修改参数项失败！')
      this.$message.success('修改参数项成功！')
    },
    //  删除对应的参数和选项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    cateId() {
      return this.selectedCatKeys[2]
    },
    titleText() {
      if (this.activeName === 'many') return '动态参数'
      else return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
  .cat_top {
    margin-top: 10px;
  }
  .el-tag {
    margin: 10px;
  }
  .input-new-tag {
    width: 120px;
  }
</style>
