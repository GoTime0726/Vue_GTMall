<template>
  <div class="">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <!--点击首页会跳转到welcome页面-->
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addCate">添加分类</el-button>
        </el-col>
      </el-row>
      <tree-table class="treeTable" :data="goodsCategoriesLists" :columns="columns" :selection-type="false" :expand-type="false" show-index
                  :show-row-hover="false" border>
        <template slot="isOK" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen"></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0" type="primary">一级</el-tag>
          <el-tag v-else-if='scope.row.cat_level===1'  type="success">二级</el-tag>
          <el-tag v-else type="warning">三级</el-tag>
        </template>
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
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
    <el-dialog v-dialogDrag title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <!-- 内容主体区域 -->
      <el-form :model="addCateForm" :rules="addCateRules" ref="addCateRef" label-width="100px">
        <el-form-item label="分类名称:" prop="cat_name">
        <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options指定数据源，props用来指定配置项 -->
          <el-cascader v-model="selectedKeys" :options="parentCateList" :props="cascaderProps" @change="parentCateChange" :clearable="true"></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
    <span slot="footer" class="dialog-footer">
      <el-button @click="addCateDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="addCateSure">确 定</el-button>
    </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Categories',
  components: {},
  props: {},
  data () {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      goodsCategoriesLists: [],
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        // 将当前列作为模板列
        {
          label: '是否有效',
          type: 'template',
          template: 'isOK'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }
        ]
      },
      // 定义父级分类数据列表
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        // 设置为可选中一级分类
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectedKeys: []
    }
  },
  computed: {},
  watch: {},
  created () {
    this.getCategoriesLists()
  },
  mounted () {
  },
  methods: {
    async getCategoriesLists () {
      const { data } = await this.$http.get('categories', { params: this.queryInfo })
      if (data.meta.status !== 200) {
        return this.$messege.error('获取商品分类列表失败')
      }
      console.log(data)
      this.goodsCategoriesLists = data.data.result
      this.total = data.data.total
    },
    // 点击每页几个页面时触发该函数，并把最新的显示多少条数据传给 queryInfo.pagesize
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCategoriesLists()
    },
    // 当监听到当前页面发生变化时触发该函数，比如点击第二页的时候，会把最新的一页赋给queryInfo.pagenum
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCategoriesLists()
    },
    addCate () {
      // 先获取父级分类数据列表，然后再显示对话框
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    addCateDialogClosed () {
      this.$refs.addCateRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 获取父级分类数据列表
    async getParentCateList () {
      const { data } = await this.$http.get('categories', {
        param: { type: 2 }
      })
      if (data.meta.status !== 200) {
        return this.$message.error('获取分类列表失败')
      }
      this.parentCateList = data.data
      console.log(this.parentCateList)
    },
    // 选择项发生变化触发该函数
    parentCateChange () {
      console.log(this.selectedKeys)
      if (this.selectedKeys.length > 0) {
        // 大于0证明选中分类父级分类的Id
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    addCateSure () {
      this.$refs.addCateRef.validate(valid => {
        if (!valid) return
        this.$http.post('categories', this.addCateForm).then(response => {
          const res = response.data
          if (res.meta.status === 201) {
            this.$message.success(res.meta.msg)
            this.getCategoriesLists()
            this.addCateDialogVisible = false
          } else {
            this.$message.error(res.meta.msg)
          }
        }).catch(() => {
          this.$message.error('操作失败')
        })
      })
    }
  }
}
</script>

<style scoped lang="less">
  .treeTable{
    margin-top: 10px;
  }
  .el-cascader{
    width: 100%;
  }
</style>
