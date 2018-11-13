<template>
	<div class="login">
	    <img src="./assets/1.jpg" alt="">
		<div class="info">
			<h2 class="title">用户后台管理系统</h2>
			<el-form ref='userForm' :rules='rules' :model="form" size="mini" label-position="left" >
		    <el-form-item prop='username' label="用户名" label-width="6em">
		      <el-input v-model="form.username"></el-input>
		    </el-form-item>
		    <el-form-item prop='password' label="密码" label-width="6em">
		      <el-input v-model="form.password" type="password"></el-input>
		    </el-form-item>
		  </el-form>
		  <div class="btns">
		  	<el-button size="mini" @click='login'>登录</el-button>
		  </div>
		</div>
	</div>
</template>
<script>
	import axios from '@/http/axios'
	export default {
		data(){
			return {
				form:{},
				rules:{
					username:[{
						required:true,
						message:'请输入用户名',
						trigger:'blur'
					}],
					password:[{
						required:true,
						message:'请输入密码',
						trigger:'blur'
					}]
				}
			}
		},
		methods:{
			login(){
				this.$refs.userForm.validate((valid)=>{
					if(valid){
						axios.post('/login',this.form)
						.then(({data:result})=>{
							if(result.status == 200 && result.message=='登录成功'){
								//登录成功的处理
								//1. 页面跳转
								window.vm.currentComponent='App';
								//2. 用户信息的保存
								localStorage.setItem('user',JSON.stringify(result.data))
							}
						})
						.catch((error)=>{
							this.$notify.error({
								title:'错误',
								message:'网络异常'
							});
						});
					}
				})
			}
		}
	}
</script>
<style>
	input:-webkit-autofill {
    -webkit-box-shadow: 0 0 0px 1000px white inset !important;
	}
	body{
		padding: 0px;
		margin: 0px;
	}
	.login > img{
		width:100%;
		height: 620px;
	}
	.login .info {
		width: 400px;
		position: absolute;
		top: 120px;
		left: 420px;
	}
	.info .title {
		text-align: center;
		margin-bottom: 30px;
	}
	.login .info .btns {
		text-align: right;
	}
	
</style>