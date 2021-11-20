<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/tg.jpg" alt="老婆" class="tg">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
      <el-container>
        <!-- 侧边栏 -->
        <el-aside :width="iscollapse ? '64px' : '200px'">
          <!-- 侧边栏伸缩按钮 -->
          <div class="toggle-button" @click="toggleCollapse">|||</div>
          <!-- 侧边栏菜单 -->
          <el-menu
      background-color="#333744"
      text-color="#fff"
      active-text-color="#409EFF"
      unique-opened="ture"
      :collapse="iscollapse"
      :collapse-transition="false"
      router
      :default-active="activePath">
      <!-- 一级菜单 -->
      <!-- index后必须绑定字符串，故在item.id后加一个'' -->
      <el-submenu :index="item.id + ''"  v-for="item in menulist" :key="item.id">
        <!-- 一级菜单模板区 -->
        <template slot="title">
          <!-- 图标 -->
          <i :class="iconsObj[item.id]"></i>
          <!-- 文本 -->
          <span>{{item.authName}}</span>
        </template>
        <!-- 二级菜单 -->
        <el-menu-item :index="'/' + subItem.path"
        v-for="subItem in item.children"
        :key="subItem.id"
        @click="saveNavState('/' + subItem.path)">
          <template slot="title">
          <!-- 图标 -->
          <i class="el-icon-menu"></i>
          <!-- 文本 -->
          <span>{{subItem.authName}}</span>
        </template>
        </el-menu-item>
      </el-submenu>
    </el-menu>
        </el-aside>
        <!-- 右侧内容主体 -->
        <el-main>
          <router-view></router-view>
        </el-main>
    </el-container>
</el-container>
</template>

<script>
export default {
  data() {
    return {
      //  左侧菜单数据
      menulist: [],
      iconsObj: {
        125: 'el-icon-user-solid',
        103: 'el-icon-s-management',
        101: 'el-icon-s-goods',
        102: 'el-icon-s-order',
        145: 'el-icon-s-data'
      },
      //  是否折叠
      iscollapse: false,
      //  激活的链接地址
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    //  获取所有菜单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
      console.log(res)
    },
    //  侧边栏折叠与展开
    toggleCollapse() {
      this.iscollapse = !this.iscollapse
    },
    //  保存链接激活状态
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
  .home-container {
    height: 100%
  }

  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0px;
    align-items: center;
    color: #fff;
    font-size: 20px;
    > div {
      display: flex;
      align-items: center;
      span {
        margin-left: 15px;
      }
    }
  }

  .el-aside {
    background-color: #333744;
  }

  .el-main {
    background-color: #eaedf1;
  }

  .tg {
    width: 57px;
    height: 57px;
    border-radius: 50px;
  }

  .el-menu {
    border-right: 0px;
  }

  .toggle-button {
    background-color: #4A5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.2em;
    // 光标移上去以后变成小手
    cursor: pointer;
  }

</style>
