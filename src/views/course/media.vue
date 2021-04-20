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
            :data="tableData"
            border
            :default-sort = "{prop: 'date', order: 'descending'}"
            style="width: 100%">
            <el-table-column
                prop="date"
                sortable
                label="日期"
                width="180">
            </el-table-column>
            <el-table-column
                prop="name"
                label="姓名"
                width="180">
            </el-table-column>
            <el-table-column
                prop="address"
                label="地址">
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button
                        size="mini"
                        @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button
                        size="mini"
                        type="danger"
                        @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <pagination :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" />
    </div>
</template>
<script>
import Pagination from '@/components/Pagination'
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
        tableData: [{
          date: '2016-05-02',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1518 弄'
        }, {
          date: '2016-05-04',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1517 弄'
        }, {
          date: '2016-05-01',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1519 弄'
        }, {
          date: '2016-05-03',
          name: '王小虎',
          address: '上海市普陀区金沙江路 1516 弄'
        }],
        total: 0,
        listQuery: {
            page: 1,
            limit: 20
        }
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