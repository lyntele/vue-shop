<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="7">
          <!-- 搜索与添加区 -->
          <el-input placeholder="请输入内容"
          v-model="queryInfo.query"
          clearable @clear="getUserlist">
            <el-button slot="append"
            icon="el-icon-search"
            @click="getUserlist"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区 -->
      <el-table :data="userlist" border stripe>
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <!-- slot-scope这种写法可以取到当前单元格，scope.row直接取到单元格对象 -->
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope" width="180px">
            <!-- 修改按钮 -->
            <el-tooltip effect="dark" content="修改" placement="top">
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            </el-tooltip>
            <!-- 删除按钮 -->
            <el-tooltip effect="dark" content="删除" placement="top">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
            </el-tooltip>
            <!-- 分配角色按钮 -->
            <el-tooltip effect="dark" content="分配角色" placement="top">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
  <!-- 添加用户的对话框 -->
    <el-dialog
    title="添加用户"
    :visible.sync="addDialogVisible"
    width="30%"
    @close="addDialogClosed">
    <!-- 内容主体区 -->
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="addForm.username"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password">
        <el-input v-model="addForm.password"></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="addForm.email"></el-input>
      </el-form-item>
      <el-form-item label="手机" prop="mobile">
        <el-input v-model="addForm.mobile"></el-input>
      </el-form-item>
    </el-form>
      <!-- 底部区域 -->
    <span slot="footer" class="dialog-footer">
      <el-button @click="addDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
</el-dialog>
  <!-- 编辑用户的对话框 -->
  <el-dialog
  title="编辑用户"
  :visible.sync="editDialogVisible"
  width="50%"
  :before-close="handleClose" @close="editDialogClosed">
  <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
    <el-form-item label="用户名">
      <el-input v-model="editForm.username" disabled></el-input>
    </el-form-item>
    <el-form-item label="手机" prop="mobile">
      <el-input v-model="editForm.mobile"></el-input>
    </el-form-item>
    <el-form-item label="邮箱" prop="email">
      <el-input v-model="editForm.email"></el-input>
    </el-form-item>
  </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
</el-dialog>
  </div>
</template>

<script>
export default {
  data() {
  //  验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        return cb()
      }
      //  返回一个错误提示
      cb(new Error('请输入合法的邮箱'))
    }
    //  验证手机号码的规则
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return cb()
      }
      //  返回一个错误提示
      cb(new Error('请输入合法的手机号码'))
    }
    return {
      //  获取用户列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      userlist: [],
      total: 0,
      //  控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      //  控制编辑用户对话框的显示与隐藏
      editDialogVisible: false,
      //  查询到的用户信息对象
      editForm: {},
      //  添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      //  添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 5, max: 16, message: '长度在 5 到 16 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      //  编辑用户表单验证对象
      editFormRules: {
        email: [
          { required: true, message: '邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getUserlist()
  },
  methods: {
    async getUserlist() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('用户列表数据获取失败!')
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(res)
    },
    //  监听pagesize改变事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      console.log(newSize)
      this.getUserlist()
    },
    //  监听handleCurrentChange改变事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      console.log(newPage)
      this.getUserlist()
    },
    //  添加用户退出时清除表单
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //  添加用户表单预验证
    addUser() {
      //  点击确定按钮，添加新用户
      //  调用validate进行表单验证
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写完整用户信息')
        //  发送请求完成添加用户的操作
        const { data: res } = await this.$http.post('users', this.addForm)
        console.log(res.meta.status)
        //  判断如果添加失败，就做提示
        if (res.meta.status !== 201) return this.$message.error('添加用户失败')
        //  添加成功的提示
        this.$message.success('添加用户成功')
        //  关闭对话框
        this.addDialogVisible = false
        //  重新请求最新的数据
        this.getUserlist()
      })
    },
    //  展示修改用户的对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('users/' + id)
      console.log(id)
      if (res.meta.status !== 200) return this.$message.error('获取用户信息失败！')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    //  编辑用户退出时清理表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //  修改用户信息并提交
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return false
        //  发起修改用户信息的请求
        const { data: res } = await this.$http.put('users/' + this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败！')
        }
        //  关闭对话框
        this.editDialogVisble = false
        //  刷新数据列表
        this.getUserlist()
        //  提示修改成功
        this.$message.success('更新用户')
      })
    },
    //  删除用户
    async removeUserById(id) {
      console.log(id)
      //  询问用户是否删除数据
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })

      //  如果用户确认删除，返回值为字符串confirm
      //  如果用户取消删除，则返回值为字符串cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }

      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }
      this.$message.success('删除用户成功！')
      this.getUserlist()
    }
  }
}
</script>

<style lang="less" scoped>

</style>
