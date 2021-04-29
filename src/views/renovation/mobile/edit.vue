<template>
    <div style="padding:20px 30px;background:#eee">
        <el-row :gutter="20">
            <!-- 左侧 -->
            <el-col :span='6' :offset="0">
                <el-card style="height:80vh">
                    <div slot="header">
                        <div>组件列表</div>
                        <span style="font-size:10px;color:#bbb">点击组件，添加至页面</span>
                    </div>
                    <el-row :gutter="20">
                        <el-col :span="12" :offset="0" v-for="(item,index) in components" :key="index" style="margin-bottom:15px">
                            <el-card shadow="hover" :body-style="{padding:'15px 8px','cursor':'pointer'}">
                                <div style="display:flex;align-items:center">
                                    <i :class="item.icon" style="font-size:25px;color:#13ce66;margin-right:8px"></i>
                                    <span style="font-size:13px">{{item.title}}</span>
                                </div>
                            </el-card>
                        </el-col>
                    </el-row>
                </el-card>
            </el-col>
            <!-- 中间 -->
            <el-col :span="8" :offset="0">
                <el-card style="min-height:80vh;overflow:inherit" :body-style="{padding:'10px 8px'}">
                    <img src="./status.png" style="width:100%;margin-bottom:10px">
                    <div style="margin-left:-10px;margin-right:-10px">
                        <div class="checked-box"
                            :class="item.checked?'checked-box-active':''"
                            v-for="(item,index) in templates"
                            :key="index"
                            @click="handleChecked(item)"
                        >
                            <!-- 操作按钮 @click.stop阻止时间冒泡 -->
                            <div v-if="item.checked" class="checked-box-btns">
                                <i class="el-icon-top" :class="index==0?'i-disabled':''" @click.stop="moveUp(index)"></i>
                                <i class="el-icon-bottom" :class="(index+1)==templates.length?'i-disabled':''" @click.stop="moveDown(index)"></i>
                                <i class="el-icon-delete" @click="deleteComponent(index)"></i>
                            </div>
                            <div style="height:100px;border:1px solid;display:flex;align-items:center;justify-content:center;">框{{item.id}}</div>
                        </div>
                    </div>
                </el-card>
            </el-col>
        </el-row>
    </div>
</template>
<script>
import util from '@/utils/utils.js'
export default {
    name:'MobileEdit',
    data(){
        return{
            components:[
                {
                  icon:"el-icon-s-order",
                  title:'课程列表'  
                },
                {
                  icon:"el-icon-search",
                  title:'搜索框'  
                },
                {
                  icon:"el-icon-s-help",
                  title:'轮播图'  
                },
                {
                  icon:"el-icon-menu",
                  title:'图标分类'  
                },
                {
                  icon:"el-icon-s-finance",
                  title:'优惠券'  
                },
                {
                  icon:"el-icon-s-order",
                  title:'限时拼团'  
                },
                {
                  icon:"el-icon-picture-outline",
                  title:'图片广告'  
                },
            ],
            templates: [{
                id: 1,
                checked: false
            }, {
                id: 2,
                checked: false
            }, {
                id: 3,
                checked: false
            }, {
                id: 4,
                checked: false
            }]
        }
    },
    methods:{
        //选中
        handleChecked(row){
            this.templates.map(v => {
                v.checked = false
                return v
            })
            row.checked = true
        },
        //上移
        moveUp(index){
            if(index==0){
                return
            }
            util.moveUp(this.templates,index)
        },
        //下移
        moveDown(index){
            if(index==(this.templates.length-1)){
                return
            }
            util.moveDown(this.templates,index)
        },
        //删除
        deleteComponent(index){
            this.$confirm('是否要删除该组件？','提示',{
                confirmButtonText:'删除',
                cancelButtonText:'取消',
                type:'warning'
            }).then(()=>{
                this.templates.splice(index,1);
                this.$message({
                    type:'success',
                    message:"删除成功"
                })
            }).catch(() => {});  
        }
    },
}
</script>
<style>
    .checked-box {
        cursor: pointer;
        position: relative;
    }
    
    .checked-box-active {
        border: 2px dotted #2196f3;
        padding: 5px 0;
        margin-bottom: 10px;
    }
    
    .checked-box-btns {
        position: absolute;
        display: flex;
        flex-direction: column;
        background-color: #ffffff;
        border: 2px dotted #2196f3;
        right: -29px;
        top: -2px;
        z-index: 999;
        border-left-width: 0;
    }
    
    .checked-box-btns i {
        padding: 4px 6px;
        color: #2196f3;
        font-weight: bold;
    }
    
    .checked-box-btns i:hover {
        background-color: #eeeeee;
    }
    
    .checked-box-btns .i-disabled {
        background-color: #ffffff;
        cursor: no-drop;
        color: #bbbbbb;
    }
</style>