<template>
    <div class="container">
        <!-- 头部 -->
        <div class="header">
            <el-button type="primary" icon="el-icon-edit">新增图文</el-button>
            <div style="float:right">
                <el-select v-model="value" placeholder="请选择" style="margin-right:10px;width:100px">
                    <el-option
                        v-for="item in options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                    </el-option>
                </el-select>
                <el-input placeholder="标题" style="width:200px"></el-input>
                <el-button type="primary" icon="el-icon-search">搜索</el-button>
            </div>
        </div>
        
        <!-- 表格 -->
        <el-table
            :data="list"
            border
            :default-sort = "{prop: 'id', order: 'descending'}"
            style="width: 100%;">
            <el-table-column
                prop="id"
                sortable
                label="ID"
                width="80"
                align="center">
            </el-table-column>
            <el-table-column
                label="图文内容">
                <template slot-scope="scope">
                    <div style="display:flex;flex-direction:row">
                        <img style="height:50px" :src='scope.row.cover' alt="">
                        <div style="margin-left:10px">
                            <span>{{scope.row.title}}</span><br>
                            <span style="color:red">￥{{scope.row.price}}</span>
                        </div>
                    </div>   
                </template>
            </el-table-column>
            <el-table-column
                prop="sub_count"
                label="订阅量"
                align="center"
                width="100">
            </el-table-column>
            <el-table-column
                label="状态"
                align="center"
                width="100">
                <template slot-scope="scope">
                    <el-tag :type="scope.row.status===1?'success':'danger'">
                        {{scope.row.status===0?'已下架':'已上架'}}
                    </el-tag>
                </template>
            </el-table-column>
            <el-table-column
                prop="created_time"
                label="创建时间"
                align="center"
                width="150">
            </el-table-column>
            <el-table-column label="操作" align="center" width="250">
                <template>
                    <el-button
                        size="mini"
                        >编辑</el-button>
                    <el-button
                        size="mini"
                        type="danger"
                        >删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <pagination :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList"/>
    </div>
</template>
<script>
import Pagination from '@/components/Pagination'
import {
    fetchList
} from '@/api/media'
export default {
    name:"media",
    components: { Pagination },
    data() {
      return {
        options: [{
          value: '选项1',
          label: '黄金糕'
        }, {
          value: '选项2',
          label: '双皮奶'
        }],
        value: '',
        list:null,//表格数据
        total: 0,
        listQuery: {
            page: 1,
            limit: 20
        }
      }
    },
    created() {
        this.getList()
    },
    methods:{
        //获取表格内容
        getList(){
            fetchList(this.listQuery).then(response => {
                // console.log(response);
                this.list = response.data.items
                this.total = response.data.total
            })
        }
    }
}
</script>
<style scoped>
    .container{
        padding: 20px;
    }
    .header{
        padding-left: 10px;
        margin-bottom: 20px;
    }
</style>