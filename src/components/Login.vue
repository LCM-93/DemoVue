<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" />
      </div>

      <!-- 登录表单区 -->
      <el-form label-width="0px" ref="loginFormRef" class="login_form" :model="loginFrom" :rules="fromRules">
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input placeholder="用户名" prefix-icon="iconfont icon-user" v-model="loginFrom.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input placeholder="密码" prefix-icon="iconfont icon-3702mima" v-model="loginFrom.password" show-password></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      loginFrom: {
        username: '',
        password: ''
      },
      fromRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, message: '密码不能少于3位', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(valid => {
        console.log('校验结果：' + valid)
        if (!valid) return
        // 使用 async\await 使得异步代码更像同步代码
        // 结果解构
        // const {data: res} = await this.$http.post('login', this.loginFrom)
        // console.log('登录返回值：' + JSON.stringify(res))
        // if (res.meta.status !== 200) return console.log('登录失败')
        // console.log('登录成功！')

        // 箭头函数中可以直接获取到外部this对象
        const that = this
        this.$http.post('login', this.loginFrom)
          .then(({ data: res }) => {
            console.log(this)
            console.log('登录返回值：' + JSON.stringify(res))
            if (res.meta.status !== 200) return this.$message.error('登录失败！')
            this.$message.success('登录成功！')
            this.handleLoginSuccess(res)
          })
          .then(function (error) {
            console.log(error)
            that.$message.error(error)
          })
      })
    },
    handleLoginSuccess (data) {
      // 1、将登录成功之后的Token保存到本地客户端的sessionStorage中
      //   1.1 项目中除了登录之外的其他接口必须在登录之后才能访问
      //   1.2 Token只应在当前网站打开期间有效，所以将Token保存在SessionStorage中
      window.sessionStorage.setItem('token', data.data.token)
      // 2、通过编程式导航跳转到后台主页，路由地址是 /home
      this.$router.push('/home')
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box {
  width: 450px;
  height: 300px;
  background-color: white;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 0 10px #666;
}

.avatar_box {
  height: 130px;
  width: 130px;
  border: 1px solid #eee;
  border-radius: 50%;
  padding: 4px;
  box-shadow: 0 0 6px #ddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}

.login_form {
  width: 100%;
  position: absolute;
  bottom: 0;
  padding: 0 20px;
  box-sizing: border-box;
}

.btns {
  display: flex;
  justify-content: flex-end;
}
</style>>
