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
            fit
            highlight-current-row
            @sort-change="sortChange"
            style="width: 100%;">
            <el-table-column
                prop="id"
                label="ID"
                sortable
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
                <template slot-scope="scope">
                    <el-button
                        size="mini"
                        type="primary"
                        @click="handleEdit(scope.$index, scope.row)"
                        >编辑</el-button>
                    <el-button
                        size="mini"
                        :type="scope.row.status===1?'':'success'"
                        @click="handleChangeStatus(scope.row)"
                        >{{scope.row.status===1?'下架':'上架'}}</el-button>
                    <el-popconfirm
                        title="这是一段内容确定删除吗？"
                        style="margin-left:10px"
                        @onConfirm="handleDelete(scope.$index, scope.row)"
                    >
                        <el-button slot="reference" size="mini" type="danger">删除</el-button>
                    </el-popconfirm>
                    
                </template>
            </el-table-column>
        </el-table>
        <pagination :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList"/>
    </div>
</template>
<script>
import Pagination from '@/components/Pagination'
import {
    fetchList,
    createMedia,
    updateMedia,
    deleteMedia
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
        handleFilter() {
            this.listQuery.page = 1
            this.getList()
        },
        sortChange(data) {
            const { prop, order } = data
            if (prop === 'id') {
                this.sortByID(order)
            }
        },
        sortByID(order) {
            if (order === 'ascending') {
                this.listQuery.sort = '+id'
            } else {
                this.listQuery.sort = '-id'
            }
            this.handleFilter()
        },
        //获取表格内容
        getList(){
            fetchList(this.listQuery).then(response => {
                // console.log(response);
                this.list = response.data.items
                this.total = response.data.total
            })
        },
        handleEdit(index, row) {
            console.log(index, row);
        },
        //上架下架
        handleChangeStatus(row){
            if(row.status===0){
                row.status=1
                this.$message({
                    message: '商品已上架',
                    type: 'success'
                });
            }else if(row.status===1){
                row.status=0
                this.$message({
                    message: '商品已下架',
                    type: 'success'
                });
            }
        },
        //删除
        handleDelete(index,row){
            deleteMedia(row.id).then(response=>{
                console.log(response);
                if(response.code===20000){
                    this.$notify({
                        title: 'Success',
                        message: '删除成功',
                        type: 'success',
                        duration: 2000
                    })
                    this.list.splice(index, 1)
                }
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