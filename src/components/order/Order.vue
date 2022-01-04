<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索区域 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入搜索内容">
            <!-- el-button组件的append属性值代表icon紧贴在依赖组件的后面 -->
            <el-button slot="append" icon="el-icon-search">搜索</el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单表格区域 -->
      <el-table stripe border :data="orderList">
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.pay_status === 1">已付款</el-tag>
            <el-tag type="danger" v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button size="mini" type="danger" icon="el-icon-edit" @click="showBox"></el-button>
            <el-button size="mini" type="success" icon="el-icon-location" @click="showProgressBox"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.query"
        :page-sizes="[10, 15, 20, 25]"
        :page-size="10"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
      <!-- 修改地址对话框 -->
      <el-dialog
        title="修改地址"
        :visible.sync="addressVisible"
        width="50%"
        @close="addressClosed">
        <el-form :model="addressForm" :rules="addressRules" ref="addressFormRef" label-width="100px">
          <el-form-item label="省市区县" prop="address1">
            <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addressVisible = false">取 消</el-button>
          <el-button type="primary" @click="editAddress">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 物流进度对话框 -->
      <el-dialog
        title="查看物流进度"
        :visible.sync="progressVisible"
        width="50%">
        <span>功能维护</span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from './citydata'
export default {
  data() {
    return {
      //  订单列表数据请求参数
      queryInfo: {
        query: 0,
        pagenum: 1,
        pagesize: 10
      },
      //  订单列表
      orderList: [],
      //  总数据条数
      total: 0,
      //  编辑地址表单
      addressForm: {
        address1: [],
        address2: ''
      },
      //  编辑地址对话框显示与隐藏
      addressVisible: false,
      //  编辑地址表单验证对象
      addressRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      //  级联选择器绑定对象
      cityData,
      progressVisible: false
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      // if (res.data.status !== 200) {
      //   return this.$message.error('获取订单信息列表失败')
      // }
      this.$message.success('获取订单信息成功')
      this.orderList = res.data.goods
      this.total = res.data.total
      console.log(this.orderList)
    },
    handleSizeChange(NewSize) {
      this.queryInfo.pagesize = NewSize
      this.getOrderList()
    },
    handleCurrentChange(NewPage) {
      this.queryInfo.pagenum = NewPage
      this.getOrderList()
    },
    //  显示编辑地址的对话框
    showBox() {
      this.addressVisible = true
    },
    addressClosed() {
      this.$refs.addressFormRef.resetFields()
    },
    //  展示物流进度对话框
    showProgressBox() {
      this.progressVisible = true
      //  因api故障无法实现该功能
    },
    //  修改地址
    editAddress() {
      this.addressVisible = true
    }
  }
}
</script>

<style lang="less" scoped>
  .el-input {
    width: 400px;
  }
</style>
