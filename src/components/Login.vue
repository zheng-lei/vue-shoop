<template>
  <div class="login-container">
      <div class="login-box">
          <div class="avatar-box">
              登录
          </div>
          <el-form class="login-form" :model="loginForm" :rules="loginFormRules" ref="loginFormRef">
            <el-form-item prop="username">
                <el-input prefix-icon="el-icon-user" v-model="loginForm.username"></el-input>
            </el-form-item>
            <el-form-item prop="password">
                <el-input type="password" prefix-icon="el-icon-lock" v-model="loginForm.password"></el-input>
            </el-form-item>
            <el-form-item class="btns">
                <el-button type="primary" @click="login">登录</el-button>
                <el-button type="info" @click="resertLoginForm">重置</el-button>
            </el-form-item>
          </el-form>
      </div>
  </div>
</template>

<script>
    export default {
        data(){
            return {
                //表单的数据绑定对象
                loginForm: {
                    username: 'admin',
                    password: '123456'
                },
                //表单验证规则
                loginFormRules: {
                    username: [
                        { required: true, message: '请输入登录名称', trigger: 'blur' },
                        { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                    ],
                    password: [
                        { required: true, message: '请输入登录密码', trigger: 'blur' },
                        { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                    ]
                }
            }
        },
        methods:{
            //重置登录表单
            resertLoginForm(){
                this.$refs.loginFormRef.resetFields();
            },
            login(){
                const $this = this;
                this.$refs.loginFormRef.validate((valid) => {
                    if(!valid) return;
                    this.$http.post('login', this.loginForm).then(function(res){
                        console.log(res)
                        if(res.data.meta.status != 200) return $this.$message.error('登陆失败！');
                        $this.$message.success("登录成功！");
                        //保存token
                        window.sessionStorage.setItem('token', res.data.data.token);
                        //登录后跳转
                        $this.$router.push('/home');
                    })
                    
                });
            }
        }
    }
</script>

<style lang="less" scoped>
    .login-container{
        background: #2b4b6b;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .login-box{
        width: 450px;
        height: 300px;
        background: #fff;
        border-radius: 3px;
        position: relative;
        // position: absolute;
        // top: 50%;
        // left: 50%;
        // transform: translate(-50%, -50%);
    }
    .avatar-box{
        margin: 40px 30px;
        font-size: 24px;
    }
    .btns{
        display: flex;
        justify-content: flex-end;
    }
    .login-form{
        position: absolute;
        bottom: 0;
        width: 100%;
        padding: 0 30px;
        box-sizing: border-box;
    }
</style>