<template>
    <div>
      <!--面包屑导航区域-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>参数列表</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片视图区域-->
      <el-card>
        <!--警告区域-->
        <el-alert
          :closable="false"
          title="注意：只允许为第三级分类设置相关参数！"
          type="warning"
          show-icon>
        </el-alert>
        <!--选择商品分类区域-->
        <el-row class="cat_opt">
          <el-col>
            <span>选择商品分类：</span>
            <el-cascader
              v-model="selectedCateKeys"
              :options="categoriesList"
              :props="{ expandTrigger: 'hover', label: 'cat_name',value: 'cat_id', children: 'children' }"
              @change="handleCateChange">
            </el-cascader>
          </el-col>
        </el-row>
        <!--tab标签页区域-->
        <el-tabs v-model="activeName" @tab-click="handleTabClick">
          <el-tab-pane label="动态参数" name="many">
            <el-button @click="addDialogVisible = true" :disabled="isBtnDisabled" type="primary" size="mini">添加参数</el-button>
            <el-table :data="manyTableData" :border="true" :stripe="true">
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag @close="handleClosed(i, scope.row)" :closable="true" v-for="(item, i) in scope.row.attr_vals" :key="i">{{item}}</el-tag>
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
              <el-table-column type="index" label="#"></el-table-column>
              <el-table-column label="参数名称" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button @click="showEditDialog(scope.row.attr_id)" size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                  <el-button @click="removeParams(scope.row.attr_id)" size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
          <el-tab-pane label="静态属性" name="only">
            <el-button @click="addDialogVisible = true" :disabled="isBtnDisabled" type="primary" size="mini">添加属性</el-button>
            <el-table :data="onlyTableData" :border="true" :stripe="true">
              <el-table-column type="expand">
                <template slot-scope="scope">
                  <el-tag @close="handleClosed(i, scope.row)" :closable="true" v-for="(item, i) in scope.row.attr_vals" :key="i">{{item}}</el-tag>
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
              <el-table-column type="index" label="#"></el-table-column>
              <el-table-column label="属性名称" prop="attr_name"></el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button @click="showEditDialog(scope.row.attr_id)" size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                  <el-button @click="removeParams(scope.row.attr_id)" size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-tab-pane>
        </el-tabs>
      </el-card>
      <!--添加参数的对话框-->
      <el-dialog
        @close="addDialogClosed"
        :title="'添加' + titleText"
        :visible.sync="addDialogVisible"
        width="50%">
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
          <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="addForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
      </el-dialog>
      <!--修改参数的对话框-->
      <el-dialog
        @close="editDialogClosed"
        :title="'修改' + titleText"
        :visible.sync="editDialogVisible"
        width="50%">
        <el-form :model="editForm" :rules="addFormRules" ref="editFormRef" label-width="100px">
          <el-form-item :label="titleText" prop="attr_name">
            <el-input v-model="editForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
      </el-dialog>
    </div>
</template>

<script>
export default {
  name: 'Params',
  data () {
    return {
      categoriesList: [],
      selectedCateKeys: [],
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入内容', trigger: 'blur' }
        ]
      },
      editForm: {},
      attrValues: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    getCateList () {
      this.$http.get('categories').then(res => {
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.categoriesList = res.data.data
      })
    },
    getParamsList () {
      this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } }).then(res => {
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        res.data.data.forEach(item => {
          item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
          // 添加文本框的显示隐藏
          item.inputVisible = false
          item.inputValue = ''
        })
        // console.log(res.data.data)
        if (this.activeName === 'many') {
          this.manyTableData = res.data.data
        } else {
          this.onlyTableData = res.data.data
        }
      })
    },
    handleCateChange () {
      // 不是三级分类无法选中
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // console.log(this.selectedCateKeys)
      this.getParamsList()
    },
    handleTabClick () {
      console.log(this.activeName)
      this.getParamsList()
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addParams () {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return
        this.$http.post(`categories/${this.cateId}/attributes`, { attr_name: this.addForm.attr_name, attr_sel: this.activeName }).then(res => {
          // console.log(res.data)
          if (res.data.meta.status !== 201) return this.$message.error(res.data.meta.msg)
          this.$message.success(res.data.meta.msg)
          this.addDialogVisible = false
          this.getParamsList()
        })
      })
    },
    showEditDialog (id) {
      this.editDialogVisible = true
      this.$http.get(`categories/${this.cateId}/attributes/${id}`, { params: { attr_sel: this.activeName } }).then(res => {
        // console.log(res.data)
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.editForm = res.data.data
        this.attrValues = res.data.data.attr_vals ? res.data.data.attr_vals.split(',') : []
      })
    },
    editParams () {
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return
        this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, { attr_name: this.editForm.attr_name, attr_sel: this.activeName, attr_vals: this.attrValues.join(',') }).then(res => {
          // console.log(res.data)
          if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
          this.$message.success(res.data.meta.msg)
          this.editDialogVisible = false
          this.getParamsList()
        })
      })
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    removeParams (id) {
      this.$confirm('此操作将永久删除该商品分类, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http.delete(`categories/${this.cateId}/attributes/${id}`).then(res => {
          if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
          this.$message.success(res.data.meta.msg)
          this.getParamsList()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    handleInputConfirm (row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrValues(row)
    },
    showInput (row) {
      row.inputVisible = true
      // $nextTick 页面元素被重新渲染之后才会调用
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    handleClosed (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrValues(row)
    },
    // 将对attr_vals的操作发送请求
    saveAttrValues(row) {
      this.attrValues = row.attr_vals
      this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, { attr_name: row.attr_name, attr_sel: row.attr_sel, attr_vals: row.attr_vals.join(',') }).then(res => {
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.$message.success(res.data.meta.msg)
      })
    }
  },
  computed: {
    isBtnDisabled () {
      return this.selectedCateKeys.length !== 3
    },
    // 当前选中的三级分类的id
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 动态计算标题文本
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style scoped>
  .cat_opt {
    margin: 15px 0;
  }
  .el-tag {
    margin: 5px;
  }
  .input-new-tag {
    width: 120px;
  }
</style>
