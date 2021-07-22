<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区 -->
        <div class="avatar_box">
          <img src="../assets/logo.png"/>
        </div>

        <!-- 登录表单区 -->
        <el-form label-width="0px" class="login_form" :model="loginFrom" :rules="fromRules">
          <!-- 用户名 -->
          <el-form-item prop="username">
            <el-input placeholder="用户名" prefix-icon="iconfont icon-user" v-model="loginFrom.username" ></el-input>
          </el-form-item>
          <!-- 密码 -->
            <el-form-item prop="password">
            <el-input placeholder="密码" prefix-icon="iconfont icon-3702mima" v-model="loginFrom.password" show-password ></el-input>
          </el-form-item>
          <!-- 按钮区域 -->
          <el-form-item class="btns">
            <el-button type="primary">登录</el-button>
            <el-button type="info">重置</el-button>
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
          {required: true, message: '请输入用户名', trigger: 'blur'}
        ],
        password: [
          {required: true, message: '请输入密码', trigger: 'blur'},
          {min: 3, message: '密码不能少于3位', trigger: 'blur'}
        ]
      }
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box{
  width: 450px;
  height: 300px;
  background-color: white;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
  box-shadow: 0 0 10px #666;
}

.avatar_box{
  height: 130px;
  width: 130px;
  border: 1px solid #eee;
  border-radius: 50%;
  padding: 4px;
  box-shadow: 0 0 6px #ddd;
  position: absolute;
  left: 50%;
  transform: translate(-50%,-50%);
  background-color: white;
  img{
    width: 100%;
    height: 100%;
    border-radius: 50%;
    background-color: #eee;
  }
}

.login_form{
  width: 100%;
  position: absolute;
  bottom: 0;
  padding: 0 20px;
  box-sizing: border-box;
}

.btns{
  display: flex;
  justify-content: flex-end;
}
</style>>
