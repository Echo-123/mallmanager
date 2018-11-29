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
        <el-input
          v-model="query"
          @clear="loadUserList()"
          clearable
          placeholder="请输入用户名"
          class="searchInput"
        >
          <el-button slot="append" @click="searchUser()" icon="el-icon-search"></el-button>
        </el-input>
      </el-col>
      <el-col :span="8">
        <el-button type="success" @click="showAddUserDia()">添加用户</el-button>
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
          <el-switch
            v-model="scope.row.mg_state"
            @change="changeUserMgState(scope.row)"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-row>
            <el-button
              size="mini"
              plain
              type="primary"
              @click="showMegBoxEdit(scope.row)"
              icon="el-icon-edit"
              circle
            ></el-button>
            <el-button
              size="mini"
              plain
              type="success"
              @click="showRoleBoxDia(scope.row)"
              icon="el-icon-check"
              circle
            ></el-button>
            <el-button
              size="mini"
              plain
              type="danger"
              @click="showMegBoxDele(scope.row.id)"
              icon="el-icon-delete"
              circle
            ></el-button>
          </el-row>
        </template>
      </el-table-column>
    </el-table>

    <!-- 4. 分页 -->
    <el-pagination
      class="pagination"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>

    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
      <el-form :model="form">
        <el-form-item label="用户名称" label-width="100px">
          <el-input v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" label-width="100px">
          <el-input v-model="form.password" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" label-width="100px">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-width="100px">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
        <el-button type="primary" @click="addUser()">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 编辑用户对话框 -->
    <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
      <el-form :model="form">
        <el-form-item label="用户名称" label-width="100px">
          <el-input v-model="form.username" disabled autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" label-width="100px">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-width="100px">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleEdit = false">取 消</el-button>
        <el-button type="primary" @click="editUser()">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRole">
      <el-form :model="form">
        <el-form-item label="用户名" label-width="100px">{{currentUserName}}</el-form-item>
        <el-form-item label="角色" label-width="100px">
          <el-select v-model="currRoleId" placeholder="请选择">
            <el-option label="请选择" :value="-1"></el-option>
            <el-option v-for="(item, i) in roles" :key="i" :label="item.roleName" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleRole = false">取 消</el-button>
        <el-button type="primary" @click="setRole()">确 定</el-button>
      </div>
    </el-dialog>
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
      total: -1,
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleRole: false,
      form: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      currentUserName: "",
      currRoleId: -1,
      // 保存角色信息的数组
      roles: [],
      currUserId: -1
    };
  },
  created() {
    this.getUserList();
  },
  methods: {
    // 分配/设置用户的角色 --发送修改的请求
    async setRole() {
      this.dialogFormVisibleRole = false;
      const res = await this.$http.put(`users/${this.currUserId}/role`, {
        rid: this.currRoleId
      });
    },
    // 分配/设置用户的角色 --打开对话框
    async showRoleBoxDia(user) {
      // 给当前的用户id赋值
      this.currUserId = user.id;

      this.dialogFormVisibleRole = true;
      // 获取当前用户名
      this.currentUserName = user.username;

      const res = await this.$http.get(`roles`);
      // console.log(res);
      this.roles = res.data.data;

      const res2 = await this.$http.get(`users/${user.id}`);
      this.currRoleId = res2.data.data.rid;
      // console.log(res2);
    },
    // 修改用户状态
    async changeUserMgState(user) {
      const res = await this.$http.put(
        `users/${user.id}/state/${user.mg_state}`
      );
      // console.log(res);
      const {
        meta: { msg, status },
        data
      } = res.data;
      if (status === 200) {
        this.$message.success(msg);
      }
    },
    // 编辑 - 发送请求
    async editUser(userId) {
      const res = await this.$http.put(`users/${this.form.id}`, this.form);
      // console.log(res);
      const {
        meta: { msg, status },
        data
      } = res.data;
      if (status === 200) {
        this.$message.success(msg);
        this.dialogFormVisibleEdit = false;
        this.getUserList();
      }
    },
    // 编辑用户 - 显示对话框
    showMegBoxEdit(user) {
      this.dialogFormVisibleEdit = true;
      this.form = user;
    },
    // 删除用户
    showMegBoxDele(userId) {
      this.$confirm("是否删除用户?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
        center: true
      })
        .then(async () => {
          // 发送删除请求
          const res = await this.$http.delete(`users/${userId}`);
          console.log(res);
          // 更新视图
          this.getUserList();

          this.$message({
            type: "success",
            message: res.data.meta.msg
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: res.data.meta.msg
          });
        });
    },
    // 添加用户 - 发送请求
    async addUser() {
      // 2. 关闭对话框
      this.dialogFormVisibleAdd = false;
      // 1. 发送请求
      const res = await this.$http.post("users", this.form);
      console.log(res);
      const {
        meta: { msg, status },
        data
      } = res.data;
      if (status === 201) {
        // 3. 提示成功
        this.$message.success(msg);
        // 4. 更新视图
        this.getUserList();
        // 5. 清空输入框
        // this.form = {};
        for (const key in this.form) {
          if (this.form.hasOwnProperty(key)) {
            this.form[key] = "";
          }
        }
      } else {
        this.$message.warning(msg);
      }
    },
    // 添加用户 - 显示对话框
    showAddUserDia() {
      this.form = {};
      this.dialogFormVisibleAdd = true;
    },
    loadUserList() {
      this.getUserList();
    },
    // 查询
    searchUser() {
      this.pagenum = 1;
      this.getUserList();
    },
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
  margin-top: 20px;
}
.pagination {
  margin-top: 20px;
}
</style>
