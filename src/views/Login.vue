<template>
  <div class="login_container">
    <div class="login_box">
    	<!-- 头像区域 -->
    	<div class="avatar_box">
    		<img src="../assets/logo.png">
    	</div>
    	<!-- 登陆表单区域     ref用来后面获取实例对象-->
    	<el-form label-width="0px" class="login_form" :model="loginForm" :rules="loginFormRules" ref="loginFormRef">
    	  <!-- 用户名 -->
		  <el-form-item prop="username">
		    <el-input prefix-icon="iconfont icon-user" v-model="loginForm.username"></el-input>
		  </el-form-item>
		  <!-- 密码 -->
		  <el-form-item prop="password">
		    <el-input prefix-icon="iconfont icon-3702mima" v-model="loginForm.password" type="password"></el-input>
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
	data(){
		return {
			// 登录表单的数据绑定对象
			loginForm: {
				username: '3167253066',
				password: 'hzddsy123'
			},
			// 表单的验证规则对象
			loginFormRules: {
				// 验证用户名
				username: [
					{ required: true, message: '请输入登录名称', trigger: 'blur' }, 
					/*trigger: 文本框失去焦点时触发验证*/
					{ min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
				],
				// 验证密码
				password: [
					{ required: true, message: '请输入登录名称', trigger: 'blur' },
					{ min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
				]
			}
		}
	},
	methods: {
		// 点击重置按钮重置登陆表单
		resetLoginForm() {
			this.$refs.loginFormRef.resetFields();
		},
		// 登陆前的预验证
		login(){
			this.$refs.loginFormRef.validate(async valid => {
				// console.log(valid);  /* 验证成功valid为true */
				if (!valid) return;
				const result = await this.$http.post("users/login", this.loginForm);  /*$http,应先在main里配置， this.loginForm为参数*/

				const {data: res} = result
				console.log(res)
				if(res.meta.status !== 200) return this.$message.error("登录失败！");
				this.$message.success("登陆成功");
				// 设置token
				window.sessionStorage.setItem('token', res.data);
				
				// 痛过编程式导航跳转到后台主页，路由地址是/home
				this.$router.push("/home");
			})
		}
	}
}
</script>


<style lang="less" scoped>  /*设置style只在当前组件内生效*/
.login_container {
	background-color: #2b4b6b;
	height: 100%;
}

.login_box {
	width: 450px;
	height: 300px;
	background-color: #fff;
	border-radius: 3px;
	position: absolute;
	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);

	.avatar_box {
		height: 130px;
		width: 130px;
		border: 1px solid #eee;
		border-radius: 50%;
		box-shadow: 0 0 10px #ddd;
		padding: 10px;
		position: absolute;  /*绝对定位*/
		left: 50%;  /*距离左侧50%*/
		transform: translate(-50%, -50%);  /*向后移动图片自身的50%,再向上移动50%, %根据avatar_box的高度来比*/
		background-color: #fff;
		img{
			width: 100%;
			height: 100%;
			border-radius: 50%;
			background-color: #eee;
		}
	}
}
.login_form{
	position: absolute;
	bottom: 0;
	width: 100%;
	padding: 0 20px;
	box-sizing: border-box;  /*嘎嘎嘎嘎*/
}
.btns{
	display: flex;
	justify-content: flex-end;
}
</style>
