<template>
	<div>
		<el-form :inline="true" :model="tableList" class="demo-form-inline query-form">
			<el-form-item label="角色名称:" prop="name">
				<el-input size="mini"  v-model="tableList.name" placeholder=""></el-input>
			</el-form-item>
			<el-form-item label="状态" prop="status">
			     <el-select v-model="tableList.status" size="mini"  placeholder="请选择状态" >
				    <el-option
				      v-for="item in selectList"
				      :key="item.key"
				      :label="item.value"
				      :value="item.key" >
				    </el-option>
				</el-select>
		    </el-form-item>
	      	<el-form-item>
	            <el-button type="primary" size="mini"  @click="getSarch(tableList)" >查询</el-button>
	        </el-form-item>
		</el-form>
		<div class="table"> 
				<el-row class="container">
					<el-col :span='24' >
						<el-row class="all-classify">
							<el-col :span='22' >
							</el-col>
							<el-col :span='2'>
								 <el-button type="success" size="mini" @click="changeFn('','add')">添加</el-button>
							</el-col>
						</el-row>
					</el-col>
				</el-row>
	            <el-table :data="tableDatas"  border style="width: 100%;"  v-loading="loading">
				    <el-table-column   prop="code" label="角色编号">  </el-table-column>
				    <el-table-column   prop="name" label="角色名称">  </el-table-column>
				    <el-table-column   prop="organId" label="所属机构">  </el-table-column>
				    <el-table-column   prop="status" label="状态">
				    	<template slot-scope="scope" >
	           				<span v-if="scope.row.status==='1'">启用</span>
	           				<span v-if="scope.row.status==='0'">禁用</span>
	           				<span v-if="scope.row.status==='9'">删除</span>
	           			</template>
				    </el-table-column>
		            <el-table-column label="相关操作" >
		          		<template slot-scope="scope" >
	           				<el-button size="small" @click="changeFn(scope.row.id,'change')">修改</el-button>
	           				<el-button  size="small"  type="danger" @click="deleteFn(scope.row.code)">删除</el-button>
	           			</template>
		       		</el-table-column>
				</el-table>
				<el-pagination class="pagination"  @current-change="handleCurrentChange" background layout="prev, pager, next , jumper"  :current-page.sync="tableList.pageIndex" :total="totalCount" >
			</el-pagination>
		</div>
	</div>
</template>
<script>
	import qs from 'qs'
	import Vue from 'vue'
	import router from '@/router'
	import {getRoleUserPost,deletRolesPost} from '../../api/api.js'
	export default{
		data(){
			return{
				oldStatus:'',
				selectList:[
					{value: "全部", key: ''}, 
					{value: "启用", key: '1'}, 
					{value: "禁用", key: '0'},
					{value: "删除", key: '9'}
		        ],
				totalCount:1,
				loading: true,
				tableList:{
					name:'',
					status:'',
					pageIndex:1
				},
				tableDatas:[],//列表的值
			}
		},
		methods:{
			//搜索接口
			getSarch(formName){
				// formName.status = '9';
				this.getTablelist(formName)
			},
			//点击哪页触发的时间
			handleCurrentChange(val) {
				this.tableList.pageIndex=val
				this.getTablelist()
				console.log(this.tableList.pageIndex);
	   			// console.log(`当前页: ${val}`);
	        },
			getTablelist(par){
				getRoleUserPost(par).then((data) => {
					this.tableDatas=data.data
					this.totalCount=data.pageInfo.totalCount
					 this.loading=false
				}).catch(message => {
					this.$message.error("请求失败，请联系客服，失败码"+message);
					 this.loading=false
				})
			},
			changeFn(id,AddOrChange){
				this.$router.push({path:'/RoleManaChange',query:{id:id,AddOrChange:AddOrChange}})
			},
			deleteFn(code){
				 this.$confirm('确定删除该用户?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
		        }).then(() => {
					deletRolesPost({code:code}).then(data =>{
						this.getTablelist()
						 this.$message({
				            type: 'info',
				            message: '删除成功'
				        });   
					}).catch(message => {
						this.$message.error("请求失败，请联系客服，失败码"+message);
					    this.loading=false
					})
		        }).catch(() => {
			        this.$message({
			            type: 'info',
			            message: '取消删除'
			        });          
		        });
				
			}
		},
		mounted(){
			this.getTablelist(this.tableList);
		}
	}
</script>