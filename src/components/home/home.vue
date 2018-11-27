<template>
  <el-container class="container">
    <el-header class="header">
      <el-col :span="4">
        <div class="grid-content bg-purple">
          <img src="../../assets/logo.png" alt>
        </div>
      </el-col>
      <el-col :span="19">
        <div class="grid-content bg-purple-light">
          <h3 class="middle">电商后台管理系统</h3>
        </div>
      </el-col>
      <el-col :span="1">
        <div class="grid-content bg-purple">
          <a href="#" class="loginout" @click.prevent="handleLoginOut()">退出</a>
        </div>
      </el-col>
    </el-header>
    <el-container>
      <el-aside width="200px" class="aside">
        <el-row class="tac">
          <el-col :span="24">
            <el-menu default-active="2" class="el-menu-vertical-demo">
              <el-submenu index="1">
                <template slot="title">
                  <i class="el-icon-location"></i>
                  <span>导航一</span>
                </template>
                <el-menu-item index="1-1-1">
                  <i class="el-icon-location"></i>选项1
                </el-menu-item>
                <el-menu-item index="1-1-1">
                  <i class="el-icon-location"></i>选项1
                </el-menu-item>
              </el-submenu>
              <el-submenu index="2">
                <template slot="title">
                  <i class="el-icon-location"></i>
                  <span>导航一</span>
                </template>
                <el-menu-item index="1-2-1">
                  <i class="el-icon-location"></i>选项1
                </el-menu-item>
                <el-menu-item index="1-2-1">
                  <i class="el-icon-location"></i>选项1
                </el-menu-item>
              </el-submenu>
              <el-menu-item index="3">
                <i class="el-icon-menu"></i>
                <span slot="title">导航二</span>
              </el-menu-item>
              <el-menu-item index="4">
                <i class="el-icon-document"></i>
                <span slot="title">导航三</span>
              </el-menu-item>
              <el-menu-item index="5">
                <i class="el-icon-setting"></i>
                <span slot="title">导航四</span>
              </el-menu-item>
            </el-menu>
          </el-col>
        </el-row>
      </el-aside>
      <el-main class="main">Main</el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  // 再组件渲染之前 if token
  // 钩子函数 在创建vm实例之前
  beforeCreate() {
    // 获取token
    const token = localStorage.getItem("token");
    // 如果不存在 回到login
    if (!token) {
      this.$router.push({ name: "login" });
    }
  },
  methods: {
    handleLoginOut() {
      // 1. 提示
      this.$message.success("退出成功");
      // 2. 清除
      localStorage.clear();
      // 3. 回到login组件
      this.$router.push({ name: "login" });
    }
  }
};
</script>

<style>
.container {
  height: 100%;
}
.header {
  background-color: #b3c0d1;
}
.aside {
  background-color: #d3dce6;
}
.main {
  background-color: #e9eef3;
  height: 100%;
}
/* 头部样式调整 */
.middle {
  text-align: center;
  color: #fff;
  font-size: 24px;
  margin: 0;
  line-height: 60px;
}
.loginout {
  line-height: 60px;
  text-decoration: none;
}
</style>
