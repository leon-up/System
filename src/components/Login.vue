<template>
  <div class="login_container">
      <div class="login_box">
        <div class="avatar_box">
          <img src="../assets/logo.png" alt="">
        </div>
        <!-- 登录表单区域 -->
        <el-form label-width="0px" class="login_form" :model="LoginForm" :rules="loginFormRules" ref="loginFormRes">
          <!-- 用户名 -->
          <el-form-item prop="username">
          <el-input prefix-icon="iconfont icon-user" v-model="LoginForm.username"></el-input>
          </el-form-item>
          <!-- 密码 -->
          <el-form-item prop="password">
          <el-input prefix-icon="iconfont icon-3702mima" type="password" v-model="LoginForm.password"></el-input>
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
  data() {
    return {
      // 登录表单的数据对象
      LoginForm: {
      username:'admin',
      password:'123456'
    },
    loginFormRules: {
      username: [
        { required: true, message: '请输入登录名称', trigger: 'blur'},
        { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur'}
      ],
      password: [
        { required: true, message: '请输入登录密码', trigger: 'blur'},
        { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur'}
      ]
    }
    }
  },
   
    methods: {
      //重置按钮
      resetLoginForm() {
        //console.log(this)
        this.$refs.loginFormRes.resetFields()
    },
      //登录按钮的表单预验证
      login() {
        this.$refs.loginFormRes.validate(async valid => {
          if(!valid) return;
          const {data: res} = await this.$http.post("login", this.LoginForm)
          if(res.meta.status !== 200) return this.$message.error('登录失败!');
          this.$message.success('登录成功!')
          //将登录成功之后的 token 保存到客户端的 sessionStorage 中
          window.sessionStorage.setItem('token', res.data.token);
          //然后编程式导航跳转到后台主页，路由地址是 /home
          this.$router.push('/home')
        })
      }
  }
     
};
</script>

<style lang="less" scoped>
.login_container {
  height: 100%;
  background-color: #2b4b6b;
}
.login_box {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3pxx;
  transform: translate(-50%, -50%);

  .avatar_box {
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    height: 130px;
    width: 130px;
    padding: 10px;
    border: 1px solid #eee;
    border-radius: 50%;
    box-shadow: 0 0 10px #ddd;
    background-color: #fff;

    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}
.login_form {
  position: absolute;
  bottom: 0;
  box-sizing: border-box;
  width: 100%;
  padding: 10px;
}

.btns {
  display: flex;
  justify-content: flex-end;
}
</style>