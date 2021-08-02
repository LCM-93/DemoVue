<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 提示信息 -->
      <el-alert title="注意：只能为第三级分类添加属性" type="warning" show-icon :closable="false"></el-alert>

      <!-- 选择分类 -->
      <el-row>
        <el-col>
          <span>选择属性：</span>
          <el-cascader v-model="selectCateId" :options="cateList" :props="cascaderProps" @change="handleCascaderChange"></el-cascader>
        </el-col>
      </el-row>

      <el-tabs v-model="activeName" @tab-click="handleTabClick" type="card">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="disableBtn">添加参数</el-button>

          <el-table :data="manyTabData" border style="width: 100%">
            <el-table-column type="expand">
              <template v-slot:default="scope">
                <el-tag v-for="(val,i) in scope.row.attr_vals" :key="i" closable>
                  {{val}}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column>
              <template v-slot:default="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="disableBtn">添加属性</el-button>
          <el-table :data="onlyTabData" border style="width: 100%">
            <el-table-column type="expand">
              <template v-slot:default="scope">
                <el-tag v-for="(val,i) in scope.row.attr_vals" :key="i" closable>
                  {{val}}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column>
              <template v-slot:default="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
  </div>
</template>
<script>
export default {
  data () {
    return {
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      cateList: [],
      selectCateId: [],
      activeName: 'many',
      manyTabData: [],
      onlyTabData: []
    }
  },
  created () {
    this.queryCateList()
  },
  computed: {
    disableBtn () {
      if (this.selectCateId.length === 3) {
        return false
      }
      return true
    },
    cateId () {
      if (this.selectCateId.length === 3) {
        return this.selectCateId[2]
      }
      return null
    }
  },
  methods: {
    async queryCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 3 } })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.errror('获取分类数据失败')
      }
      this.cateList = res.data
    },
    handleCascaderChange () {
      this.queryParams()
    },
    handleTabClick (tab, event) {
      console.log(this.activeName)
      this.queryParams()
    },
    async queryParams () {
      // 通过数组的长度判断
      if (this.selectCateId.length !== 3) {
        this.selectCateId = []
        // 清空表格数据
        this.manyTabData = []
        this.onlyTabData = []
        return
      }
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.errror('查询属性失败！')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
      })
      if (this.activeName === 'many') {
        this.manyTabData = res.data
      } else {
        this.onlyTabData = res.data
      }
    }
  }
}
</script>
<style lang="less" scoped>
.el-alert {
  margin-bottom: 15px;
}
.el-tabs {
  margin-top: 15px;
}
.el-tag {
  margin: 5px;
}
</style>
