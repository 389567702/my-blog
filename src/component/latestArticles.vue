<template>
    <div>
                <div class="grid-content bg-purple">
                    <div class="tagtitle">
                        <p :class="{'fadetitle':fadetitle}" style="margin-left:30px;">最新</p>
                    </div>
                    <el-tabs>
                        <el-row :gutter="20">
                                <el-col :xs="24" :sm="24" :md="12" :lg="12"  v-for="item in articleList" :key="item._id" class="artitem" >
                                    <div  @click="articlesDetailsFn(item._id)">
                                        <div class="box-card articles-box">
                                            <div class="post-time">
                                                <span class="post-timecon">{{new Date(item.date).format('yyyy-MM-dd')}}</span>
                                            </div>
                                            <div class="post-title">
                                                <h1>{{item.title}}</h1>
                                            </div>
                                            <div class="post-abstract" v-compiledMarkdown>
                                                {{item.articleContent}}
                                            </div>
                                        </div>
                                        <div class="artitem_bottom">
                                            <div class="avatar" v-for="list in item.user">
                                                <a :href=list.html_url target="_blank">
                                                    <img :src=list.avatar_url alt="">
                                                    <p>{{list.name}}</p>
                                                </a>
                                            </div>
                                            <div class="post-label-box">
                                                <span class="post-label" v-for="list in item.label">{{list}}</span>
                                            </div>
                                        </div>
                                    </div>
                                </el-col>
                                <!--<button style="background-color:red; width: 200px;height:40px; color:#fff;" @click="nextpage">下1一页</button>-->
                                <!--<el-col :xs="24" :sm="12" :md="12" :lg="12" v-for="item in articleList" :key="item._id">-->
                                    <!--<el-card class="box-card articles-box">-->
                                        <!--<div class="post-title" @click="articlesDetailsFn(item._id)">-->
                                            <!--{{item.title}}-->
                                        <!--</div>-->
                                        <!--<div class="post-time">-->
                                            <!--{{new Date(item.date).format('yyyy-MM-dd hh:mm:ss')}}-->
                                            <!--<span class="post-label">{{item.label}}</span>-->
                                        <!--</div>-->
                                        <!--<div class="post-abstract" v-compiledMarkdown>-->
                                            <!--{{item.articleContent}}。-->
                                        <!--</div>-->
                                    <!--</el-card>-->
                                <!--</el-col>-->
                                <div class="scrollbottomtip">
                                    <p :class="{ scrolltip: scrolltip }" style="position:relative;top:-15px;height:24px;">滚动加载更多</p>
                                    <div :class="{scrollload:scrollload,scrollloadlast:scrollloadlast}" >
                                        <p>数据加载中</p>
                                        <i class="el-icon-loading"></i>
                                    </div>
                                </div>
                            </el-row>
                            <p :class="{'hide':lastpage}" class="lastpagetip">哼！我也是有底线的...</p>
                    </el-tabs>
                </div>
            <!--</el-col>-->
        <!--</el-row>-->
    </div>
</template>

<script>
import { Loading } from 'element-ui';
import marked from 'marked';
import highlight from 'highlight.js'
import '../assets/atom-one-light.css'
export default {
    name: 'latestArticles',
    data(){
        return {
            articleList: [],
            page:0,
            lastpage:true,
            first:true,
            ScrollFirst:true,
            scrolltip:true,
            scrollload:true,
            scrollloadlast:false,
            scrollpage:true,
            fadetitle:false,
            tagtitle:'',
        }
    },
    created(){


    },
    mounted(){
//        this.$root.eventbus.$on('tabname',(target) => {
//            console.log("lasttt")
//            console.log(target)
//        this.tagtitle=target;
//        console.log(this.tagtitle)
//    });

        this.fadetitle=false;
        this.articleList=this.$store.state.newlistcon;
        console.log(this.$store.state.taglistfirst)
        document.title ='binlive-最新';
        if(this.$store.state.newlistfirst){
            let loadingInstance = Loading.service({ fullscreen: true });
            this.$http.get('/api/articleList').then(
                    res => {
                console.log('heer')
            if(res.body.length<20){
                this.scrolltip= true
            }
            this.articleList = res.body;
            loadingInstance.close();
            this.fadetitle= true,
//            this.tagtitle= res.body[0].tag
            this.$store.commit('updatenewlistcon',this.articleList)
            this.first=false
            this.$store.commit('newlistfirst',this.first);
            // let a = res.body[2].articleContent.replace(/[^\u4e00-\u9fa5]/gi,'')
        },
            res => {
                console.log(res)
            }
        );

        }
        Date.prototype.format = function(format) {
            var o = {
                "M+": this.getMonth() + 1, //month
                "d+": this.getDate(), //day
                "h+": this.getHours(), //hour
                "m+": this.getMinutes(), //minute
                "s+": this.getSeconds(), //second
                "q+": Math.floor((this.getMonth() + 3) / 3), //quarter
                "S": this.getMilliseconds() //millisecond
            }
            if (/(y+)/.test(format)) {
                format = format.replace(RegExp.$1,(this.getFullYear() + "").substr(4 - RegExp.$1.length));
            }
            for (var k in o){
                if (new RegExp("(" + k + ")").test(format)){
                    format = format.replace(RegExp.$1,RegExp.$1.length == 1 ? o[k] :("00" + o[k]).substr(("" + o[k]).length));
                }
            }
            return format;
        }
        marked.setOptions({
            renderer: new marked.Renderer(),
            gfm: true,
            tables: true,
            breaks: false,
            pedantic: false,
            sanitize: false,
            smartLists: true,
            smartypants: false,
            highlight: function (code) {
                return highlight.highlightAuto(code).value;
            }
        })

//        监听滚动
        if(this.scrollpage){
            window.addEventListener('scroll', this.handleScroll);
        }
    },
    watch:{
        "$route": "scrollpageFn"
    },
    methods: {
        scrollpageFn(){
            let tag = this.$route.path
            if(tag=='/'){
                this.scrollpage=true
            }
        },
        articlesDetailsFn: function(id){
            console.log("12123")
            this.scrollpage=false
            this.$router.push({ name: 'details', params: { id: id }})
            // this.$router.push('articlesDetails'+id+'')
        },
//        分页
        nextpage: function(){
            if(this.lastpage){
                this.page++
                var page=this.page;
                this.$http.get('/api/articleList/'+page).then(
                        res => {
                    console.log(res.body)
                let arrcon=this.articleList
                let arrnew=res.body
                let arrconlast=arrcon.concat(arrnew);
                this.articleList=arrconlast
                console.log("这里")
                console.log(arrconlast)
                console.log(this.articleList.length)
                this.ScrollFirst=true;
                this.scrolltip=false;
                this.scrollload=true;
                if(res.body.length<10){
                    this.lastpage=false;
                    this.scrollloadlast=true;
                }
                this.$store.commit('updatenewlistcon',this.articleList);
                this.articleList=this.$store.state.newlistcon
                console.log(this.$store.state.newlistcon)
                },
                    res => {
                        console.log(res)
                    }
                )
            }
        },
//        监听滚动
        handleScroll () {
                let scrollBottom=document.body.clientHeight-window.innerHeight -document.body.scrollTop
                if(scrollBottom<30&&this.scrollpage){
                    console.log(this.scrollpage)
                    console.log(scrollBottom)
                    if(this.ScrollFirst){
                        this.scrolltip=true;
                        this.scrollload=false;
                        this.ScrollFirst=false;
                        this.nextpage()
                    }
                }
        }
    },
    directives: {
        compiledMarkdown: {
            bind: function(el){
                el.innerHTML = marked(el.innerText)
                el.innerHTML = el.innerHTML.replace(/[^\u4e00-\u9fa5]/gi,'')

                if(el.querySelector('pre')){
                    el.querySelector('pre').style.display = "none"
                }
                if(el.querySelector('blockquote')){
                    el.querySelector('blockquote').style.display = "none"
                }
            }
        }
    },
}
</script>

<style scoped>
@import '../style/latestArticles.css';
</style>
