<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-button type="primary" @click="showAddModifyDialog(true)">添加角色</el-button>
      </el-row>

      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template v-slot:default="scope">
            <el-row v-for="(item1,i1) in scope.row.children" :key="item1.id" :class="[i1===0?'bdTop':'','bdBottom','vcenter']">
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row v-for="(item2,i2) in item1.children" :key="item2.id" :class="[i2 !==0? 'bdTop':'','vcenter']">
                  <el-col :span="8">
                    <el-tag closable type="success" @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="16">
                    <el-tag v-for="(item3) in item2.children" :key="item3.id" closable type="warning" @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>{{scope.row}}</pre> -->
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template v-slot:default="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="modifyRole(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteRole(scope.row.id)"></el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加修改角色的弹窗 -->
    <el-dialog :title="isAddRoleMode?'添加角色':'修改角色'" :visible.sync="addModifyRoleVisible" width="50%" @close="addModifyRoleDialogClose">
      <el-form :model="addModifyForm" :rules="addModifyFormRules" ref="addModifyFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addModifyForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述">
          <el-input v-model="addModifyForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addModifyRoleVisible = false">取 消</el-button>
        <el-button type="primary" @click="addModifyRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 添加分配权限的弹窗 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClose">
      <el-tree :data="rightList" :props="setRightProps" ref="rightTree" show-checkbox node-key="id" default-expand-all :default-checked-keys="defaultKey"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="setRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data () {
    return {
      rolesList: [],
      isAddRoleMode: true,
      addModifyRoleVisible: false,
      addModifyForm: {
        roleName: '',
        roleDesc: ''
      },
      addModifyFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' }
        ]
      },
      setRightDialogVisible: false,
      setRightRoleId: '',
      rightList: [],
      setRightProps: {
        children: 'children',
        label: 'authName'
      },
      defaultKey: []
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取角色列表
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolesList = res.data
    },
    // 移除角色下面的权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该权限, 是否继续?',
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
        `roles/${role.id}/rights/${rightId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      role.children = res.data
    },
    // 展示添加修改角色dialog
    showAddModifyDialog (isAddMode) {
      this.isAddRoleMode = isAddMode
      if (isAddMode) {
        this.addModifyForm = {
          roleName: '',
          roleDesc: ''
        }
      }
      this.addModifyRoleVisible = true
    },
    // 监听添加修改角色dialog关闭
    addModifyRoleDialogClose () {
      this.$refs.addModifyFormRef.resetFields()
    },
    // 添加或修改角色
    addModifyRole () {
      this.$refs.addModifyFormRef.validate(async valid => {
        console.log('校验结果：' + valid)
        if (!valid) return
        if (this.isAddRoleMode) {
          const { data: res } = await this.$http.post('roles', this.addModifyForm)
          console.log(res)
          this.addModifyRoleVisible = false
          if (res.meta.status !== 201) {
            return this.$message.error('添加角色失败！')
          }
          this.$message.success('添加角色成功!')
        } else {
          const { data: res } = await this.$http.put(`roles/${this.addModifyForm.roleId}`, this.addModifyForm)
          console.log(res)
          this.addModifyRoleVisible = false
          if (res.meta.status !== 200) {
            return this.$message.error('修改角色失败！')
          }
          this.$message.success('修改角色成功!')
        }
        this.$refs.addModifyFormRef.resetFields()
        this.getRolesList()
      })
    },
    // 点击修改角色
    async modifyRole (roleId) {
      const { data: res } = await this.$http.get(`roles/${roleId}`)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败！')
      }
      this.addModifyForm = res.data
      this.showAddModifyDialog(false)
    },
    // 点击删除角色
    async deleteRole (roleId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
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
        `roles/${roleId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.getRolesList()
    },
    // 监听设置权限dialog关闭
    setRightDialogClose () {
      this.defaultKey = []
    },
    // 设置权限
    async setRight () {
      const keys = [
        ...this.$refs.rightTree.getCheckedKeys(),
        ...this.$refs.rightTree.getHalfCheckedKeys()
      ]
      const keyStr = keys.join(',')
      console.log(keyStr)
      const { data: res } = await this.$http.post(`roles/${this.setRightRoleId}/rights`, { rids: keyStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')
      this.setRightDialogVisible = false
      this.getRolesList()
    },
    // 点击设置权限
    async showSetRightDialog (role) {
      this.setRightRoleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      this.rightList = res.data
      console.log(this.rightList)
      this.getNodeCheckIds(role, this.defaultKey)
      this.setRightDialogVisible = true
    },
    getNodeCheckIds (node, arry) {
      if (!node.children) {
        return arry.push(node.id)
      }
      node.children.forEach(item => {
        this.getNodeCheckIds(item, arry)
      })
    }
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdTop {
  border-top: 1px solid #eee;
}
.bdBottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
