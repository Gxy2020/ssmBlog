<template>
    <div>
        <div class="markdown" >
            <el-row :gutter="30" >
                <el-col :span="8">
                    <el-input  v-model="blog.title" placeholder="请输入标题" />
                </el-col>
                <el-col :span="6">
                    <el-select v-model="name" :placeholder="uname" >
                        <el-option v-for="item in lab"
                                   :key="item.labId"
                                   :label="item.name"
                                   :value="item.name">
                        </el-option>
                    </el-select>
                </el-col>
                <el-col :span="6">
                    <el-button type="primary" >返回</el-button>
                </el-col>
            </el-row>
            <mavon-editor
                    v-model="blog.markdown"
                    ref="md"
                    @change="change"
                    @save="saveMavon"
                    style="min-height: 600px;margin-top: 15px"/>
            <div class="sub">
                <el-button type="success" style="margin-top: 5px" @click="submit" >提交</el-button>
            </div>
        </div>
    </div>
</template>

<script>
    import { mavonEditor } from 'mavon-editor'
    import 'mavon-editor/dist/css/index.css'
    export default {
        name: "Edit",
        components: {
            mavonEditor,
        },
        data(){
            return{
                html: '',
                blog:'',
                name:'',
                lab:[],
                id:this.$route.params.id,
                uname:''
            }
        },
        created() {
            this.getLab();
            // console.log(this.id);
            this.getBlog();
        },
        mounted() {
        },
        methods:{
            getBlog(){
               // console.log(this.$route.params.blog==null)
              const  that=this;
                if(that.$route.params.blog==null){
                        that.$axios.get('selectBlogById/'+that.id).then((res)=>{
                           that.blog=res.data.data.blog
                            that.uname=that.blog.lab.name;
                            // console.log(res.data.data.blog);
                            // console.log(that.uname)
                        })
                }else {
                    that.blog=JSON.parse(this.$route.params.blog);
                    // console.log(that.blog);
                    that.uname=that.blog.lab.name;
                }
                // console.log(this.blog)
            },
            saveMavon(value, render){
                // console.log(value);   // md语法
                // console.log(render);  //html内容
                this.readmeContent=render;
            },
            change(value, render){
                // render 为 markdown 解析后的结果[html]
                this.html = render;
            },
            getLab(){
                let that=this;
                this.$axios.get('findLabs').then((res)=> {
                    const result=res.data;
                    // console.log(result);
                    if (result.code==200){
                        that.lab=result.data.labList;
                        // console.log(result.data.labList)
                    }
                }).catch(reason =>
                    console.log(reason));
                // console.log()
            },
            submit(){
                console.log('提交操作')
                console.log(this.blog)
                if (this.blog.title !==undefined && this.blog.title!=''&&this.name!=undefined&&this.name.length>0&&
                    this.blog.markdown!=undefined&&this.blog.markdown!=''){
                    console.log('都不为空,可以发送请求');
                const that=this
                this.$axios({
                    url:'updateBlog/'+that.name,
                    method:'put',
                    data:{
                        'id':that.blog.id,
                        'title':that.blog.title,
                        'markdown': that.blog.markdown,
                        'html':'<div class=blogDetail>'+that.html+'</div>'
                    }
                }).then((res)=>{
                    const data=res.data;
                    if (data.code==200){
                        this.$message.success("发布成功");
                    }
                })
                }else {
                    this.$message.warning("未填写标题或未选择标签");
                }
                // console.log(this.blog.id)
                // console.log(this.blog.title)
                // console.log(this.name)
                // console.log(this.blog.markdown);
                // console.log('<div class=blogDetail>'+this.html+'</div>');
            },
        },

    }
</script>

<style scoped lang="less">
    .markdown{
        margin-top: 10px;
    }
    .sub{
        display: flex;
        .el-button{
            margin: auto;
        }
    }
</style>
