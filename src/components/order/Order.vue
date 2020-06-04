<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="6">
            <el-input placeholder="请输入内容" v-model="queryInfo.query" class="input-with-select">
                <el-button slot="append" icon="el-icon-search" @click="getOrderList"></el-button>
            </el-input>
        </el-col>
      </el-row>
      <el-table :data="orderlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price" width="120px"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status" width="80px">
          <template slot-scope="scope">
              <el-tag v-if="scope.row.pay_status === '1'" type="success">已付款</el-tag>
              <el-tag v-else type="danger">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send" width="80px"></el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="200px">
          <template slot-scope="scope">{{scope.row.create_time | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="120px">
          <template>
            <el-button
              type="danger"
              icon="el-icon-edit"
              size="mini"
              @click="editDialogVisible = true"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <el-dialog
      title="修改地址"
      :visible.sync="editDialogVisible"
      width="50%" @close="editDialogClosed">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
          <el-form-item label="省市区/县" prop="address1" label-width="100px">
            <el-cascader :options="cityData" v-model="editFormRules.address1"></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2" label-width="100px">
            <el-input v-model="editForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editDialogVisible = false">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog
      title="查看物流"
      :visible.sync="progressVisible"
      width="50%" @close="progressVisible= false">
        <el-timeline>
          <el-timeline-item
            v-for="(message, index) in progressInfo"
            :key="index"
            :timestamp="message.time">
            {{message.context}}
          </el-timeline-item>
        </el-timeline>
        <span slot="footer" class="dialog-footer">
          <el-button @click="progressVisible= false">取 消</el-button>
          <el-button type="primary" @click="progressVisible= false">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>
<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      editDialogVisible: false,
      editForm: {
        address1: '',
        address2: ''
      },
      editFormRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请填写详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressVisible: false,
      progressInfo: {}
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    async getOrderList () {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('编辑用户信息失败！')
      }
      this.orderlist = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    async showProgressBox () {
      this.progressVisible = true
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      this.progressInfo = res.data
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
  margin: 0px;
}
</style>
