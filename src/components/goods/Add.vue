<template>
    <div>
      <!--面包屑导航区域-->
      <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>添加商品</el-breadcrumb-item>
      </el-breadcrumb>
      <!--卡片视图区域-->
      <el-card>
        <el-alert
          :closable="false"
          title="添加商品信息"
          type="info"
          center
          show-icon>
        </el-alert>
        <!--步骤条区域-->
        <el-steps :align-center="true" :space="200" :active="+activeIndex" finish-status="success">
          <el-step title="基本信息"></el-step>
          <el-step title="商品参数"></el-step>
          <el-step title="商品属性"></el-step>
          <el-step title="商品图片"></el-step>
          <el-step title="商品内容"></el-step>
          <el-step title="完成"></el-step>
        </el-steps>
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
          <!--tab栏区域-->
          <el-tabs @tab-click="tabClicked" :before-leave="beforeTabLeave" v-model="activeIndex" :tab-position="'left'">
            <el-tab-pane name="0" label="基本信息">
              <el-form-item label="商品名称" prop="goods_name"><el-input v-model="addForm.goods_name"></el-input></el-form-item>
              <el-form-item label="商品价格" prop="goods_price"><el-input type="number" v-model="addForm.goods_price"></el-input></el-form-item>
              <el-form-item label="商品重量" prop="goods_weight"><el-input type="number"  v-model="addForm.goods_weight"></el-input></el-form-item>
              <el-form-item label="商品数量" prop="goods_number"><el-input type="number"  v-model="addForm.goods_number"></el-input></el-form-item>
              <!--选择商品分类区域-->
              <el-form-item label="商品分类" prop="goods_cat">
                <el-cascader
                  v-model="addForm.goods_cat"
                  :options="categoriesList"
                  :props="{ expandTrigger: 'hover', label: 'cat_name',value: 'cat_id', children: 'children' }"
                  @change="handleCateChange">
                </el-cascader>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane name="1" label="商品参数">
              <el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
                <el-checkbox-group v-model="item.attr_vals">
                  <el-checkbox :border="true" v-for="(cb, i) in item.attr_vals" :key="i" :label="cb"></el-checkbox>
                </el-checkbox-group>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane name="2" label="商品属性">
              <el-form-item v-for="item in onlyTableData" :key="item.attr_id" :label="item.attr_name">
                <el-input v-model="item.attr_vals"></el-input>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane name="3" label="商品图片">
              <el-upload
                :on-success="handleSuccess"
                :headers="headerObj"
                action="https://www.liulongbin.top:8888/api/private/v1/upload"
                :on-preview="handlePreview"
                :on-remove="handleRemove"
                list-type="picture">
                <el-button size="small" type="primary">点击上传</el-button>
                <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
              </el-upload>
            </el-tab-pane>
            <el-tab-pane name="4" label="商品内容">
              <!--富文本编辑器-->
              <quill-editor v-model="addForm.goods_introduce"></quill-editor>
              <el-button @click="addGoods" class="btn-add" type="primary">添加商品</el-button>
            </el-tab-pane>
          </el-tabs>
        </el-form>
        <!--图片预览-->
        <el-dialog
          title="图片预览"
          :visible.sync="preDialogVisible"
          width="50%">
          <img :src="prePath" alt="" class="prev-img">
        </el-dialog>
      </el-card>
    </div>
</template>

<script>
import _ from 'lodash'
export default {
  name: 'Add',
  data () {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      categoriesList: [],
      manyTableData: [],
      onlyTableData: [],
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      prePath: '',
      preDialogVisible: false
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
    handleCateChange () {
      // 不是三级分类无法选中
      if (this.addForm.goods_cat.length !== 3) this.addForm.goods_cat = []
    },
    beforeTabLeave (activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类！')
        return false
      }
    },
    tabClicked () {
      if (this.activeIndex === '1') {
        this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } }).then(res => {
          // console.log(res.data)
          if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
          res.data.data.forEach(item => {
            item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(',')
          })
          this.manyTableData = res.data.data
        })
      } else if (this.activeIndex === '2') {
        this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } }).then(res => {
          // console.log(res.data)
          if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
          this.onlyTableData = res.data.data
        })
      }
    },
    handlePreview (file) {
      this.prePath = file.response.data.url
      this.preDialogVisible = true
    },
    handleRemove (file) {
      console.log(file)
      const filePath = file.response.data.tmp_path
      const i = this.addForm.pics.findIndex(item => {
        return item.pic === filePath
      })
      this.addForm.pics.splice(i, 1)
    },
    handleSuccess (response) {
      // console.log(response)
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    addGoods () {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return this.$message.error('请填写必要的表单项')
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        this.manyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(',') }
          this.addForm.attrs.push(newInfo)
        })
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        console.log(form)
        this.$http.post('goods', form).then(res => {
          if (res.data.meta.status !== 201) return this.$message.error(res.data.meta.msg)
          this.$message.success(res.data.meta.msg)
          this.$router.push('/goods')
        })
      })
    }
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
  .el-checkbox {
    margin-right: 10px !important;
  }
  .prev-img {
    width: 100%;
  }
  .btn-add {
    margin-top: 10px;
  }
</style>
