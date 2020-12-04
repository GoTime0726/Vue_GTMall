<template>
  <div class="home_ContainerBox">
    <el-container class="home_Container">
      <el-header >
        <div>
          <img src="../assets/img/heima.png" alt="">
          <span>电商后台管理系统</span>
        </div>
        <el-button type="info" @click="logOut">退出</el-button>
      </el-header>
      <el-container>
        <el-aside :width="isCollapse?'64px':'200px'">
          <div class="toggle-button" @click="toggle">| | |</div>
          <el-menu router :collapse-transition='false' :collapse='isCollapse' :default-active="activePath"
                   background-color="#333744" text-color="#fff" active-text-color="#409eff" :unique-opened="true">
            <!--:表示属性绑定的是布尔值，不加则表示为字符串，或者直接写unique-opened-->
            <!--一级菜单-->
            <el-submenu :index="index+''" v-for="(item,index) in menuLists" :key="index">
              <!--一级菜单模板区域-->
              <template slot="title">
                <!--图标和文本-->
                <i :class="iconObj[item.id]" style="margin-right: 10px"></i>
                <span>{{item.authName}}</span>
              </template>
              <el-menu-item :index="'/'+menuitem.path" @click="saveNavState('/'+menuitem.path)"
                            v-for="(menuitem,menuIndex) in item.children" :key="menuIndex">
                <template slot="title" >
                  <i class="el-icon-menu"></i>
                  <span>{{menuitem.authName}}</span>
                </template>
              </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
export default {
  name: 'Home',
  components: {},
  props: {},
  data () {
    return {
      menuLists: [],
      iconObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-quanxian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-dingdan',
        145: 'iconfont icon-tongji'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  computed: {},
  watch: {},
  created () {
    this.getMenuLists()
    this.activePath = window.sessionStorage.getItem('activePath1')
  },
  mounted () {
  },
  methods: {
    logOut () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuLists () {
      const { data } = await this.$http.get('/menus')
      console.log(data)
      if (data.meta.status !== 200) return this.$message.error('获取菜单列表失败')
      this.menuLists = data.data
    },
    toggle() {
      this.isCollapse = !this.isCollapse
    },
    saveNavState(activePath) {
      this.activePath = activePath
      window.sessionStorage.setItem('activePath1', activePath)
    }
  }
}
</script>

<style scoped lang="less">
  .home_ContainerBox{
    height: 100%;
  }
  .home_Container{
    height: 100%;
    .el-header{
      background-color: #373D41;
      display: flex;
      justify-content: space-between;
      padding-left: 0;
      align-items: center;
      color: #fff;
      font-size: 20px;
      div{
        display: flex;
        align-items: center;
        span{
          margin-left: 15px;
        }
      }
    }
    .el-aside{
      background-color: #333744;
      .toggle-button{
        background-color: #4a5064;
        font-size: 10px;
        line-height: 24px;
        color:#fff;
        text-align: center;
        letter-spacing: 0.2em;
        cursor:pointer
      }
      .el-menu{
        border-right: none ;
      }
    }
    .el-main{
      background-color:#eaedfa;
    }
  }

</style>
