<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="15">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryParam.query" clearable @clear="clickQueryGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="clickQueryGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="clickToAddGoods">添加商品</el-button>
        </el-col>
      </el-row>

      <el-table :data="goodsList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column prop="goods_price" label="价格（元）" width="100px"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="100px"></el-table-column>
        <el-table-column prop="goods_number" label="商品数量" width="100px"></el-table-column>
        <el-table-column label="上架时间" width="140px">
          <template v-slot:default="scope">
            {{scope.row.add_time | dataFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="140px">
          <template v-slot:default="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteGoods(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryParam.pagenum" :page-sizes="[5, 10 ,20 ,50]" :page-size="queryParam.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data () {
    return {
      queryParam: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      goodsList: []
    }
  },
  created () {
    this.queryGoodsList()
  },
  methods: {
    async queryGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryParam })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('查询商品数据出错')
      }
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    clickQueryGoodsList () {
      this.queryParam.pagenum = 1
      this.queryGoodsList()
    },
    handleSizeChange (newSize) {
      this.queryParam.pagesize = newSize
      this.queryGoodsList()
    },
    handleCurrentChange (newIndex) {
      this.queryParam.pagenum = newIndex
      this.queryGoodsList()
    },
    async deleteGoods (goodsId) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(e => e)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(`goods/${goodsId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败')
      }
      this.$message.success('删除商品成功！')
      this.queryGoodsList()
    },
    clickToAddGoods () {
      this.$router.push('/add_goods')
    }
  }
}
</script>
<style lang="less" scoped>
</style>
