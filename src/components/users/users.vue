<template>
  <el-card class="box-cart">
    <!-- 1.面包屑 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 2.搜索 -->
    <el-row class="searchRow">
      <el-col>
        <el-input
          @clear="loadUserList"
          placeholder="请输入内容"
          v-model="query"
          class="inputSearch"
          clearable
        >
          <el-button @click="searchUser" slot="append" icon="el-icon-search"></el-button>
        </el-input>
        <el-button @click="showAddUserDia" type="success">添加用户</el-button>
      </el-col>
    </el-row>
    <!-- 3.表格 -->
    <el-table :data="userlist" style="width: 100%">
      <el-table-column type="index" label="#" width="60"></el-table-column>
      <el-table-column prop="username" label="姓名" width="80"></el-table-column>
      <el-table-column prop="email" label="邮箱"></el-table-column>
      <el-table-column prop="mobile" label="电话"></el-table-column>
      <el-table-column label="创建时间">
        <template slot-scope="scope">{{scope.row.create_time | fmdate}}</template>
      </el-table-column>
      <el-table-column label="用户状态">
        <template slot-scope="scope">
          <el-switch :disabled="scope.row.username==='admin' ? true : false"
            @change="changeMgState(scope.row)"
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column prop="address" label="操作">
        <template slot-scope="scope">
          <el-button
            :disabled="scope.row.username==='admin' ? true : false"
            size="mini"
            :plain="false"
            type="primary"
            icon="el-icon-edit"
            @click="showEditUserDia(scope.row)"
            circle
          ></el-button>
          <el-button
            :disabled="scope.row.username==='admin' ? true : false"
            size="mini"
            :plain="false"
            type="danger"
            icon="el-icon-delete"
            @click="showDeleUserMsgBox(scope.row.id)"
            circle
          ></el-button>
          <el-button
            :disabled="scope.row.username==='admin' ? true : false"
            @click="showSetUserRoledia(scope.row)"
            size="mini"
            :plain="false"
            type="success"
            icon="el-icon-check"
            circle
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 4.分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>

    <!-- 添加用户的对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
      <el-form :model="form">
        <el-form-item label="用户名" label-width="100px">
          <el-input v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密 码" label-width="100px">
          <el-input type="password" v-model="form.password" autocomplete="off"></el-input>
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
        <el-button type="primary" @click="addUser">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 编辑用户对话框 -->
    <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
      <el-form :model="form">
        <el-form-item label="用户名" label-width="100px">
          <el-input disabled v-model="form.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" label-width="100px">
          <el-input v-model="form.email" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="电话" label-width="100px">
          <el-input v-model="form.mobile" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="calcelEdit">取 消</el-button>
        <el-button type="primary" @click="editUser()">确 定</el-button>
      </div>
    </el-dialog>

    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRlo">
      <el-form :model="form">
        <el-form-item label="用户名" label-width="100px">
          {{currUserName}}
        </el-form-item>
        <el-form-item label="角色" label-width="100px">
          <el-select v-model="currRoleId">
            <el-option label="请选择" :value="-1"></el-option>
            <el-option :label="item.roleName" :value="item.id"
            v-for="(item,i) in roles" :key="i"
            ></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleRlo = false">取 消</el-button>
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
      //表格的数据
      userlist: [],
      //分页相关数据
      total: -1,
      pagenum: 1,
      pagesize: 2,
      //添加对话框的属性
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleRlo: false,
      //添加用户表单数据
      form: {
        username: "",
        password: "",
        email: "",
        mobule: ""
      },
      //分配角色
      currRoleId: -1,
      //当前用户id
      currUserId: -1,
      currUserName:'',
      //保存所有角色数据
      roles:[]
    };
  },
  created() {
    this.getUserList();
  },
  methods: {
    //确认分配角色
    async setRole(){
      // users/:id/role
      const res = await this.$http.put(`users/${this.currUserId}/role`,{rid:this.currRoleId})
      // console.log(res)
      if (res.data.meta.status ===200) {
        this.$message.success(res.data.meta.msg)
        this.dialogFormVisibleRlo = false
      }else{
        this.$message.warning(es.data.meta.msg)
      }
    },
    //分配角色
    async showSetUserRoledia(user) {
      this.dialogFormVisibleRlo = true
      this.currUserName = user.username
      this.currUserId = user.id
      //获取所有的角色
      const res1 = await this.$http.get(`roles`)
      // console.log(res1)
      this.roles = res1.data.data
      //获取当前用户的角色 id
      const res = await this.$http.get(`users/${user.id}`)
      // console.log(res)
      this.currRoleId = res.data.data.rid
    },
    //获取用户列表的请求
    // query查询参数可以为空pagenum当前页码不能为空pagesize每页显示条数不能为空
    //需要授权携带 token
    async getUserList() {
      const token = localStorage.getItem("token");
      this.$http.defaults.headers.common["Authorization"] = token;
      const res = await this.$http.get(
        `users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`
      );
      console.log(res);
      const {
        meta: { status, msg },
        data: { users, total }
      } = res.data;
      if (status === 200) {
        //给数据赋值
        this.userlist = users;
        this.total = total;
        // 提示
        // this.$message.success(msg);
      } else {
        // this.$message.warning(msg);
      }
    },
    //分页相关方法
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.pagesize = val;
      this.pagenum = 1;
      this.getUserList();
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.pagenum = val;
      this.getUserList();
    },
    //搜索用户
    searchUser() {
      this.getUserList();
    },
    //清空搜索
    loadUserList() {
      this.getUserList();
    },
    //添加用户 --显示对话框
    showAddUserDia() {
      this.from = {};
      this.dialogFormVisibleAdd = true;
    },
    //确认添加用户
    async addUser() {
      //2.关闭对话框
      this.dialogFormVisibleAdd = false;
      const res = await this.$http.post("users", this.form);
      // console.log(res)
      const {
        meta: { status, msg },
        data
      } = res.data;
      if (status === 201) {
        //1.提示成功
        this.$message.success(msg);
        //3.更新试图
        this.getUserList();
        //4.清空文本框
        this.form = {};
      } else {
        this.$message.warning(msg);
      }
    },
    //删除用户 --打开消息弹框
    showDeleUserMsgBox(userId) {
      this.$confirm("删除用户?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          //删除用户请求
          const res = await this.$http.delete(`users/${userId}`);
          console.log(res);
          if (res.data.meta.status === 200) {
            this.$message({
              type: "success",
              message: "删除成功!"
            });
            this.pagenum = 1;
            this.getUserList();
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    //编辑用户 -- 显示对话框
    showEditUserDia(user) {
      this.form = user;
      this.dialogFormVisibleEdit = true;
    },
    //确认编辑
    async editUser() {
      // users/:id
      const res = await this.$http.put(`users/${this.form.id}`, this.form);
      console.log(res);
      //关闭对话框
      if (res.data.meta.status === 200) {
        this.$message.success(res.data.meta.msg);
        this.dialogFormVisibleEdit = false;
        //更新试图
        this.getUserList();
      } else {
        this.$message.warning(res.data.meta.msg);
      }
    },
    //取消编辑
    calcelEdit() {
      this.dialogFormVisibleEdit = false;
      this.getUserList();
    },
    //修改状态
    async changeMgState(user) {
      //users/:uId/state/:type
      const res = await this.$http.put(
        `users/${user.id}/state/${user.mg_state}`
      );
      console.log(res);
    }
  }
};
</script>

<style>
.box-cart {
  height: 100%;
}
.inputSearch {
  width: 400px;
}
.searchRow {
  margin-top: 20px;
}
</style>