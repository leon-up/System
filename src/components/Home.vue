<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <!-- 菜单区域 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">| | |</div>
        <el-menu background-color="#373d41" text-color="#fff" active-text-color=" #409eff" unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
        <!-- 一级菜单 -->
          <el-submenu :index="items.id + ''" v-for="items in menulist" :key="items.id">
            <!-- 一级菜单的模板区域 -->
            <template slot="title">
              <!-- 一级菜单图标区域 -->
              <i :class="iconsObj[items.id]"></i>
              <!-- 一级菜单文本区域 -->
              <span>{{items.authName}}</span>
            </template>

            
             <el-menu-item :index="'/' + subItem.path" v-for="subItem in items.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
               <template slot="title">
              <!-- 二级菜单图标区域 -->
              <i class="el-icon-menu"></i>
              <!-- 二级菜单文本区域 -->
              <span>{{subItem.authName}}</span>
            </template>
             </el-menu-item>
          </el-submenu>
          
        </el-menu>
      </el-aside>
      <!-- 主体区域 -->
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
      menulist: [],
      iconsObj: {
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
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      //清空 sessionStorage 中的 token
      window.sessionStorage.clear();
      //通过编程式导航重定向到登录页
      this.$router.push("/login");
    },
    async getMenuList() {
      const {data: res} = await this.$http.get('menus')
        //console.log(res)
        if(res.meta.status !== 200) return this.$message.error('获取菜单列表失败')
        this.menulist = res.data
    },
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  },
};
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 0;
  color: #fff;
  background-color: #373d41;

  > div {
    display: flex;
    align-items: center;

    span {
      padding-left: 15px;
      font-size: 18px;
    }
  }
}
.el-aside {
  background-color: #333744;

    .el-menu {
      border-right: 0;
    }
}
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 15px;
}
.toggle-button {
  background-color: #4a5064;
  line-height: 24px;
  text-align: center;
  color: #fff;
  font-size: 14px;
  cursor: pointer;
}
</style>