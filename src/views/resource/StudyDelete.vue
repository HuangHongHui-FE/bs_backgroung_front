<template>
	<div>
		<!-- 面包屑导航区域 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
		  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
		  <el-breadcrumb-item>study管理</el-breadcrumb-item>
		  <el-breadcrumb-item>study博客管理</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图区域 -->
		<el-card>
		  <!-- 搜索与添加区域 -->
		  <el-row :gutter="30">
		  	<el-col :span="10">
			  <el-input placeholder="请输入博客名" v-model="queryInfo.query" clearable @clear="getStudyList">
			    <el-button slot="append" icon="el-icon-search" @click="getStudyList"></el-button>
			  </el-input>
		  	</el-col>
			<el-col :span="4">
		  		<el-button type="primary" v-on:click="addDialogVisible=true" @click="goStudyWhite">添加博客</el-button>
		  	</el-col>
		  </el-row>

		  <!-- 博客列表区域 -->
		  <el-table :data="studylist" stripe>
		  	<el-table-column type="index"></el-table-column>
		  	<el-table-column label="标题" prop="inputTitle"></el-table-column>
		  	<el-table-column label="时间" prop="time"></el-table-column>
		  	<el-table-column label="封面">
				  	<template slot-scope="scope">
						<img :src="scope.row.file || blogLogo" class="fenmianImg"/>
					</template>
				  
				  <!-- 作用域插槽 -->
		  		<!-- <template slot-scope="scope">
					<el-collapse>
						<el-collapse-item title="封面图片">
							<img :src="scope.row.file" class="fenmianImg"/>
						</el-collapse-item>
					</el-collapse>
		  		</template> -->
			</el-table-column>

		  	<el-table-column label="操作" width="120px">
		  		<template slot-scope="scope">
					  <el-tooltip effect="dark" content="删除博客" placement="top" :enterable="false">
					    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removestudyById(scope.row._id)"></el-button>
					  </el-tooltip>
		  		</template>
		  	</el-table-column>
		  </el-table>

		  <!-- 分页区域 -->
	  	  <el-pagination
	        @current-change="handleCurrentChange"
	        :current-page="queryInfo.pagenum"
	        layout=" prev, pager, next, jumper">
	      </el-pagination>
		</el-card>
	</div>	
</template>


<script>
export default{
	data(){
		return{
			// 获取博客列表的参数对象
			queryInfo: {
				query: '',
				pagenum: 1
			},
			studylist: [],
			blogLogo: require("../../assets/blogLogo.png")
		}
	},


	created(){
		this.getStudyList()
	},


	methods: {
		async getStudyList(){
			const {data: res} = await this.$http.get('resource/study/studyList', { params: this.queryInfo })
			console.log(res)
			if (res.meta.status !== 200){
				if(res.meta.status === 404){
					this.studylist = res.data
					return this.$message.warning('没有更多数据！')
				}
				return this.$message.error('获取博客列表失败！')
			}
			this.studylist = res.data
		},
		// 这是监听页码值改变的事件
		handleCurrentChange(newPage){
			console.log(newPage)
			this.queryInfo.pagenum = newPage
			this.getStudyList()
		},
		// 根据id删除对应的博客信息
		async removestudyById(id){
			// 弹框询问是否确认删除, 返回值为Promise对象，可以用async await优化
			const confirmResult = await this.$confirm('此操作将永久删除该博客, 是否继续?', '提示', {
	          confirmButtonText: '确定',
	          cancelButtonText: '取消',
	          type: 'warning'
	        }).catch(err => {
	        	return err
	        })

			// 确认删除返回值为字符串confirm, 取消删除返回值为字符串cancel
	        if (confirmResult !== "confirm"){
	        	return this.$message.error('已取消删除')
	        }
	        const {data: res} = await this.$http.post('resource/study/deleteStudy', {id})
	        if(res.meta.status !== 200){
	        	return this.$message.error("删除博客失败")
	        }
	        this.$message.success("删除博客成功！")
	        this.getStudyList()
		},
		goStudyWhite(){
			this.$router.replace('/studyWhite')
			window.sessionStorage.setItem('activePath', "/studyWhite")
			this.$emit('changeActive')
			
		}
	}
}
</script>



<style>
	.fenmianImg{
		width:auto;
		height: 100px;
	}
</style>