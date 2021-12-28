<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 添加分类区域 -->
      <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" @close="addCateDialogClosed">
        <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
          <el-form-item label="分类名称：" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级名称：">
            <div class="block">
              <!-- options用来指定数据源 -->
              <el-cascader
                v-model="selectedKeys"
                :options="parentCateList"
                :props="cascaderprops"
                clearable></el-cascader>
            </div>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </div>
      </el-dialog>
      <!-- 表格 -->
      <tree-table :data="catelist" :columns="columns" :selection-type="false"
      :expand-type="false" show-index index-text="#" border class="treeTable">
        <!-- 是否有效 -->
        <template slot="isok"  slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false"
          style="color: lightgreen"></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order"  slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="100"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //  添加分类对话框是否可见
      addCateDialogVisible: false,
      //  查询条件
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //  商品分类的数据列表
      catelist: [],
      //  总数据条数
      total: 0,
      //  定义表格的列
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          //  表示将当前列定义为模板列
          type: 'template',
          //  表示当前这一列使用模板的名称
          template: 'isok'
        },
        {
          label: '排序',
          //  表示将当前列定义为模板列
          type: 'template',
          //  表示当前这一列使用模板的名称
          template: 'order'
        },
        {
          label: '操作',
          //  表示将当前列定义为模板列
          type: 'template',
          //  表示当前这一列使用模板的名称
          template: 'opt'
        }
      ],
      //  添加分类的表单数据对象
      addCateForm: {
        //  分类名称
        cat_name: '',
        //  分类父 ID
        cat_pid: 0,
        //  分类层级
        cat_level: 0
      },
      //  添加分类表单的验证规则对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      //  父级分类的列表
      parentCateList: [],
      //  指定级联选择器的配置对象
      cascaderprops: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //  储存级联选择器中的分类
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //  获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories',
        { params: this.querInfo })
      if (res.meta.status !== 200) return this.$message.error('请求商品分类数据失败')
      console.log(res.data)
      this.catelist = res.data.result
      this.total = res.data.total
    },
    //  监听pagesize改变事件
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    //  监听pagenum的改变
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    //  显示或隐藏添加分类对话框
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    //  获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories',
        { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类失败')
      console.log(res.data)
      this.parentCateList = res.data
    },
    //  监听添加分类对话框的关闭
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    //  添加分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }

        this.$message.success('添加分类成功')
        this.getCateList()
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }
</style>
