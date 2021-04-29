<template>
  <div class="app-container">
    <!-- 专栏详细信息 -->
    <el-card shadow="never" v-model="detailData">
      <div style="display: flex; flex-direction: row">
        <img style="height: 100px;margin-right:10px" :src='detailData.cover' alt="" />
        <div style="margin-left: 10px">
          <h4 style="margin-top:5px;margin-bottom:0px">
            {{detailData.title}}
            <span style="float:right;font-size:80%;color:#BBB;font-weight:normal">{{detailData.isend?'已完结':'连载中'}}</span>
          </h4>
          <p style="font-size:80%;color:#BBB;margin:8px 0">{{detailData.content}}</p>
          <p style="color: red">￥{{detailData.price}}</p>
          <el-button type="warning" @click="handleChangeStatus(detailData.status)">{{detailData.status?'下架':'上架'}}</el-button>
          <el-button @click="handleChangeisend(detailData.isend)">{{detailData.isend?'设置为连载中':'设置为已完结'}}</el-button>
        </div>
      </div>
    </el-card>
     <!-- 头部 -->
    <div class="header">
      <el-button type="primary" icon="el-icon-edit"
        >新增目录</el-button
      >
      <div style="float: right">
        <el-select style="margin-right: 10px; width: 100px" v-model="listQuery.status" placeholder="商品状态" clearable class="filter-item">
          <el-option v-for="item in options" :key="item.index" :label="item" :value="item" />
        </el-select>
        <el-input
          v-model="listQuery.title"
          @keyup.enter.native="handleFilter"
          placeholder="标题"
          style="width: 200px"
        ></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleFilter"
          >搜索</el-button
        >
      </div>
    </div>
    <!-- Note that row-key is necessary to get a correct row order. -->
    <el-table ref="dragTable" v-loading="listLoading" :data="list" row-key="id" border fit highlight-current-row style="width: 100%">
      <el-table-column type="index" align="center"  width="65">
        <!-- <template slot-scope="scope">
          <span>{{ scope.row.id }}</span>
        </template> -->
      </el-table-column>

      <el-table-column label="单品内容">
        <template slot-scope="{row}">
          <div style="display: flex; flex-direction: row">
            <img style="height: 50px" :src="row.cover" alt="" />
            <div style="margin-left: 10px">
              <span>{{ row.title }}</span
              ><br />
              <span style="color: red">￥{{ row.price }}</span>
            </div>
          </div>
        </template>
      </el-table-column>

      <el-table-column width="110px" align="center" label="订阅量">
        <template slot-scope="{row}">
          <span>{{ row.sub_count }}</span>
        </template>
      </el-table-column>

      <el-table-column class-name="status-col" label="状态" width="110">
        <template slot-scope="{row}">
          <el-tag :type="row.status === 1 ? 'success' : 'danger'">
            {{ row.status === 0 ? "已下架" : "已上架" }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
            >编辑</el-button
          >
          <el-popconfirm
            title="这是一段内容确定删除吗？"
            style="margin-left: 10px"
            @onConfirm="handleDelete(scope.$index, scope.row)"
          >
            <el-button slot="reference" size="mini" type="danger"
              >删除</el-button
            >
          </el-popconfirm>
        </template>
      </el-table-column>
      <el-table-column align="center" label="Drag" width="80">
        <template slot-scope="{}">
          <svg-icon class="drag-handler" icon-class="drag" />
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
// import { fetchList } from '@/api/article'
import Sortable from 'sortablejs'
import {fetchDetail,fetchDetailCourse,deleteColumn} from '@/api/column'

export default {
  name: 'columnDeatil',
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      options: [ "已上架", "已下架"],
      list: [],
      total: null,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 10
      },
      sortable: null,
      oldList: [],
      newList: [],
      detailData:null,
    }
  },
  created() {
    this.getList();
    const row=this.$route.params
    this.fetchData(row)
  },
  methods: {
    handleFilter() {
        this.listQuery.page = 1;
        this.getList();
    },
    async getList() {
      this.listLoading = true
      const { data } = await fetchDetailCourse(this.listQuery)
      this.list = data.items
      this.total = data.total
      this.listLoading = false
      this.oldList = this.list.map(v => v.id)
      this.newList = this.oldList.slice()
      this.$nextTick(() => {
        this.setSort()
      })
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
      
      this.sortable = Sortable.create(el, {
        ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
        setData: function(dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData('Text', '')
        },
        onEnd: evt => {
          const targetRow = this.list.splice(evt.oldIndex, 1)[0]
          this.list.splice(evt.newIndex, 0, targetRow)

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0]
          this.newList.splice(evt.newIndex, 0, tempIndex)
        }
      })
    },
    //获取详细信息
    async fetchData(row){
      console.log(row);
      let rs = await fetchDetail(row)
      // console.log(rs);
      this.detailData=rs.data
      // console.log(this.detailData);
    },
    //上架下架
    handleChangeStatus(status){
      if(status===1){
        this.detailData.status=0
      }else{
        this.detailData.status=1
      }
      // console.log(this.detailData.status);
    },
    handleChangeisend(isend){
      if(isend===1){
        this.detailData.isend=0
      }else{
        this.detailData.isend=1
      }
      // console.log(this.detailData.isend);
    },
    //删除
    handleDelete(index, row) {
      deleteColumn(row.id).then((response) => {
        // console.log(response);
          this.$notify({
            title: "Success",
            message: "删除成功",
            type: "success",
            duration: 2000,
          });
          this.list.splice(index, 1);
      });
    },
  }
}
</script>

<style>
.sortable-ghost{
  opacity: .8;
  color: #fff!important;
  background: #42b983!important;
}
.icon-star{
  margin-right:2px;
}
.drag-handler{
  width: 20px;
  height: 20px;
  cursor: pointer;
}
.show-d{
  margin-top: 15px;
}
.header {
  padding-left: 10px;
  margin: 20px 0;
}
</style>
