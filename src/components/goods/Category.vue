<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <!--添加分类按钮-->
      <el-row>
        <el-col><el-button type="primary" @click="showAddCateDialog">添加分类</el-button></el-col>
      </el-row>
      <!--表格-->
      <tree-table
        class="tree-table"
        :data="categoryList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-index="true"
        index-text="#"
        :border="true">
        <!--是否有效-->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" style="color: lightgreen;" v-if="scope.row.cat_deleted === false"></i>
          <i class="el-icon-error" style="color: red" v-else></i>
        </template>
        <!--排序-->
        <template slot="sort" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag size="mini" type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!--操作-->
        <template slot="operation" slot-scope="scope">
          <el-button @click="showEditCateDialog(scope.row.cat_id)" type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button @click="removeCateById(scope.row.cat_id)" type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!--分页-->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 5, 10, 20, 50]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>
    <!--添加分类的对话框-->
    <el-dialog
      @close="addCateDialogClosed"
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!--级联选择框 options指定数据源-->
          <el-cascader
            clearable
            v-model="selectedKeys"
            :options="parentCateList"
            :props="{ expandTrigger: 'hover', checkStrictly: 'true', value: 'cat_id', label: 'cat_name', children: 'children' }"
            @change="parentCateChanged">
          </el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="addCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
    </el-dialog>
    <!--修改分类对话框-->
    <el-dialog
      title="修改分类"
      :visible.sync="editCateDialogVisible"
      width="50%">
      <el-form :model="editCateForm" :rules="addCateFormRules" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="editCateDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editCate">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Category',
  data () {
    return {
      categoryList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      // 表格列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'sort'
        },
        {
          label: '操作',
          type: 'template',
          template: 'operation'
        }
      ],
      addCateDialogVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        // 0->一级分类 1->二级分类 2->三级分类
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      selectedKeys: [],
      editCateDialogVisible: false,
      editCateForm: {}
    }
  },
  created () {
    this.getCategoryList()
  },
  methods: {
    getCategoryList () {
      this.$http.get('categories', { params: this.queryInfo }).then(res => {
        // console.log(res)
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.categoryList = res.data.data.result
        this.total = res.data.data.total
      })
    },
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getCategoryList()
    },
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getCategoryList()
    },
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    getParentCateList () {
      this.$http.get('categories', { params: { type: 2 } }).then(res => {
        // console.log(res.data)
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.parentCateList = res.data.data
      })
    },
    parentCateChanged () {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCate () {
      // console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(valid => {
        if (!valid) return
        this.$http.post('categories', this.addCateForm).then(res => {
          if (res.data.meta.status !== 201) return this.$message.error(res.data.meta.msg)
          this.$message.success(res.data.meta.msg)
          this.getCategoryList()
          this.addCateDialogVisible = false
        })
      })
    },
    // 关闭对话框 表单数据重置
    addCateDialogClosed () {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    showEditCateDialog (id) {
      this.$http.get('categories/' + id).then(res => {
        // console.log(res.data)
        if (res.data.meta.status !== 200) {
          return this.$message.error(res.data.meta.msg)
        }
        this.editCateForm = res.data.data
      })
      this.editCateDialogVisible = true
    },
    editCate () {
      this.$refs.editCateFormRef.validate(valid => {
        // console.log(valid)
        if (!valid) return
        this.$http.put('categories/' + this.editCateForm.cat_id, { cat_name: this.editCateForm.cat_name }).then(res => {
          // console.log(res.data)
          if (res.data.meta.status !== 200) {
            return this.$message.error(res.data.meta.msg)
          }
          this.$message.success(res.data.meta.msg)
          this.getCategoryList()
          this.editCateDialogVisible = false
        })
      })
    },
    removeCateById (id) {
      this.$confirm('此操作将永久删除该商品分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete('categories/' + id).then(res => {
          if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
          this.$message.success(res.data.meta.msg)
          this.getCategoryList()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
  .tree-table {
    margin-top: 15px;
  }
  .el-cascader {
    width: 100%;
  }
</style>
