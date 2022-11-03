<template>
  <div class="order">
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input @clear="getOrderList" :clearable="true" v-model="queryInfo.query" placeholder="请输入内容">
            <el-button @click="getOrderList" slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!--表格区域-->
      <el-table :data="orderList" :border="true" :stripe="true" height="90%">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column width="100px" label="订单价格(元)" prop="order_price"></el-table-column>
        <el-table-column width="100px" label="是否付款">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.pay_status === '1'" type="success">已付款</el-tag>
            <el-tag v-else type="danger">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column width="70px" label="是否发货" prop="is_send"></el-table-column>
        <el-table-column width="140px" label="下单时间">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column width="130px" label="操作">
          <template slot-scope="">
            <el-button @click="showEditDialog" size="mini" type="primary" icon="el-icon-edit"></el-button>
            <el-button @click="showProgressDialog" size="mini" type="success" icon="el-icon-location"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--分页区域-->
      <el-pagination
        :background="true"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 50]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
      <!--修改地址的对话框-->
      <el-dialog
        @close="editAddressDialogClosed"
        title="修改地址"
        :visible.sync="editAddressDialogVisible"
        width="50%">
        <el-form :model="editAddressForm" :rules="addAddressFormRules" ref="editAddressFormRef" label-width="100px">
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader :props="{ expandTrigger: 'hover' }" :clearable="true" :options="cityData" v-model="editAddressForm.address1"></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="editAddressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editAddressDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editAddress">确 定</el-button>
        </span>
      </el-dialog>
      <!--展示物流进度的对话框-->
      <el-dialog
        title="物流进度"
        :visible.sync="progressDialogVisible"
        width="50%">
        <el-timeline>
          <el-timeline-item
            v-for="(activity, index) in progressData"
            :key="index"
            :timestamp="activity.time">
            {{activity.context}}
          </el-timeline-item>
        </el-timeline>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
import cityData from './citydata'
export default {
  name: 'Order',
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderList: [],
      editAddressDialogVisible: false,
      editAddressForm: {
        address1: [],
        address2: ''
      },
      addAddressFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData: cityData,
      progressDialogVisible: false,
      progressData: []
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    getOrderList () {
      this.$http.get('orders', { params: this.queryInfo }).then(res => {
        // console.log(res.data)
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.orderList = res.data.data.goods
        this.total = res.data.data.total
      })
    },
    handleSizeChange (newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange (newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    showEditDialog () {
      this.editAddressDialogVisible = true
    },
    editAddress () {},
    editAddressDialogClosed () {
      this.$refs.editAddressFormRef.resetFields()
    },
    showProgressDialog () {
      this.$http.get('/kuaidi/1106975712662').then(res => {
        if (res.data.meta.status !== 200) return this.$message.error(res.data.meta.msg)
        this.progressData = res.data.data
      })
      this.progressDialogVisible = true
    }
  }
}
</script>

<style lang="less">
  .el-cascader {
    width: 100%;
  }
  .order {
    height: 95%;
    .el-card {
      height: 100%;
    }
    .el-card__body {
      height: 90%;
    }
  }
</style>
