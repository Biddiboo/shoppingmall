<template>
    <div class="login_container">
        <div class="login_box">
            <!-- 头像区域 -->
            <h3 class="box_title">用户登录</h3>
            <!-- 登录表单区域 -->
            <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
                <!-- 用户名 -->
                <el-form-item prop="username">
                    <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user" placeholder="会员名/邮箱/手机号"></el-input>
                </el-form-item>
                <!-- 密码 -->
                <el-form-item prop="password">
                    <el-input type="password" v-model="loginForm.password" prefix-icon="iconfont icon-3702mima" placeholder="请输入登录密码"></el-input>
                </el-form-item>
                <!-- 按钮区域 -->
                <el-form-item>
                    <el-button type="primary" class="login_btn" @click="login">登 录</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: '',
        password: ''
      },

      loginFormRules: {
        username: [
          { required: true, message: '请输入账号', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) {
          return this.$message.error('登陆失败！')
        }
        this.$message.success('登陆success！')
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container{
    height: 100%;
    background-color: #f4abb6;
}

.box_title{
    color: #f45d6d;
    padding: 10px 30px;
}

.login_box{
    width: 300px;
    height: 300px;
    background-color: white;
    border-radius: 10px;
    position: absolute;
    right: 10%;
    top: 50%;
    transform: translate(0,-50%);
}

.login_form{
    width: 100%;
    padding: 10px 30px;
    box-sizing: border-box;
}

.login_form el-form-item{
    border-radius: 0px;
}

.login_btn{
    background-color: #FF6473;
    border: 0px;
    width: 100%;
    font-size: 15px;
    font-weight: bold;
    border-radius: 0px;
}

.login_btn:hover{
    background-color: #f45d6d;
}

.login_btn:active{
    background-color: #f45d6d;
}

.login_btn:focus{
    background-color: #FF6473;
}
</style>
