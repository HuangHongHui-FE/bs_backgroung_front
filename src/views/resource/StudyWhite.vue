<template>
	<div>
		<!-- 面包屑导航 -->
		<el-breadcrumb separator-class="el-icon-arrow-right">
		  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
		  <el-breadcrumb-item>学习管理</el-breadcrumb-item>
		  <el-breadcrumb-item>写学习博客</el-breadcrumb-item>
		</el-breadcrumb>

		<!-- 卡片视图 -->
		<el-card>
			<div class="maxDiv">
				<div class="steps">
					<el-steps direction="vertical" :active="index">
						<el-step title="标题" icon="el-icon-edit"></el-step>
						<el-step title="内容" icon="el-icon-upload"></el-step>
						<el-step title="封面" icon="el-icon-picture"></el-step>
						<el-step title="提交" icon="el-icon-picture"></el-step>
					</el-steps>
				</div>
				<!-- 标题 -->
				<div class="title">
					<input type="text" @blur="onblur" @keyup.enter="enter" placeholder="请输入标题" v-model="formData.inputTitle" ref="inputRef">
					<p>{{tip}}</p>
				</div>

				<div class="content">
					<mavon-editor
						v-model="formData.markdown" 
						ref="md"
						@change="change"
						:toolbars="toolbars"
						style="min-height: 550px"
					/>
				</div>

				<div class="upload" accept="image/*" @change="upload">
					<input type="file" accept="image/gif,image/jpeg,image/jpg,image/png">
				</div>

				<div class="subButton">
					<el-button @click="showSubDialog" type="primary">
						发表<i class="el-icon-upload el-icon--right"></i>
					</el-button>
				</div>

				<!-- 是否确认发表 -->
				<template>
					<el-dialog
					title="提示"
					:visible.sync="subDialogVisible"
					width="35%"
					center
					allow-create
					filterable
					:close-on-click-modal='false'
					:show-close="false">
					<span class="dialogSpan">
						确认发表:{{formData.inputTitle}}?
					</span>
					<template #footer>
						<span class="dialog-footer">
						<el-button @click="cancelSub">取 消</el-button>
						<el-button type="primary" @click="submit">确 定</el-button>
						</span>
					</template>
					</el-dialog>
				</template>
			</div>
		</el-card>
	</div>

</template>


<script>
import { mavonEditor } from 'mavon-editor'
import 'mavon-editor/dist/css/index.css'

export default{
    components: {
        mavonEditor
    },
    data(){
        return{
            formData: {
                inputTitle: '',
                // 左侧原始输入的
                markdown:'',
                // 解析成html的
                html:'',
                // 上传的封面
                file: ''
            },
            tip: '请输入',
            subDialogVisible: false,
            // 右边的步骤条
            index: 0,
            // 编辑器的配置
            toolbars: {
                bold: true, // 粗体
                italic: true, // 斜体
                header: true, // 标题
                underline: true, // 下划线
                strikethrough: true, // 中划线
                mark: true, // 标记
                superscript: true, // 上角标
                subscript: true, // 下角标
                quote: true, // 引用
                ol: true, // 有序列表
                ul: true, // 无序列表
                link: true, // 链接
                imagelink: true, // 图片链接
                code: true, // code
                table: true, // 表格
                fullscreen: true, // 全屏编辑
                readmodel: true, // 沉浸式阅读
                htmlcode: true, // 展示html源码
                help: true, // 帮助
                /* 1.3.5 */
                undo: true, // 上一步
                redo: true, // 下一步
                trash: true, // 清空
                save: false, // 保存（触发events中的save事件）
                /* 1.4.2 */
                navigation: true, // 导航目录
                /* 2.1.8 */
                alignleft: true, // 左对齐
                aligncenter: true, // 居中
                alignright: true, // 右对齐
                /* 2.2.1 */
                subfield: true, // 单双栏模式
                preview: true // 预览
            },

            options: [
                { name: "其他" }
            ]
        }
    },
    created(){

    },
    methods: {
        onblur(){
            if(this.formData.inputTitle === ''){
                this.tip = '不能为空'
            }else{
                this.tip = ''
            }
        },

        enter(){
            if(this.formData.inputTitle.length !== 0){
                this.index = 2
            }
            this.$refs.inputRef.blur()
        },

        change(value, render){
            // render 为 markdown 解析后的结果[html]
            this.formData.html = render;
        },

        // 封面图片上传
        upload(e){
            const target = e.target || e.srcElement
           	const file = target.files[0]

            if (file) {
                const reader = new FileReader();
                //将文件以Data URL形式读入页面
                reader.readAsDataURL(file);
                reader.onload = e => {
                    const AllowImgFileSize = 2100000;
                    //var ImgFileSize = reader.result.substring(reader.result.indexOf(",") + 1).length;//截取base64码部分（可选可不选，需要与后台沟通）
                    if (AllowImgFileSize < reader.result.length) {
                        return this.$messgae.console.error('上传失败，请上传不大于2M的封面!');
                    }else{
                        this.formData.file = reader.result
                    }
                }
            }
        },

        // 提交
        async submit(){
            console.log(this.formData)
			this.formData.time = new Date().toLocaleString()
            const {data: res} = await this.$http.post('resource/study/studyWrite', this.formData)
            if(res.meta.status === 200){
                this.subDialogVisible = false
                this.$message.success("发表成功！")
                // console.log(this.$router)
                this.$router.replace("/studyDelete")
				
				window.sessionStorage.setItem('activePath', "/studyDelete")
				this.$emit('changeActive')
            }else{
                return this.$message.error("发表失败！")
            }
        },
        // 提交前的提示
        showSubDialog(){
            if(this.formData.inputTitle.length === 0){ return this.$message.warning("请填写题目后再发表！") }
            if(this.formData.markdown.length === 0){ return this.$message.warning("请填写内容后再发表！") }
            this.index = 4
            this.subDialogVisible = true
        },
        // 取消了提交
        cancelSub(){
            this.subDialogVisible = false
            this.index = 3
            this.$message.warning("取消了发表")
        }
    },
    watch:{
        'formData.inputTitle': function(newVal) {
            if(newVal !== '' && newVal.length !== 0){
                if(this.index < 1){
                    this.index = 1
                }
            }else{
                this.index = 0
            }
        },
        'formData.html': function(newVal){
            if(this.formData.inputTitle === '' || this.formData.inputTitle.length === 0){
                this.index = 0
                return
            }
            if(newVal.length !== 0 && newVal !== ''){
                if(this.index < 2){
                    this.index = 2
                }
            }else{
                this.index = 1
            }
        },
        'formData.file': function(newVal){
            if(this.formData.inputTitle === '' || this.formData.inputTitle.length === 0){
                this.index = 0
                return
            }
            if(this.formData.html === '' || this.formData.html.length === 0){
                this.index = 1
                return
            }
            if(newVal !== ''){
                if(this.index < 3){
                    this.index = 3
                }
            }else{
                this.index = 2
            }
        }
    }
}
</script>


<style lang="less" scoped>
    .maxDiv{
        .steps{
            position: fixed;
            top: 15%;
            right: 0%;
            width: 10%;
            height: 80%;
        }
        .title{
            height: 50px;
            line-height: 50px;
            input{
                vertical-align: middle;
                width: 75%;
                height: 80%;
                border-radius: 20px;   
                outline: none;
                font-size: 20px;
                padding-left: 15px;
            }
            p{
                margin-left: 3%;
                display: inline;
                font-size: 16px;
                color: red;
                width: 5%;
            }
            
        }
        .content{
            margin-top: 1%;
            width: 85%;
            overflow: auto;
        }
        .upload{
            display: inline-block;
            width: 40%;
            overflow: hidden;
            input{
                display: inline-block;
                color: rgb(236, 57, 37);
                // background-color: deeppink;
            }
        }
        .subButton{
            display: inline-block;
            margin-left: 5%;
            background-color: firebrick;
        }
    }
</style>