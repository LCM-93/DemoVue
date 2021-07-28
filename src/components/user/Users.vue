<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="15">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryParam.query" clearable @clear="queryUserList">
            <el-button slot="append" icon="el-icon-search" @click="queryUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addUserVisible = true">添加用户</el-button>
        </el-col>
      </el-row>

      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column label="状态">
          <template v-slot:default="slotProps">
            <el-switch v-model="slotProps.row.mg_state" @change="changeUserStatus(slotProps.row)">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="slot">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showModifyDialog(slot.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="showDeleteDialog(slot.row.id)"></el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryParam.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryParam.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加用户的弹窗 -->
    <el-dialog title="添加用户" :visible.sync="addUserVisible" width="50%" @close="addDialogClose">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" show-password></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addUserVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户弹窗 -->
    <el-dialog title="修改用户信息" :visible.sync="modifyUserVisible" width="50%" @close="modifyDialogClose">
      <el-form :model="modifyForm" :rules="addFormRules" ref="modifyFormRef" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="modifyForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="modifyForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="modifyForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="modifyUserVisible = false">取 消</el-button>
        <el-button type="primary" @click="modifyUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    // 自定义校验规则
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^\w+@\w+(\.\w+)+$/
      if (regEmail.test(value)) {
        return callback()
      }
      return callback(new Error('请输入合法的邮箱'))
    }

    var checkMobile = (rule, value, callback) => {
      const regMobile = /^1[34578]\d{9}$/
      if (regMobile.test(value)) {
        return callback()
      }
      return callback(new Error('请输入合法的手机号'))
    }

    return {
      queryParam: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      userList: [],
      addUserVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 2, max: 10, message: '用户名长度为2~10位', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '密码长度为6~15位', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      modifyUserVisible: false,
      modifyForm: {}
    }
  },
  created () {
    this.queryUserList()
  },
  methods: {
    // 查询用户列表
    async queryUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryParam })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 处理查询pageSize变化
    handleSizeChange (newSize) {
      this.queryParam.pagesize = newSize
      this.queryUserList()
    },
    // 处理查询pageNum变化
    handleCurrentChange (newIndex) {
      this.queryParam.pagenum = newIndex
      this.queryUserList()
    },
    // 修改用户状态
    async changeUserStatus (userInfo) {
      const { data: res } = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('更新用户状态失败！')
      this.$message.success('更新用户状态成功！')
    },
    // 添加新用户
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        console.log('校验结果：' + valid)
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        console.log(res)
        this.addUserVisible = false
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        this.$refs.addFormRef.resetFields()
        this.queryUserList()
      })
    },
    // 监听添加用户弹窗关闭
    addDialogClose () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击展示修改用户弹窗
    async showModifyDialog (userid) {
      const { data: res } = await this.$http.get(`users/${userid}`)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户数据失败')
      }
      this.modifyForm = res.data
      this.modifyUserVisible = true
    },
    // 修改用户信息
    modifyUser () {
      this.$refs.modifyFormRef.validate(async valid => {
        console.log('校验结果：' + valid)
        if (!valid) return
        const { data: res } = await this.$http.put(`users/${this.modifyForm.id}`, { email: this.modifyForm.email, mobile: this.modifyForm.mobile })
        console.log(res)
        this.modifyUserVisible = false
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败！')
        }
        this.$message.success('更新用户信息成功!')
        this.$refs.modifyFormRef.resetFields()
        this.queryUserList()
      })
    },
    // 监听修改用户弹窗关闭
    modifyDialogClose () {
      this.modifyForm = {}
      this.$refs.modifyFormRef.resetFields()
    },
    // 删除用户
    showDeleteDialog (userid) {
      this.$confirm('此操作将永久删除用户，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`users/${userid}`)
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('删除用户失败！')
        }
        this.$message.success('删除用户成功!')
        this.queryUserList()
      }).catch(() => {
        this.$message.info('取消删除用户!')
      })
    }
  }
}
</script>
<style lang="less" scoped>
</style>
