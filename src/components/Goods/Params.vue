<template>
  <div class="">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <!--点击首页会跳转到welcome页面-->
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>
      <el-row>
        <el-col>
          <span class="selectedgoodsCate">选择商品分类：</span>
          <!-- v-model双向绑定选中的id -->
          <el-cascader v-model="selectedKeys" :options="cateList" :props="cascaderProps" @change="parentCateChange" :clearable="true"></el-cascader>
        </el-col>
      </el-row>
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleTagClose(scope.row,i)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column  label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" slot="reference" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" :disabled="isBtnDisabled"  @click="addDialogVisible=true">添加属性</el-button>
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleTagClose(scope.row,i)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="属性名称"></el-table-column>
            <el-table-column  label="操作">
              <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row)">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-delete" slot="reference" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!--添加参数的对话框-->
    <el-dialog :title="'添加'+titleText" width="50%" :visible.sync="addDialogVisible" :close-on-click-modal="false" @close="addDialogClosed">
      <!--内容主体-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!--底部-->
      <span slot="footer">
        <el-button @click="addDialogVisible=false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!--修改参数的对话框-->
    <el-dialog :title="'修改'+titleText" width="50%" :visible.sync="editDialogVisible" :close-on-click-modal="false" @close="editDialogClosed">
      <!--内容主体-->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <!--底部-->
      <span slot="footer">
        <el-button @click="editDialogVisible=false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Params',
  components: {},
  props: {},
  data () {
    return {
      cateList: [],
      cascaderProps: {
        expandTrigger: 'hover',
        checkStrictly: false,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联框选中的id值
      selectedKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入参数名称', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {
    isBtnDisabled() {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的ID
    cateId() {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  },
  watch: {},
  created () {
    this.getCateList()
  },
  mounted () {
  },
  methods: {
    getParamsData() {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        // 选中的不是三级分类
        return
      }
      // 证明选中的是三级分类
      // 根据所选分类的id，和当前所处的面板，获取对应的参数
      this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } }).then(response => {
        const res = response.data
        console.log(res)
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
          // 添加tag的显示状态
          item.inputVisible = false
          item.inputValue = ''
        })
        if (res.meta.status === 200) {
          if (this.activeName === 'many') {
            this.manyTableData = res.data
          } else {
            this.onlyTableData = res.data
          }
        }
      }).catch(() => this.$message.error('请求失败'))
    },
    getCateList() {
      this.$http.get('categories').then(response => {
        const res = response.data
        if (res.meta.status === 200) {
          this.cateList = res.data
        } else {
          this.$message.error(res.meta.msg)
        }
      }).catch(() => {
        this.$message.error('请求失败')
      })
    },
    parentCateChange() {
      // 当点击级联选择框时，会触发该函数，然后向后台获取级联选择器里面所属数据的参数
      this.getParamsData()
    },
    handleTabClick() {
      // 当点击动态参数或者静态属性按钮时，会触发该函数，然后向后台获取级联选择器里面所属数据的参数
      this.getParamsData()
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 确定添加参数
    addParams() {
      this.$refs.addFormRef.validate(valid => {
        if (valid) {
          this.$http.post(`categories/${this.cateId}/attributes`, {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }).then(response => {
            const res = response.data
            if (res.meta.status === 201) {
              this.$message.success(res.meta.msg)
              this.getParamsData()
              this.addDialogVisible = false
            } else {
              this.$message.error(res.meta.msg)
            }
          }).catch(() => {
            this.$message.error('操作失败')
          })
        }
      })
    },
    editParams() {
      this.$refs.editFormRef.validate(valid => {
        if (valid) {
          this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
            attr_name: this.editForm.attr_name,
            attr_sel: this.activeName
          }).then(response => {
            const res = response.data
            if (res.meta.status === 200) {
              this.$message.success(res.meta.msg)
              this.getParamsData()
              this.editDialogVisible = false
            } else {
              this.$message.error(res.meta.msg)
            }
          }).catch(() => {
            this.$message.error('操作失败')
          })
        }
      })
    },
    showEditDialog(paramsInfo) {
      // paramsInfo中没有对象，直接深拷贝一份，也没必要从服务端重新获取了
      // 如果直接赋值，则为引用，表格上的数据也会随对话框中数据的修改而实时改变
      this.editForm = { ...paramsInfo }
      this.editDialogVisible = true
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // eslint-disable-next-line camelcase
    removeParams(attrId) {
      this.$confirm('此操作将永久删除改参数,是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(response => {
        if (response !== 'confirm') {
          return this.$message.info('已经取消删除')
        }
        // eslint-disable-next-line camelcase
        this.$http.delete(`categories/${this.cateId}/attributes/${attrId}`).then(response => {
          const res = response.data
          if (res.meta.status === 200) {
            this.getParamsData()
            this.$message.success('删除数据成功')
          }
        }).catch(() => this.$message.info('删除数据失败'))
      }).catch(
        err => err
      )
    },
    // 确定添加tag
    handleInputConfirm(row) {
      if (row.inputValue.trim().length !== 0) {
        // 将新添加的tag push进数组
        row.attr_vals.push(row.inputValue.trim())
        // 向服务端put新数据
        this.saveAttrVals(row)
      }
      row.inputValue = ''
      row.inputVisible = false
    },
    // 将 attr_vals 保存到服务端
    saveAttrVals(row) {
      this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(',')
      }).then(response => {
        const res = response.data
        if (res.meta.status === 200) {
          this.$message.success(res.meta.msg)
        } else {
          this.$message.error(res.meta.msg)
        }
      }).catch(() => {
        this.$message.error('操作失败')
      })
    },
    showInput(row) {
      row.inputVisible = true
      // 让文本框自动获得焦点，$nextTick当页面重新渲染后才会执行里面的代码，
      // 当inputVisible = true，页面的元素还没有重新渲染，显示的是but而不是input，因此
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleTagClose(row, i) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  }
}
</script>

<style scoped lang="less">
  .el-alert{
    margin-bottom:15px;
  }
  .selectedgoodsCate{
    font-size: 14px;
  }
  .el-tag{
    margin-right:10px;
  }
  .input-new-tag{
    width: 120px;
  }
</style>>
