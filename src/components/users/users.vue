<template>
  <el-card class="box-card">
    <!-- 1. 首页/用户管理/用户列表 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 2. 搜索框 -->
    <el-row class="searchRow">
      <el-col :span="8">
        <el-input v-model="query" placeholder="请输入用户名" class="searchInput">
          <el-button slot="append" icon="el-icon-search"></el-button>
        </el-input>
      </el-col>
      <el-col :span="8">
        <el-button type="success">添加用户</el-button>
      </el-col>
    </el-row>

    <!-- 3. 表格 -->
    <el-table :data="userlist" style="width: 100%">
      <el-table-column prop="id" type="index" width="80"></el-table-column>
      <el-table-column prop="username" label="姓名" width="100"></el-table-column>
      <el-table-column prop="email" label="邮箱"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>

      <el-table-column prop="create_time" label="创建日期">
        <!-- 如果单元格的内容不是字符串/文本 需要给要展示的内容外层包裹一个template -->
        <!-- slot-scope 传值 userlist是外层容器绑定的数据 -->
        <!-- userlist中的每个对象中的create_time -->
        <template slot-scope="scope">{{scope.row.create_time | fmtdate}}</template>
      </el-table-column>

      <el-table-column prop="mg_state" label="用户状态">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-row>
            <el-button size="mini" plain type="primary" icon="el-icon-edit" circle></el-button>
            <el-button size="mini" plain type="success" icon="el-icon-check" circle></el-button>
            <el-button size="mini" plain type="danger" icon="el-icon-delete" circle></el-button>
          </el-row>
        </template>
      </el-table-column>
    </el-table>

    <!-- 4. 分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
  </el-card>
</template>

<script>
export default {
  data() {
    return {
      query: "",
      // 提供table要渲染的数据
      // create_time: 1486720211
      // email: "adsfad@qq.com"
      // id: 500
      // mg_state: true
      // mobile: "12345678"
      // role_name: "主管"
      // username: "admin"
      userlist: [
        {
          create_time: "",
          email: "",
          id: "",
          mg_state: "",
          mobile: "",
          role_name: "",
          username: ""
        }
      ],
      // 获取用户数据请求的参数
      pagenum: 1,
      pagesize: 2,
      total: -1
    };
  },
  created() {
    this.getUserList();
  },
  methods: {
    // 分页相关的方法
    // 每页条数改变
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.pagesize = val;
      // 重置页码pagenum
      this.pagenum = 1;
      this.getUserList();
    },
    // 当前页面改变时 pagenum=1->2
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.pagenum = val;
      this.getUserList();
    },
    async getUserList() {
      // 接口文档：需要授权的API -> 设置请求头 Authorization=token ->axios文档
      // 设置请求头 进行授权
      const AUTH_TOKEN = localStorage.getItem("token");
      this.$http.defaults.headers.common["Authorization"] = AUTH_TOKEN;

      const res = await this.$http.get(
        `users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${
          this.pagesize
        }`
      );
      console.log(res);

      const {
        meta: { msg, status },
        data: { total, users }
      } = res.data;
      if (status === 200) {
        this.total = total;
        this.userlist = users;
        this.$message.success(msg);
      } else {
        this.$message.warning(msg);
      }
    }
  }
};
</script>

<style>
.searchRow {
  margin-top: 5px;
}
</style>
