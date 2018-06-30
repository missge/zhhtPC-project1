<template>
	<div>
		<div class="FormFile">
			<el-form :model="rolesList" :rules="rules" :ref="rolesList" label-width="150px" class="demo-ruleForm">
				  <el-form-item label="角色编号" prop="code">
				 	 <span readonly>{{rolesList.code}}</span>
				  </el-form-item>
				  <el-form-item label="角色名称" prop="name">
				    <el-input v-model="rolesList.name" placeholder="请输入角色名称"></el-input>
				  </el-form-item>
				  <el-form-item label="所属机构" prop="organId">
				    <el-select v-model="rolesList.organId" placeholder="请选择所属机构">
				      <el-option
					      v-for="item in organId_status"
					      :key="item.key"
					      :label="item.value"
					      :value="item.key" >
					    </el-option>
				    </el-select>
				  </el-form-item>
				  <el-form-item label="状态" prop="status">
				    <el-select v-model="rolesList.status" placeholder="请选择状态">
					    <el-option
					      v-for="item in state_status"
					      :key="item.key"
					      :label="item.value"
					      :value="item.key" >
					    </el-option>
				    </el-select>
				  </el-form-item>
				  <el-form-item label="角色类型" prop="isCommon">
				    <el-radio-group v-model="rolesList.isCommon">
				      <!-- <el-radio v-for="items in visible_status" label="items.key">{{items.value}}</el-radio> -->
				      <el-radio label="0">非通用角色</el-radio>
				      <el-radio label="1">通用角色</el-radio>
				      <el-radio label="2">供应商默认角色</el-radio>
				      <el-radio label="3">商城默认角色</el-radio>
				    </el-radio-group>
				  </el-form-item>
	  			  <el-form-item label="权限" prop="ids">
					   <el-tree
						  :data="treeJson"
						  show-checkbox
						  default-expand-all
						  node-key="id"
						  ref="rolestree"
						  highlight-current
						  :default-checked-keys="rolesList.ids">
						</el-tree>
	  			  </el-form-item>
				 <el-form-item label="描述">
				    <el-input type="textarea" v-model="rolesList.memo"></el-input>
				 </el-form-item>
				  <el-form-item>
				    <el-button type="primary" @click="submitForm(rolesList)">保存</el-button>
				    <el-button  @click="goback()">返回</el-button>
				  </el-form-item>
			</el-form>
		</div>
	</div>
</template>
<script>
	import {editRoles,editRolesPost,createRolesPost,rolesCreate,roleType} from '../../api/api.js'
	import Vue from 'vue'
	import router from '@/router'
	 export default {
	    data() {
	      return {
	      	id:this.$route.query.id,
	      	AddOrChange:this.$route.query.AddOrChange,
	      	state_status:[
		      	{
		            "value": "启用",
		            "key": 1
		        },
		        {
		            "value": "禁用",
		            "key": 0
		        },
		        {
		            "value": "删除",
		            "key": 9
		        }
	        ],
		    visible_status:[
		      	{
		            "value": "非通用角色",
		            "key": "0"
		        },
		        {
		            "value": "通用角色",
		            "key": "1"
		        },
		        {
		            "value": "供应商默认角色",
		            "key": "2"
		        },
		        {
		            "value": "商城默认角色",
		            "key": "3"
		        }
	        ],
	      	organId_status:[
	      		{
		            "value":"测试",
		            "key":"00000002"
		        }
	      	],
		    treeJson:[],
	        rolesList: {
	        	code:'',
	          	name: '',
	          	status: 1,
	          	isCommon:"0",
	          	organId: '00000002',
	          	ids: [],
	          	memo: ''
	        },
	        rules: {
	          name: [
	            { required: true, message: '请输入角色名称', trigger: 'blur' }
	          ]
	        },
	        tableDatas:''
	      };
	    },
	    methods: {
		    submitForm(formName) {
		        this.$refs[formName].validate((valid) => {
		          if (valid) {
		            let tmpids= this.$refs.rolestree.getCheckedKeys();
 					formName.ids = tmpids.join(',');
 					console.log(formName);
			    	if(this.AddOrChange=='change'){
						editRolesPost(formName).then((data)=>{
							if(data.code==1){
								this.$message.success("修改成功");
							    this.$router.push('/RoleManagement');
							}else{
									this.$message.error(data.descript);
							}
						})
					}else{
						createRolesPost(formName).then((data)=>{
							if(data.code==1){
									this.$message.success("添加成功");
							        this.$router.push('/RoleManagement');
							}else{
									this.$message.error(data.descript);
							}
						})
					} 					
		          } else {
		            this.$message.error("请输入角色名称");
		            return false;
		          }
		        });
		    },
		    resetForm(formName) {
		        this.$refs[formName].resetFields();
		    },
		    getTablelist(){
		  //   	roleType().then((data) => {
				// 	this.state_status = data.state_status;
				// 	this.visible_status = data.visible_status;
				// }).catch(message => {
				// 	this.$message.error("请求失败，请联系客服，失败码"+message);
				// 	this.loading=false
				// });
		    	//标志是点击添加，还是修改按钮
				if(this.AddOrChange=='change'){
					editRoles({id:this.id}).then((data) => {
						let temp = this.initTreeData(data.jsonsAll)
						this.treeJson = temp;
						this.rolesList = data.roles;
						this.rolesList.ids = data.ids ? data.ids.split(",") : [];

						console.log(this.rolesList.ids)
					}).catch(message => {
						this.$message.error("请求失败，请联系客服，失败码"+message);
						this.loading=false
					});
				}else{
					rolesCreate().then((data) => {
						let temp = this.initTreeData(data.jsonsAll)
						this.treeJson = temp;
						console.log(data.code)
						this.rolesList.code = data.code;
					}).catch(message => {
						this.$message.error("请求失败，请联系客服，失败码"+message);
						this.loading=false
					});
				}
			},
			initTreeData(data){
				let treeData = [];
				let _self = this;
				data.map(function(item){ 
					let obj={}, children = '';;
					obj.label = item.name;
					obj.id = item.id;
        //树形菜单遍历  调用element tree组件
					if(item.fatherModuleDTOs){
						children = item.fatherModuleDTOs;
					}else if(item.childModuleDTOs){
						children = item.childModuleDTOs;
					}else if(item.privilegesDtos){
						children = item.privilegesDtos;
					}else{
						children = '';
					}
					if(children !== '') {
						obj.children = _self.initTreeData(children);
					}
					treeData.push(obj);
				})
				return treeData;
			},
			goback(){
		    	this.$router.push('./RoleManagement')
		    }
	    },
	    mounted(){
	    	this.getTablelist();
	    }
 	}
</script>
<style>
	.FormFile{
		margin:0 auto;
		background:#fff;
	}	
	.FormFile>.el-form{
		width:80%;
		padding: 24px;
	}
	.FormFile .el-form-item__content{text-align:left;}
</style>
