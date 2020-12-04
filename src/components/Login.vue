<template>
  <div class="login_container">
    <div class="login_box">
      <div class="avatar_box">
        <img src="../assets/img/logo.png" alt="">
      </div>
      <el-form :model='loginForm' ref="loginFormRef" :rules="loginFormRules" label-width="0px" class="login_form">
        <el-form-item prop="username">
          <el-input v-model='loginForm.username' prefix-icon="iconfont icon-01"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model='loginForm.password' type="password" prefix-icon="iconfont icon-mima"></el-input>
        </el-form-item>
        <el-form-item class="login_button">
          <el-button type="primary" @click="LoginFormBtn">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  components: {},
  props: {},
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          { required: true, message: '请输入登录名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {},
  watch: {},
  created () {
  },
  mounted () {
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    LoginFormBtn () {
      this.$refs.loginFormRef.validate(async (valid) => {
        if (!valid) return
        const { data } = await this.$http.post('login', this.loginForm)
        if (data.meta.status !== 200) {
          return this.$message.error('登录失败')
        } else {
          this.$message.success(' 登录成功 ')
          window.sessionStorage.setItem('token', data.data.token)
          this.$router.push('/home')
        }
        console.log(data)
      })
    }
  }
}
</script>

<style scoped lang="less">
  .login_container{
    background:#2b4b6b ;
    height: 100%;
    .login_box{
      width: 450px;
      height: 300px;
      background: #ffffff;
      border-radius: 3px;
      position: absolute;
      left:50%;
      top:50%;
      transform:translate(-50%,-50%);
      .avatar_box{
        height: 130px;
        width: 130px;
        border: 1px solid #eee;
        border-radius: 50%;
        padding: 10px;
        box-shadow: 0 0 10px #ddd;
        position: absolute;
        left:50%;
        transform:translate(-50%,-50%);
        background: #ffffff;
        img{
          width: 100%;
          height: 100%;
          border-radius: 50%;
          background-color: #eeeeee;
        }
      }
      .login_form{
        position: absolute;
        bottom: 0;
        width: 100%;
        box-sizing: border-box;
        padding: 15px;
        .login_button{
          display: flex;
          justify-content: flex-end;
        }
      }
    }
  }
</style>
