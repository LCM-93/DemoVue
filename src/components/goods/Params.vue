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
          <el-button type="primary" size="mini" :disabled="disableBtn" @click="addDialogVisible = true">添加参数</el-button>

          <el-table :data="manyTabData" border style="width: 100%">
            <el-table-column type="expand">
              <template v-slot:default="scope">
                <el-tag v-for="(val,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)">
                  {{val}}
                </el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputEnable" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)"></el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column>
              <template v-slot:default="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="clickEdit(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteAttr(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="disableBtn">添加属性</el-button>
          <el-table :data="onlyTabData" border style="width: 100%">
            <el-table-column type="expand">
              <template v-slot:default="scope">
                <el-tag v-for="(val,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i, scope.row)">
                  {{val}}
                </el-tag>
                <el-input class="input-new-tag" v-if="scope.row.inputEnable" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)"></el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column>
              <template v-slot:default="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini" @click="clickEdit(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteAttr(scope.row)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- 添加参数对话框 -->
    <el-dialog :title=" '添加' + getTitleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form :model="addFrom" :rules="addFromRules" ref="addFromRef" label-width="100px">
        <el-form-item :label="getTitleText" prop="attr_name">
          <el-input v-model="addFrom.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑参数对话框 -->
    <el-dialog :title=" '修改' + getTitleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form :model="editFrom" :rules="addFromRules" ref="editFromRef" label-width="100px">
        <el-form-item :label="getTitleText" prop="attr_name">
          <el-input v-model="editFrom.attr_name"></el-input>
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
      onlyTabData: [],
      addDialogVisible: false,
      addFrom: {
        attr_name: ''
      },
      addFromRules: {
        attr_name: [
          { required: true, message: '请输入' + this.getTitleText, trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      editFrom: {
        attr_name: ''
      }
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
    },
    getTitleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
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
        item.inputEnable = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTabData = res.data
      } else {
        this.onlyTabData = res.data
      }
    },
    handleInputConfirm (item) {
      if (item.inputValue.trim() === '') {
        item.inputValue = ''
        item.inputEnable = false
        return
      }
      item.attr_vals.push(item.inputValue.trim())
      item.inputValue = ''
      item.inputEnable = false
      this.saveNewTag(item)
    },
    showInput (item) {
      item.inputEnable = true
      //   让输入框自动获取焦点
      // $nextTick方法的作用：当页面元素被重新渲染之后，才会至指定回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    async saveNewTag (item) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${item.attr_id}`, {
        attr_name: item.attr_name,
        attr_sel: item.attr_sel,
        attr_vals: item.attr_vals.join(' ')
      })
      console.log(res)
      if (res.meta.status !== 200) {
        item.attr_vals.pop()
        return this.$message.error('修改参数项失败')
      }
      this.$message.success('修改参数项成功！')
    },
    async deleteAttr (item) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该属性, 是否继续?',
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
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${item.attr_id}`)
      console.log(res)
      if (res.meta.status !== 200) {
        item.attr_vals.pop()
        return this.$message.error('删除属性失败')
      }
      this.$message.success('删除属性成功')
      this.queryParams()
    },
    addDialogClosed () {
      this.$refs.addFromRef.resetFields()
    },
    addParams () {
      this.$refs.addFromRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addFrom.attr_name,
          attr_sel: this.activeName
        })
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.addDialogVisible = false
        this.queryParams()
      })
    },
    async handleClose (i, item) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
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
      item.attr_vals.splice(i, 1)
      this.saveNewTag(item)
    },
    async clickEdit (id) {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${id}`, { attr_sel: this.activeName })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      this.editFrom = res.data
      this.editDialogVisible = true
    },
    editDialogClosed () {
      this.editFrom = {}
      this.$refs.editFromRef.resetFields()
    },
    async editParams () {
      this.$refs.editFromRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editFrom.attr_id}`,
          {
            attr_name: this.editFrom.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
        this.queryParams()
        this.editDialogVisible = false
      })
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
.input-new-tag {
  width: 90px;
}
.button-new-tag {
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
</style>
