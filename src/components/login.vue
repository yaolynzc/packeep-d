<template>
  <div :style="backgroundDiv" class="loginwrap">
  <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-position="left" label-width="0px" class="demo-ruleForm login-container">
    <h3 class="title">快递托管信息平台</h3>
    <el-form-item prop="id">
      <el-input
        type="text"
        v-model="ruleForm.id"
        auto-complete="off"
        @keyup.enter.native="getFocus('inputPassword')"
        placeholder="账号"></el-input>
    </el-form-item>
    <el-form-item prop="pass">
      <el-input
        type="password"
        v-model="ruleForm.pass"
        ref="inputPassword"
        auto-complete="off"
        @keyup.enter.native="submitForm('ruleForm')"
        placeholder="密码"></el-input>
    </el-form-item>
    <el-checkbox v-model="checked" checked class="remember">下次自动登录</el-checkbox>
    <!--<span>没有账号？<a>注册</a></span>-->
    <el-form-item style="width:100%;">
      <el-button type="primary" style="width:100%;" @click.native.prevent="submitForm('ruleForm')" :loading="logining">登录</el-button>
    </el-form-item>
  </el-form>
  </div>
</template>

<script>
import md5 from 'md5'
export default {
  name: 'login',
  data () {
    return {
      backgroundDiv: {
        backgroundImage: 'url(' + require('../assets/img/backimg.jpg') + ')'
      },
      logining: false,
      ruleForm: {
        id: '',
        pass: ''
      },
      rules: {
        id: [
          {required: true, message: '请输入账户名或手机号', trigger: 'blur'}
        ],
        pass: [
          {required: true, message: '请输入密码', trigger: 'blur'}
        ]
      },
      checked: true
    }
  },
  created () {
    this.naviMethod()
  },
  methods: {
    naviMethod () {
      if (this.checkLogin()) {
        this.$router.push('/index')
      }
    },
    submitForm (formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          let uid = this.ruleForm.id
          let upass = md5(this.ruleForm.pass)
          // console.log(uid + '-' + upass)
          let obj = {
            pwd: upass
          }
          let that = this
          this.$http.get('http://211.159.218.41/api/user/' + uid, {params: obj})
            .then(function (res) {
              if (res.data.success) {
                that.$message({
                  message: '登录成功！',
                  type: 'success'
                })
                // 记住密码，每次自动登录
                if (that.checked) {
                  localStorage.setItem('uid', uid)
                  localStorage.setItem('upass', upass)
                } else {
                  // 否则，每次手动登录
                  sessionStorage.setItem('uid', uid)
                  sessionStorage.setItem('upass', upass)
                }
                // 导航到首页
                that.$router.push('/index')
              } else {
                that.$message({
                  message: '用户名或密码错误！',
                  type: 'error'
                })
              }
            })
            .catch(function (err) {
              console.log(err.message)
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    // 获取光标
    getFocus (inputname) {
      this.$refs[inputname].focus()
    },
    // 检测是否登录
    checkLogin () {
      let res = false
      let localUid = localStorage.getItem('uid')
      let sessionUid = sessionStorage.getItem('uid')
      // console.log(sessionUid + '/' + localUid)
      if (localUid || sessionUid) {
        res = true
      }
      return res
    }
  }
}
</script>

<style scoped>
  .loginwrap {
    /*background-color: #2c3e50;*/
    background-size: cover;
    /*background-image: url('~@/assets/img/backimg.jpg');*/
    height: 100%;
    width: 100%;
    position: fixed;
    top:0px;
    left:0px;
    /*position:absolute; left:0px; top:0px;*/
  }
  .login-container {
    /*box-shadow: 0 0px 8px 0 rgba(0, 0, 0, 0.06), 0 1px 0px 0 rgba(0, 0, 0, 0.02);*/
    -webkit-border-radius: 5px;
    border-radius: 5px;
    -moz-border-radius: 5px;
    background-clip: padding-box;
    margin: 180px auto;
    width: 350px;
    padding: 35px 35px 15px 35px;
    border: 1px solid #eaeaea;
    box-shadow: 0 0 25px #cac6c6;
    background-color: #ffffff;
  }
  .title {
    margin: 0px auto 40px auto;
    text-align: center;
    color: #505458;
  }
  .remember {
    margin: 0px 0px 35px 0px;
  }
</style>
