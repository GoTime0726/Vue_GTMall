<template>
  <div class="">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="RolesLists" stripe border>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id" :class="['bdBottom',i1===0?'bdTop':'','rightsCenter']">
              <el-col :span="5">
                <el-tag closable @close='removeRightsById(scope.row,item1.id)'>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2===0?'':'bdTop','rightsCenter']">
                  <el-col :span="6">
                    <el-tag type="success" closable @close='removeRightsById(scope.row,item2.id)'>{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable @close='removeRightsById(scope.row,item3.id)' v-for="(item3) in item2.children" :key='item3.id' :class="[i2===0?'':'bdTop']" type='warning'>
                      {{item3.authName}}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
            <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
            <el-button type="warning" size="mini" icon="el-icon-setting" @click="showRightsDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-dialog title="分配权限" :visible.sync="showRightsdialogVisible" width="50%" @close="setRightsDialogClosed">
      <el-tree  ref="treeRef" show-checkbox node-key="id" default-expand-all :data="allRights" :default-checked-keys="dfk" :props="treeProps">
      </el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showRightsdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Roles',
  components: {},
  props: {},
  data () {
    return {
      RolesLists: [],
      showRightsdialogVisible: false,
      allRights: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认勾选的节点的 key 的数组
      dfk: [],
      roleId: ''
    }
  },
  computed: {},
  watch: {},
  created () {
    this.getRolesLists()
  },
  mounted () {
  },
  methods: {
    async getRolesLists() {
      const { data } = await this.$http.get('roles')
      if (data.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.RolesLists = data.data
    },
    async removeRightsById(role, rightId) {
      console.log(role)
      const result = await this.$confirm('确认删除该标签吗?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') return this.$message.info('您已经取消删除')
      const { data } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (data.meta.status !== 200) { return this.$message.error('删除角色指定权限失败') }
      // role为三级权限所对应的对象，自身有children属性，由于删除后可以拿到最新的数据，因此通过以下赋值即可，不需要重新获取列表
      role.children = data.data
      this.$message.success('删除角色指定权限成功')
    },
    async showRightsDialog(role) {
      this.roleId = role.id
      const { data } = await this.$http.get('rights/tree')
      this.allRights = data.data
      this.getLeafKeys(role, this.dfk)
      this.showRightsdialogVisible = true
    },
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    setRightsDialogClosed() {
      this.dfk = []
    },
    async allotRight() {
      // 得到全选和半选状态下的对应的 被选择的节点的key,即所谓的id
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (data.meta.status !== 200) {
        return this.$message.error('获取失败')
      }
      console.log('1')
      this.$message.success('获取成功')
      this.showRightsdialogVisible = false
      this.getRolesLists()
    }
  }
}
</script>

<style scoped lang="less">
  .el-tag {
    margin: 7px;
  }
  .bdTop{
    border-top: 1px solid #eee;
  }
  .bdBottom{
    border-bottom: 1px solid #eee;
  }
  .rightsCenter{
    display: flex;
    align-items: center;
  }
</style>
