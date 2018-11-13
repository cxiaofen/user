<template>
	<div class="category">
		<!-- 按钮区 -->
		<div class="btns">
			<el-button size='mini' @click='toAddCategory'>新增</el-button>
			<el-button size='mini' @click='batchDeleteCategory'>批量删除</el-button>
		</div>
		<!-- 按钮区结束 -->
		<!-- 栏目管理表格 -->
		<div class="category_tbl" v-loading="loading">
			<el-table
		      :data="categories"
		      style="width: 100%" 
		      size='mini'
		      @selection-change="handleSelectionChange" 
		      :border='true'>
		      <el-table-column
		        type="selection"
		        width="40">
		      </el-table-column>
		      <el-table-column
		        prop="name"
		        label="栏目名称"
		        width="120">
		      </el-table-column>
		      <el-table-column
		        prop="parent.name"
		        label="父栏目"
		        width="120">
		      </el-table-column>
		      <el-table-column
		        prop="comment"
		        label="描述">
		      </el-table-column>
		      <el-table-column
		        label="操作"
		        width="80">
		        <template slot-scope='{row}'>
		        	<i class="fa fa-trash" @click='deleteCategory(row.id)'></i>
		        	<i class="fa fa-pencil" @click='toUpdateCategory(row)'></i>
		        </template>
		      </el-table-column>
		    </el-table>
		</div>
		<!-- 表格结束 -->
		<!-- 模态框 -->
		<el-dialog :title="cDialog.title" :visible.sync="cDialog.visible">
		  <el-form :model="cDialog.form" size='mini' label-position='left'>
		    <el-form-item label="栏目名称" label-width="6em">
		      <el-input v-model="cDialog.form.name" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="父栏目" label-width="6em">
		      <el-select v-model="cDialog.form.parentId" placeholder="一级栏目">
		        <el-option :key='c.id' v-for='c in categories' :label="c.name" :value="c.id"></el-option>
		      </el-select>
		    </el-form-item>
		    <el-form-item label="描述" label-width="6em">
		      <el-input v-model="cDialog.form.comment" type='textarea' :rows='2'></el-input>
		    </el-form-item>
		  </el-form>
		  <div slot="footer" class="dialog-footer">
		    <el-button size='mini' @click='closeCDialog'>取 消</el-button>
		    <el-button size='mini' type='primary' @click='saveOrUpdateCategory'>确 定</el-button>
		  </div>
		</el-dialog>
		<!-- 模态框结束 -->
	</div>
</template>
<script>
    import axios from '@/http/axios'
	export default {
         data(){
         	return {
         		multipleSelection:[],
         		categories:[],
         		loading:false,
         		cDialog:{
         			title:'',
         			visible:false,
         			form:{}
         		}
         	}
         },
         created(){
         	// 查询所有栏目信息
         	this.findAllCategories();
         },
         methods:{
         	// 查询所有栏目
            findAllCategories(){
            	this.loading=true;
            	axios.get('/manager/category/findAllCategory')
            	.then(({data:result})=>{
            		this.categories = result.data;
            	})
            	.catch(()=>{
            		this.$notify.error({
            			title:'错误',
            			message:'网络异常！'
            		});
            	})
            	.finally(()=>{
            		this.loading = false;
            	})
            },
            // 通过id删除
            deleteCategory(id){
            	this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        }).then(()=>{
		        	axios.get('/manager/category/deleteCategoryById?id='+id)
	            	.then(()=>{
	            		this.$notify.success({
	            			title:'成功',
	            			message:'删除成功！'
	            		});
	            		this.findAllCategories();
	            	})
	            	.catch(()=>{
	            		this.$notify.error({
	            			title:'错误',
	            			message:'删除失败！'
	            		})
	            	});
			    })
            },
            handleSelectionChange(val) {
	            this.multipleSelection = val;
	        },
	        // 批量删除
	        batchDeleteCategory(){
                this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        }).then(()=>{
                   let ids = this.multipleSelection.map((item)=>{
	                	return item.id;
	                });
	                axios.post('/manager/category/batchDeleteCategory',{ids})
	                .then(()=>{
	                	this.$notify.success({
		            		title:'成功',
		            		message:'删除成功！'
		            	});
		            	this.findAllCategories();
	                })
	                .catch(()=>{
	                	this.$notify.error({
	            			title:'错误',
	            			message:'删除失败！'
	            		})
	                })
		        })
            },
            // 添加栏目
            toAddCategory(){
            	this.cDialog.title='添加栏目';
            	this.cDialog.visible = true;
            },
            // 关闭模态框
            closeCDialog(){
                this.cDialog.form={};
                this.cDialog.visible=false;
            },
            //保存或更新栏目
            saveOrUpdateCategory(){
            	axios.post('/manager/category/saveOrUpdateCategory',this.cDialog.form)
            	.then(()=>{
	                	this.$notify.success({
		            		title:'成功',
		            		message:'提交成功！'
		            	});
		            	this.closeCDialog();
		            	this.findAllCategories();
	                })
	                .catch(()=>{
	                	this.$notify.error({
	            			title:'错误',
	            			message:'提交失败！'
	            		})
	            });
            },
            // 修改栏目
            toUpdateCategory(row){
            	this.cDialog.title='修改栏目';
            	row.parentId = row.parent.id;
            	this.cDialog.form=row;
            	this.cDialog.visible=true;
            }
        }
	}
</script>
<style>
	.btns{
		margin-bottom: 1em;
	}
	i.fa{
		margin:0 .3em;
		cursor: pointer;
	}
	i.fa.fa-trash{
		color: #F56C6C;
	}
</style>