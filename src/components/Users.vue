<template>
  <div class="">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <!--点击首页会跳转到welcome页面-->
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图，就是包围搜索框外边的框框-->
    <el-card class="box-card">
      <!--搜索添加-->
      <el-row :gutter="20"> <!--一行中列与列的间距-->
        <el-col :span="10">  <!--一行中某一列的宽度-->
          <el-input placeholder="请输入内容"  v-model="queryInfo.query" :clearable="true" @clear="getUserLists">
            <el-button slot="append" icon="el-icon-search" @click="getUserLists"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
        <el-col :span="6"><div class="grid-content bg-purple"></div></el-col>
      </el-row>
      <el-table :data="userList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名" ></el-table-column>
        <el-table-column prop="email" label="邮箱" ></el-table-column>
        <el-table-column prop="mobile" label="电话" ></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template slot-scope="scope">
            <el-switch @change="mgStateChange(scope.row)"
              v-model="scope.row.mg_state">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope"> <!--slot-scope接收到作用域插槽的数据scope=scope.row这一行数据-->
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="showDeleteDialog(scope.row.id)"></el-button>
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"  @click="setRoles(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!--handleSizeChange:监听pagesize发生
          handleCurrentChange：监听页码值改变，可以拿到最新的页码值，newpage
          current-page：当前所在显示的第几页
          page-sizes：供可选择的每页切换几条数据的下拉
          page-size： 当前情况下每页显示多少条数据
      -->
      <el-pagination
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        :page-sizes="[1, 2, 5, 8]"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-size="queryInfo.pagesize"
        >
      </el-pagination>
    </el-card>
    <el-dialog v-dialogDrag title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <!-- 内容主体区域 -->
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
    <el-dialog v-dialogDrag title="编辑用户" :visible.sync="editDialogVisible" width="50%"
                            @close="editDialogClose">
    <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="editForm.username" disabled></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
        <el-input v-model="editForm.email"></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="mobile">
        <el-input v-model="editForm.mobile"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
  </el-dialog>
  <el-dialog  @close="setRolesDialogClosed"
    v-dialogDrag title="分配角色" :visible.sync="setRolesDialogVisible" width="50%">
    <div>
      <p>当前用户：{{roleInfo.username}}</p>
      <p>当前角色：{{roleInfo.role_name}}</p>
      <p>分配新角色：
        <el-select v-model="selectedRoleId" placeholder="请选择">
          <el-option v-for="item in roleList" :key="item.id" :label="item.roleName" :value="item.id">
          </el-option>
        </el-select>
      </p>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="setRolesDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="saveSetRoles">确 定</el-button>
    </span>
  </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'users',
  components: {},
  props: {},
  data () {
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
      if (regEmail.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的邮箱'))
    }
    var checkMobile = (rule, value, callback) => {
      const regPhone = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regPhone.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的手机号'))
    }
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 3
      },
      userList: [],
      total: 0,
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '用户名的长度在3~10个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '用户名的长度在6~15个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editForm: { username: '' },
      editFormRules: {
        email: [{ required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      deleteDialogVisible: false,
      setRolesDialogVisible: false,
      roleInfo: {},
      roleList: [],
      selectedRoleId: ''
    }
  },
  computed: {},
  watch: {},
  created () {
    this.getUserLists()
  },
  mounted () {
  },
  methods: {
    async getUserLists() {
      /* 请求方式为get，因此需要用到params作为参数 */
      const { data } = await this.$http.get('users', { params: this.queryInfo })
      if (data.meta.status !== 200) return this.$message.error('获取用户列表失败')
      this.userList = data.data.users
      this.total = data.data.total
    },
    /* 监听Pagesize发生改变 */
    handleSizeChange (newsize) {
      this.queryInfo.pagesize = newsize
      this.getUserLists()
    },
    /* 监听页码值改变 */
    handleCurrentChange (newpage) {
      this.queryInfo.pagenum = newpage
      this.getUserLists()
    },
    /* 一旦监听到switch开关发生改变，就会执行该函数，并把scope.row作为实参传给userIfo,控制台输出userIfo对象，根据API接口并把当前状态传到后台 */
    async mgStateChange(userInfo) {
      /* 通过后台去修改用户数据，持久化，刷新也不会改变 */
      const { data } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      if (data.meta.status !== 200) {
        /* 当没有请求到后台数据或者请求失败，在点击更改状态后，状态会改变，但是后台并没有做修改，因此需要重置其为点击前的状态 */
        userInfo.meta.mg_state = !userInfo.meta.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        // 隐藏添加用户的对话框
        this.addDialogVisible = false
        // 重新获取用户列表数据
        this.getUserLists()
      })
    },
    async showEditDialog(id) {
      this.editDialogVisible = true
      const { data } = await this.$http.get(`users/${id}`)
      if (data.meta.status !== 200) return this.$message.error('获取用户信息失败')
      this.editForm = data.data
    },
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    editUser() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return this.$message.error('表单输入有误')
        const { data } = await this.$http.put(`users/${this.editForm.id}`, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (data.meta.status !== 200) {
          return this.$message.error('更新用户数据失败')
        }
        this.getUserLists()
        this.editDialogVisible = false
        this.$message.success('更新用户数据成功')
      })
    },
    async showDeleteDialog(id) {
      const result = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      console.log(id)
      if (result !== 'confirm') return this.$message.info('您已经取消删除')
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败！')
      }
      this.$message.success('删除用户成功！')
      this.getUserLists()
    },
    async setRoles(role) {
      const { data } = await this.$http.get('roles')
      if (data.meta.status !== 200) return this.$message.error('获取角色信息失败')
      this.roleList = data.data
      this.setRolesDialogVisible = true
      this.roleInfo = role
    },
    async saveSetRoles() {
      if (!this.selectedRoleId) {
        return this.$message.error('请选择要分配的角色')
      }
      const { data } = await this.$http.put(`users/${this.roleInfo.id}/role`, { rid: this.selectedRoleId })
      console.log(data)
      if (data.meta.status !== 200) {
        return this.$message.error('分配用户角色失败')
      }
      this.getUserLists()
      this.setRolesDialogVisible = false
      this.$message.success('分配用户角色成功')
    },
    setRolesDialogClosed() {
      this.selectedRoleId = ''
      this.roleInfo = ''
    }
  }
}
</script>

<style scoped lang="less">
</style>
