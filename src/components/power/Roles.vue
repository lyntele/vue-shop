<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" stripe border>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1, i1) in scope.row.children" :key="item1.id"
            :class="['bdbottom', i1 === 0? 'bdtop' : '' , 'vcenter']">
              <!-- 渲染一级权限 -->
              <el-col :span="5" class="firstRight">
                <el-tag closable @close="removeRightsById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 二级权限 -->
                <el-row v-for="(item2, i2) in item1.children" :key="item2.id"
                :class="[i2=='0' ? '': 'bdtop' ,'vcenter']">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightsById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable @close="removeRightsById(scope.row, item3.id)">
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>

            <!-- <pre>{{scope.row}}</pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRolesById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色区域 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" @close="addDialogClosed">
      <el-form :model="addForm" ref="addRolesRef">
        <el-form-item label="角色名称">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 编辑角色区域 -->
    <el-dialog title="编辑角色" :visible.sync="editDialogVisible" @close="editDialogClosed">
      <el-form :model="editForm" ref="editFormRef">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoles">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 分配权限区域 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightClosed">
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all="true"
      :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //  角色列表
      rolesList: [],
      //  权限列表
      rightslist: [],
      //  控制添加角色表单的显示
      addDialogVisible: false,
      //  控制编辑角色表单的显示
      editDialogVisible: false,
      //  控制更改权限表单的显示
      setRightDialogVisible: false,
      //  添加角色的表单
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      //  编辑角色表单
      editForm: {
        roleName: '',
        roleDesc: ''
      },
      //  树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //  默认选定的节点id值数组
      defKeys: [],
      //  当前即将分配权限的角色Id
      roleId: ''
    }
  },
  created() {
    this.getRoleslist()
  },
  methods: {
    async getRoleslist() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败！')
      this.rolesList = res.data
      console.log(this.rolesList)
    },
    //  添加新角色
    addRoles() {
      //  点击确定按钮添加新角色
      //  调用validate表单验证
      this.$refs.addRolesRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写完整的角色信息')
        //  发送请求完成添加角色操作
        const { data: res } = await this.$http.post('roles', this.addForm)
        //  添加失败提示
        if (res.meta.status !== 201) return this.$message.error('添加角色失败！')
        //  添加成功提示
        this.$message.success('添加用户成功')
        //  关闭对话框
        this.addDialogVisible = false
        //  重新请求数据
        this.getRoleslist()
      })
    },
    //  添加用户退出时清除表单
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //  删除角色
    async removeRolesById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => {
        return err
      })

      //  若用户确认删除，返回值为字符串confirm
      //  若用户取消删除，返回字符串cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      //  确认删除
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }
      this.$message.success('删除用户成功！')
      this.getRoleslist()
    },
    //  编辑用户退出时清除表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //  展示对应角色的编辑框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取角色信息失败')
      console.log(res.data)
      this.editForm = res.data
      console.log('已取到用户信息')
      this.editDialogVisible = true
    },
    //  编辑角色
    async editRoles() {
      const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
        roleName: this.editForm.roleName,
        roleDesc: this.editForm.roleDesc
      })
      console.log(res.data)
      if (res.meta.status !== 200) return this.$message.error('更新角色失败！')
      this.editDialogVisible = false
      this.getRoleslist()
      this.$message.success('更新用户')
    },
    //  删除三级权限
    async removeRightsById(role, rightId) {
      //  弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消了删除')
      }
      console.log('确认了删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      role.children = res.data
    },
    //  展示编辑用户权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限列表失败')
      this.rightslist = res.data
      console.log('已经获取权限信息')
      //  递归获取三级结点的id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    //  用递归的形式，获取角色下所有三级权限的id并保存到defKeys数组中
    getLeafKeys(node, arr) {
      //  若当前node节点不包含children属性，则为三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      //  forEach函数便利整体item
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    //  在关闭编辑权限对话框后清空defKeys数组
    setRightClosed() {
      this.defKeys = []
    },
    //  点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      console.log(idStr)
      console.log(this.roleId)
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败！')
      this.$message.success('更新权限成功')
      this.getRoleslist()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
  .el-tag {
    margin: 7px;
  }
  .bdtop {
    border-top:1px solid #eee;
  }
  .bdbottom {
    border-bottom: 1px solid #eee;
  }
  .firstRight {
    text-align: center;
  }
  .vcenter {
    display:flex;
    align-items: center;
  }
</style>
