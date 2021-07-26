<template>
  <el-container>
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/logo.png" />
        <span>电商管理后台</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse? '60px' : '200px' ">
        <div class="toggle-button" @click="togleCollapse">|||</div>
        <el-menu unique-opened :collapse="isCollapse" router :collapse-transition="false" background-color="#333744" text-color="#fff" active-text-color="#409FFF" :default-active="activePath">
          <el-submenu :index="menu.id+''" v-for="menu in menuList" :key="menu.id">
            <template slot="title">
              <i :class="icons[menu.id]"></i>
              <span>{{menu.authName}}</span>
            </template>
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in menu.children" :key="subItem.id" @click="saveActivePath('/'+subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ subItem.authName}}</span>
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
  data () {
    return {
      menuList: [],
      icons: {
        '125': 'iconfont icon-user',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  created () {
    this.getMenus()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/Login')
    },
    async getMenus () {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status === 200) {
        this.menuList = res.data
      }
    },
    togleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    saveActivePath (activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>
<style lang="less" scoped>
.el-container {
  width: 100%;
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0px;
  align-items: center;
  color: #fff;
  font-size: 20px;

  div {
    display: flex;
    align-items: center;
    img {
      width: 90px;
      height: 90px;
    }
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #333744;

  .el-menu {
    border: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.toggle-button {
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  // 鼠标放上去变成小手
  cursor: pointer;
}
</style>
