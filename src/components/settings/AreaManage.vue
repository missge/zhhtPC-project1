<template>
	<div>
		<div class="AreaFormFile">
			<el-form :ref="areaform" :model="areaform" :rules="rules" class="demo-ruleForm el-form-item__content">
				<el-form-item label="模板名称:" prop="name">
					<el-input v-model="areaform.name" placeholder=""></el-input>
				</el-form-item>

				<div><span>地区选择：</span></div>
				<el-form-item prop="zzs" >
					<div class="lable"><span>直辖市：</span></div>
					<el-checkbox-group v-model="areaform.zzs">
					   <el-checkbox v-for="(item,index) in zzs" :key="index" :label="item.citys[0].code">{{item.citys[0].name}}</el-checkbox>
					</el-checkbox-group>
				</el-form-item>

				<el-form-item prop="qts">
					<div class="lable"><span>省　市：</span></div>
					<div v-for="items in qts">
						<el-checkbox class="provinces" :label="items.code" :indeterminate="items.indeterminate" v-model="items.checkAllState"  @change="handleCheckAllChange(items,'qts',items.checkAllState)">{{items.name}}</el-checkbox>
						<el-checkbox-group v-model="areaform.qts" @change="handleCheckedCitiesChange(items,'qts')">
					   		<el-checkbox v-for="(item,index) in items.citys" :key="index" :label="item.code">{{item.name}}</el-checkbox>
						</el-checkbox-group>
					</div>
				</el-form-item>

				<el-form-item prop="zzq">
					<div class="lable"><span>自治区：</span></div>
					<div v-for="items in zzq">
						<el-checkbox class="provinces" :label="items.code" :indeterminate="items.indeterminate" v-model="items.checkAllState"  @change="handleCheckAllChange(items,'zzq',items.checkAllState)">{{items.name}}</el-checkbox>
						<el-checkbox-group v-model="areaform.zzq"  @change="handleCheckedCitiesChange(items,'zzq')">
					   		<el-checkbox v-for="(item,index) in items.citys" :key="index" :label="item.code">{{item.name}}</el-checkbox>
						</el-checkbox-group>
					</div>
				</el-form-item>

				<el-form-item>
				    <el-button type="primary" @click="seveFn(areaform)">保存</el-button>
				    <el-button @click="goback()">返回</el-button>
				</el-form-item>
			</el-form>
		</div>
	</div>
</template>

<script>
	import {getfindAreaCity,editAreaTemp,addSaveTemp,editSaveTemp} from '../../api/api.js';
	import qs from 'qs'
	import Vue from 'vue'
	import router from '@/router'
	export default {
	    data() {
	      return {
	      	rules: {
			  name: [
			  	{ required: true, message: '请输入模板名称', trigger: 'blur' },
			  ]
	        },
	      	id:this.$route.query.id,
	      	AddOrChange:this.$route.query.AddOrChange,
        	zzs:[],
        	qts:[],
        	zzq:[],
        	areaform: {
	          name: '',
	          zzs: [],
	          qts: [],
	          zzq: []
	        },
	        cityOptions:''
	      };
	    },
	    methods: {
	    	getCityOptions(){
	    		getfindAreaCity().then((data) => {
					// this.getCityOptions(data.data);
					this.getDataList(data);
					this.getTablelist();
				}).catch(message => {
					this.$message.error("请求失败，请联系客服，失败码"+message);
				})
	    	},
			getTablelist(){
				//标志是点击添加，还是修改按钮
				if(this.AddOrChange=='change'){
					let _self = this;
					editAreaTemp({id:this.id}).then((data) => {
						this.areaform.name = data.areaTemplatesName;
						let citylist = data.data;
						_self.setCheckedCity(citylist);
					}).catch(message => {
						this.$message.error("请求失败，请联系客服，失败码"+message);
						this.loading=false
					})
				}else{
					
				}
			},
			setCheckedCity(citylist){
				console.log(citylist);
				console.log(this.cityOptions);
				var _self = this;
				_self.cityOptions.map(function(cs){
					citylist.map(function(c){
						let cityid = c.cityId+'';
						if(cityid === cs.code){
							
							switch(cs.IsDirect){
								case '0':
				    				_self.areaform.zzs.push(cityid);
				    				break;
				    			case '1':
				    				_self.areaform.qts.push(cityid);
				    				break;
				    			case '2':
				    				_self.areaform.zzq.push(cityid);
				    				break;
							}
						}
					});
				});

				console.log(this.areaform)
			},
			seveFn(formName){
				this.$refs[formName].validate((valid,object) => {
					console.log(formName);
			    	 if (valid) {
			    	 	let temForm = {}
			    	 	temForm.tempName = formName.name;
			    	 	temForm.strTemp = formName.zzs.concat(formName.qts,formName.zzq).join('-')
			    	 	if(this.AddOrChange=='change'){
			    	 		temForm.id = this.id;
			    	 		console.log(temForm);
							editSaveTemp(temForm).then((data)=>{
								if(data.code==1){
									this.$message({
							          message: '修改成功',
							          type: 'success'
							        });
							        this.$router.push('/AreaTemp');
								}else{
									this.$message.error(data.descript);
								}
							})

						}else{
							addSaveTemp(temForm).then((data)=>{
								if(data.code==1){
									this.$message({
							          message: '添加成功',
							           type: 'success'
							        });
							        this.$router.push('/AreaTemp');
								}else{
									this.$message.error(data.descript);
								}
							})
						}

			          } else {
			            this.$message.error(object.name[0].message);
			            return false;
			          }
		        });
			},
			handleCheckAllChange(pra,pra1,state) {
				let opt = state ? 'addAll' : 'deleteAll';
				let checkedArr = this.getCitydata(pra,this.areaform[pra1],opt);
		        this.areaform[pra1] = checkedArr;
		        pra.indeterminate = false;
		        
		    },
		    handleCheckedCitiesChange(pra,pra1) {
		    	let checkedArr = this.getCitydata(pra,this.areaform[pra1],'add')
		        let checkedCount = checkedArr.length;
		        pra.checkAllState = checkedCount === pra.citys.length;
		        pra.indeterminate = checkedCount > 0 && checkedCount < pra.citys.length;
		        
		    },
		    getCitydata(pra,data,opt){
		    	// 获取数据的值
		    	let tempCitys = pra.citys;
		    	let areaData = data;
		    	let tempArr = [];

				switch(opt){
		    		case 'add':
		    			areaData.map(function(d){
		    				tempCitys.map(function(t){
		    					if(t.code == d){
		    						tempArr.push(d);
		    					}		    				
		    				})
		    			})
		    			break;	
		    		case 'addAll':
						tempCitys.map(function(t){
			    			tempArr.push(t.code);		    				
			    		});

			    		tempArr = [...new Set(areaData.concat(tempArr))];
		    			break;
		    		case 'deleteAll':
		    			tempArr = areaData;
			    		for(let i=0;i<tempCitys.length;i++){
						    for(let j=0;j<tempArr.length;j++){
						        if(tempArr[j]==tempCitys[i].code){
						            tempArr.splice(j,1);
						            j--;
						        }
						    }
						}
		    			break;
		    	}
		    	return tempArr;
			},
		    getDataList(data){

		    	let provinces=data.data;
		    	let city = this.cityOptions = data.citys;
		    	let zzs = [],zzq = [],qts = [];

		    	provinces.map(function(p){
		    		p.citys = [];
		    		
		    		city.map(function(c){

		    			if(p.code === c.provinceCode){
		    				p.citys.push(c)
		    			}
		    		});
		    	});

		    	provinces.map(function(d){
		    		switch(d.IsDirect){
		    			case '0':
		    				zzs.push(d);
		    				break;
		    			case '1':
		    				d.checkAllState = false;
		    				d.indeterminate = false;
		    				qts.push(d);
		    				break;
		    			case '2':
		    				zzq.push(d);
		    				break;
		    		}
		    	})

		    	this.zzs = zzs;
		    	this.qts = qts;
		    	this.zzq = zzq;
		    },
		    goback(){
		    	this.$router.push('./AreaTemp')
		    }
	    },
	    mounted(){
	    	//cityOptions从接口中调数据
	    	this.getCityOptions();

	    }
 	}
</script>
<style>
	.AreaFormFile{
		margin:0 auto;
		background:#fff;
	}	
	.AreaFormFile>.el-form{
		width:80%;
		padding: 24px;
	}
	.AreaFormFile .el-form-item__content{text-align:left;}
	.AreaFormFile .el-form-item {
    	/*margin-bottom: 0;*/
	}

	.AreaFormFile .provinces .el-checkbox__label{
		font-weight: bold;
		font-size: 14px
	}

	.AreaFormFile .provinces.el-checkbox:first-child{
		margin-left: 0px;
	}

	.AreaFormFile .el-checkbox__label{
		font-size: 12px
	}

	.AreaFormFile .lable>span{
		font-size: 16px;
		font-weight: bold;
	}

	.AreaFormFile .el-checkbox:first-child {
    	margin-left: 30px;
	}
</style>