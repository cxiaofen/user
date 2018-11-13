<template>
	<div class="article">
	    <!-- 按钮 -->
		<div class="btns">
		    <el-select size='mini' v-model="params.categoryId" clearable placeholder="所有栏目" style='width:120px'>
		        <el-option
			      v-for="c in categories"
			      :key="c.id"
			      :label="c.name"
			      :value="c.id">
			    </el-option>
			</el-select>
			<el-input
			  placeholder="请输入关键字"
			  size='mini'
			  style='width:120px'
			  v-model="params.keywords"
			  clearable>
			</el-input>
			<el-button size='mini' @click='toAddArticle'>新增</el-button>
			<el-button size='mini' @click='batchDeleteArticle'>批量删除</el-button>
		</div>
		<!-- 按钮结束 -->
		<!-- 文章管理表格 -->
		<div class="article-tbl" v-loading="loading">
			<el-table
		      :data="articles"
		      style="width: 100%" 
		      size='mini'
		      @selection-change="handleSelectionChange" 
		      :border='true'>
		      <el-table-column
		        fixed='left'
		        type="selection"
		        width="40">
		      </el-table-column>
		      <el-table-column
		        prop="title"
		        label="文章标题"
		        width="350">
		      </el-table-column>
		      <el-table-column
		        prop="category.name"
		        align='center'
		        label="所属栏目"
		        width="130">
		      </el-table-column>
		      <el-table-column
		        prop="author"
		        label="作者"
		        align='center'
		        width="120">
		      </el-table-column>
		      <el-table-column
		        prop="publishtime"
		        align='center'
		        width="200"
		        label="发布时间">
		      </el-table-column>
		      <el-table-column
		        prop="readtimes"
		        label="阅读次数"
		        align='center'
	            width="160">
		      </el-table-column>
		      <el-table-column
		        fixed='right' 
		        label="操作"
		        width="130"
		        align='center'>
		        <template slot-scope='{row}'>
		        	<i class="fa fa-trash" @click='deleteArticle(row.id)'></i>
		        	<i class="fa fa-pencil" @click='toUpdateArticle(row)'></i>
		        </template>
		      </el-table-column>
		    </el-table>
		</div>
		<!-- 文章管理表格结束 -->
		<!-- 分页 -->
		<div class="page">
			<el-pagination 
		    layout="prev, pager, next"
		    :page-size='params.pageSize'
		    :total="total"
		    @current-change='handlerCurrentChange'>
			</el-pagination>
		</div>
		<!-- 模态框 -->
		<el-dialog fullscreen :title="articleDialog.title" :visible.sync="articleDialog.visible">
		<!-- {{articleDialog.form}} -->
		  <el-form :model="articleDialog.form" size='mini' label-position='left'>
		    <el-form-item label="咨询标题" label-width="6em">
		      <el-input v-model="articleDialog.form.title" autocomplete="off"></el-input>
		    </el-form-item>
		    <el-form-item label="资讯栏目" label-width="6em">
		      <el-select v-model="articleDialog.form.categoryId" placeholder="一级栏目">
		        <el-option :key='c.id' v-for='c in categories' :label="c.name" :value="c.id"></el-option>
		      </el-select>
		    </el-form-item>
		    <el-form-item label="列表样式" label-width="6em">
		      <ul class="list_style">
		      	<li class="style_one" :class="{current:articleDialog.form.liststyle=='style-one'}" @click="articleDialog.form.liststyle = 'style-one'">
		      		<img src="@/assets/style-one.jpg" alt="">
		      	</li>
		      	<li class="style_two" :class="{current:articleDialog.form.liststyle=='style-two'}" @click="articleDialog.form.liststyle = 'style-two'">
		      		<img src="@/assets/style-two.jpg" alt="">
		      	</li>
		      </ul>
		    </el-form-item>
		    <el-form-item label="缩略图" label-width="6em">
		        <el-upload
				  action="http://120.78.164.247:8099/manager/file/upload"
				  :on-success='handleUploadSuccess'
				  :file-list='fileList'
				  :on-remove='handleFileRemove'
				  list-type="picture">
				  <el-button size="small" type="primary">点击上传</el-button>
				  <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
				</el-upload>
		    </el-form-item>
		    <el-form-item label="咨询正文" label-width="5em">
		      <mavon-editor ref="articleContent" v-model="articleDialog.form.content"/>
		    </el-form-item>
		  </el-form>
		  <div slot="footer" class="dialog-footer">
		    <el-button size='mini' @click='closeArticleDialog'>取 消</el-button>
		    <el-button size='mini' type='primary' @click='saveOrUpdateArticle'>确 定</el-button>
		  </div>
		</el-dialog>
	</div>
</template>
<script>
    import axios from '@/http/axios'
	export default {
		data(){
			return {
				articles:[],
				multipleSelection:[],
				loading:false,
				fileList:[],
				categories:[],
				total:10,
				params:{
					page:0,
					pageSize:5,
					categoryId:undefined,
					keywords:undefined
				},
				articleDialog:{
					title:'',
					visible:false,
					form:{
						liststyle:'style-one',
						fileIds:[]
					}
				}
			}
		},
		methods:{
			handleFileRemove(file){
				// 通过id删除附件
				axios.get('manager/file/delete',{
					params:{id:file.name}
				})
				.then(()=>{
					this.$notify.success({
						title:'成功',
						message:'删除成功！'
					});
					_.remove(this.articleDialog.form.fileIds,(id)=>{
						return id == file.name
					})
					this.articleDialog.form.fileIds.push(1);
					this.articleDialog.form.fileIds.pop();
				})
				.catch(()=>{
					this.$notify.error({
						title:'失败',
						message:'删除失败！'
					});
				});
			},
			handleSelectionChange(val){
				this.multipleSelection = val;
			},
			// 翻页
			handlerCurrentChange(page){
				this.params.page = page -1;
			},
			//查询所有文章
			findAllArticle(){
				this.loading = true;
				axios.get('/manager/article/findArticle',{ params:this.params })
				.then(({data:result})=>{
					this.total = result.data.total;
					this.articles = result.data.list;
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
			// 查询所有栏目
			findAllCategory(){
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
			},
			// 添加文章
			toAddArticle(){
				this.articleDialog.title = '发布资讯';
				this.articleDialog.form = {
						liststyle:'style-one',
						fileIds:[]
				};
				this.fileList = [];
				this.articleDialog.visible = true;
			},
			// 关闭模态框
			closeArticleDialog(){
				this.articleDialog.form = {};
                this.articleDialog.visible = false;
			},
			// 上传文件
			handleUploadSuccess(response,file,fileList){
				file.name = response.data.id;
				this.articleDialog.form.fileIds.push(response.data.id);
			},
			// 保存或更新
			saveOrUpdateArticle(){
			    this.articleDialog.form.source = this.$refs.articleContent.d_render;
			    // console.log(_.clone(this.$refs.articleContent));
	        	axios.post('/manager/article/saveOrUpdateArticle',this.articleDialog.form)
	        	.then(()=>{
	        		this.$notify.success({
            			title:'成功',
            			message:'添加成功！'
            		});
            		this.closeArticleDialog();
            		this.articleDialog.form = {};
            		this.findAllArticle();
	        	})
	        	.catch(()=>{
	        		this.$notify.error({
            			title:'错误',
            			message:'添加失败！'
            		});
	        	})
			},
			// 删除文章
			deleteArticle(id){
		       this.$confirm('此操作将永久删除该数据, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        }).then(()=>{
		        	axios.get('/manager/article/deleteArticleById?id='+id)
		        	.then(()=>{
		        		this.$notify.success({
	            			title:'成功',
	            			message:'删除成功！'
	            		});
	            		this.findAllArticle();
		        	})
		        	.catch(()=>{
		        		this.$notify.error({
	            			title:'错误',
	            			message:'删除失败！'
	            		});
		        	})
		        })
			},
			//批量删除
			batchDeleteArticle(){
				this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        }).then(()=>{
		        	let ids = this.multipleSelection.map((item)=>{
		        		return item.id;
		        	});
		        	axios.post('/manager/article/batchDeleteArticle',{ids})
		        	.then(()=>{
		        		this.$notify.success({
	            			title:'成功',
	            			message:'删除成功！'
	            		});
	            		this.findAllArticle();
		        	})
		        	.catch(()=>{
		        		this.$notify.error({
	            			title:'错误',
	            			message:'删除失败！'
	            		});
		        	})
		        })
			},
			// 修改
			toUpdateArticle(row){
				// 删除category，添加categoryId
				let article = _.clone(row)
				article.categoryId = article.category.id;
				delete article.category;
				// 处理附件的默认显示
				this.fileList = article.articleFileVMs.map((item)=>{
					return {
						name:item.cmsFile.id,
						url:'http://47.107.37.90:8099/group1'+item.cmsFile.id
					}
				})
				// 删除articleFileVMs，添加fileIds
				article.fileIds = article.articleFileVMs.map(item=>item.cmsFile.id);
				delete article.articleFileVMs
				// 将值为空的属性删除
				for(let key in article){
					let val = article[key];
					if(!val){
						delete article[key];
					}
				}

				this.articleDialog.form = article;
				this.articleDialog.title = '修改栏目';
				this.articleDialog.visible = true;
			}
		},
		created(){
			this.findAllArticle();
			this.findAllCategory();
		},
		// 监听当params改变时刷新
		watch:{
			params:{
				handler:function(){
					this.findAllArticle();
				},
				deep:true
			}
		}
	}
</script>
<style>
	.btns{
		margin:0.5em;
	}
	i.fa{
		margin:0 .3em;
		cursor: pointer;
	}
	i.fa.fa-trash{
		color: #F56C6C;
	}
	.list_style > li {
		width: 200px;
		height: 80px;
		border: 1px solid #ededed;
		border-radius: 3px;
		padding: .5em;
	}
	.list_style > .current{
		border-color: #409eff;
	}
	.list_style > li  img{
        width: 100%;
	}
	.list_style .style_one {
		float: left;
	}
	.list_style .style_two {
		margin-left: 220px;
	}
</style>