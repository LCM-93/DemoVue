<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="15">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryParam.query" clearable @clear="clickToSearch">
            <el-button slot="append" icon="el-icon-search" @click="clickToSearch"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
        </el-col>
      </el-row>
      <el-table :data="orderList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="order_number" label="订单编号"></el-table-column>
        <el-table-column prop="order_price" label="订单价格"></el-table-column>
        <el-table-column label="是否付款">
          <template v-slot:default="scope">
            <el-tag type="success" v-if="scope.row.pay_status==1" size="small">已付款</el-tag>
            <el-tag type="danger" v-else size="small">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send" label="是否发货"></el-table-column>
        <el-table-column label="下单时间">
          <template v-slot:default="scope">
            {{scope.row.create_time | dataFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
          <el-button type="success" icon="el-icon-location" size="mini" @click="showTimeline"></el-button>
        </el-table-column>
      </el-table>

      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryParam.pagenum" :page-sizes="[5, 10, 20, 50]" :page-size="queryParam.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 物流信息弹窗 -->
    <el-dialog title="物流信息" :visible.sync="timelineVisible" width="50%" @close="timelineDialogClose">
      <el-timeline>
        <el-timeline-item v-for="(activity, index) in logisticList" :key="index" :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      queryParam: {
        query: '',
        pagesize: 10,
        pagenum: 1
      },
      total: 0,
      orderList: [],
      timelineVisible: false,
      logisticList: []
    }
  },
  created () {
    this.queryOrderList()
  },
  methods: {
    async queryOrderList () {
      const { data: res } = await this.$http.get('/orders', { params: this.queryParam })
      if (res.meta.status !== 200) {
        this.$message.error('查询订单列表出错')
        return
      }
      this.orderList = res.data.goods
      this.total = res.data.total
      console.log(this.orderList)
    },
    clickToSearch () {
      this.queryParam.pagenum = 1
      this.queryOrderList()
    },
    // 处理查询pageSize变化
    handleSizeChange (newSize) {
      this.queryParam.pagesize = newSize
      this.queryOrderList()
    },
    // 处理查询pageNum变化
    handleCurrentChange (newIndex) {
      this.queryParam.pagenum = newIndex
      this.queryOrderList()
    },
    async showTimeline () {
      // const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      // if (res.meta.status !== 200) {
      //   return this.$message.error('查询物流信息失败')
      // }
      // this.logisticList = res.data
      this.logisticList = [
        {
          'time': '2018-05-10 09:39:00',
          'ftime': '2018-05-10 09:39:00',
          'context': '已签收,感谢使用顺丰,期待再次为您服务',
          'location': ''
        },
        {
          'time': '2018-05-10 08:23:00',
          'ftime': '2018-05-10 08:23:00',
          'context': '[北京市]北京海淀育新小区营业点派件员 顺丰速运 95338正在为您派件',
          'location': ''
        },
        {
          'time': '2018-05-10 07:32:00',
          'ftime': '2018-05-10 07:32:00',
          'context': '快件到达 [北京海淀育新小区营业点]',
          'location': ''
        },
        {
          'time': '2018-05-10 02:03:00',
          'ftime': '2018-05-10 02:03:00',
          'context': '快件在[北京顺义集散中心]已装车,准备发往 [北京海淀育新小区营业点]',
          'location': ''
        },
        {
          'time': '2018-05-09 23:05:00',
          'ftime': '2018-05-09 23:05:00',
          'context': '快件到达 [北京顺义集散中心]',
          'location': ''
        },
        {
          'time': '2018-05-09 21:21:00',
          'ftime': '2018-05-09 21:21:00',
          'context': '快件在[北京宝胜营业点]已装车,准备发往 [北京顺义集散中心]',
          'location': ''
        },
        {
          'time': '2018-05-09 13:07:00',
          'ftime': '2018-05-09 13:07:00',
          'context': '顺丰速运 已收取快件',
          'location': ''
        },
        {
          'time': '2018-05-09 12:25:03',
          'ftime': '2018-05-09 12:25:03',
          'context': '卖家发货',
          'location': ''
        },
        {
          'time': '2018-05-09 12:22:24',
          'ftime': '2018-05-09 12:22:24',
          'context': '您的订单将由HLA（北京海淀区清河中街店）门店安排发货。',
          'location': ''
        },
        {
          'time': '2018-05-08 21:36:04',
          'ftime': '2018-05-08 21:36:04',
          'context': '商品已经下单',
          'location': ''
        }
      ]
      this.timelineVisible = true
    },
    timelineDialogClose () {
      // this.logisticList = []
      this.timelineVisible = false
    }
  }
}
</script>
<style lang="less" scoped>
</style>
