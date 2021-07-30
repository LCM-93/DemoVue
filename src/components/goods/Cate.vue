<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      </el-row>

      <el-table :data="cateList" row-key="cat_id" lazy border :tree-props="treeProps">
        <!-- <el-table-column type="index" label="#"></el-table-column> -->
        <el-table-column prop="cat_name" label="分类名称">
        </el-table-column>
        <el-table-column label="是否有效">
          <template v-slot:default="scope">
            <i class="el-icon-error" style="color: red" v-if="scope.row.cat_deleted == true"></i>
            <i class="el-icon-success" style="color: lightgreen" v-if="scope.row.cat_deleted == false"></i>
          </template>
        </el-table-column>
        <el-table-column label="分类">
          <template v-slot:default="scope">
            <el-tag v-if="scope.row.cat_level == 0">一级</el-tag>
            <el-tag v-if="scope.row.cat_level == 1" type="success">二级</el-tag>
            <el-tag v-if="scope.row.cat_level == 2" type="warning">三级</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showModifyDialog(scope.row.cat_id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteCate(scope.row.cat_id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryParam.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryParam.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加分类的弹窗 -->
    <el-dialog title="添加分类" :visible.sync="addCateVisible" width="50%" @close="addCateDialogClose">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="80px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader v-model="parentCateSelectIds" :options="parentCateList" :props="cascaderProps" @change="handleCascaderChange" clearable filterable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改分类的弹窗 -->
    <el-dialog title="修改分类" :visible.sync="modifyCateVisible" width="50%" @close="modifyCateDialogClose">
      <el-form :model="modifyCateForm" :rules="addCateFormRules" ref="modifyCateFormRef" label-width="80px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="modifyCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="modifyCateVisible = false">取 消</el-button>
        <el-button type="primary" @click="modifyCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      queryParam: {
        type: 3,
        pagenum: 1,
        pagesize: 10
      },
      cateList: [],
      total: 0,
      treeProps: {
        children: 'children'
      },
      addCateVisible: false,
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      parentCateList: [],
      parentCateSelectIds: [],
      cascaderProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
        checkStrictly: true
      },
      modifyCateVisible: false,
      modifyCateForm: {
        cat_name: ''
      }
    }
  },
  created () {
    this.queryCateList()
  },
  methods: {
    // 查询分类列表
    async queryCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryParam })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.errror('获取分类数据失败')
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    // 监听添加分类弹窗关闭
    addCateDialogClose () {
      this.$refs.addCateFormRef.resetFields()
      this.parentCateSelectIds = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 添加分类
    addCate () {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        console.log(res)
        this.addCateVisible = false
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.queryCateList()
      })
    },
    // 点击展示添加分类弹窗
    async showAddCateDialog () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类失败！')
      }
      this.parentCateList = res.data
      this.addCateVisible = true
    },
    // 监听父级分类级联选择变化
    handleCascaderChange () {
      if (this.parentCateSelectIds.length > 0) {
        this.addCateForm.cat_pid = this.parentCateSelectIds[this.parentCateSelectIds.length - 1]
        this.addCateForm.cat_level = this.parentCateSelectIds.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 监听分页大小变化
    handleSizeChange (newSize) {
      this.queryParam.pagesize = newSize
      this.queryCateList()
    },
    // 监听页面标签
    handleCurrentChange (currentIndex) {
      this.queryParam.pagenum = currentIndex
      this.queryCateList()
    },
    // 展示修改分类信息弹窗
    async showModifyDialog (cateId) {
      const { data: res } = await this.$http.get(`categories/${cateId}`)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.errror('查询分类信息失败')
      }
      this.modifyCateForm = res.data
      this.modifyCateVisible = true
    },
    // 监听修改分类信息弹窗关闭
    modifyCateDialogClose () {
      this.$refs.modifyCateFormRef.resetFields()
      this.modifyCateForm = {}
    },
    // 修改分类信息
    modifyCate () {
      this.$refs.modifyCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.modifyCateForm.cat_id}`, { cat_name: this.modifyCateForm.cat_name })
        if (res.meta.status !== 200) {
          return this.$message.errror('修改分类信息失败')
        }
        this.$message.success('修改分类信息成功！')
        this.modifyCateVisible = false
        this.queryCateList()
      })
    },
    // 删除分类信息
    async deleteCate (cateId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该分类, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
      // 点击确定 返回值为：confirm
      // 点击取消 返回值为： cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(
        `categories/${cateId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除分类失败！')
      }
      this.$message.success('删除分类信息成功！')
      this.queryCateList()
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
