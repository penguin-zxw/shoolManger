<template>
  <div class="container">
    <!-- 头部 -->
    <div class="header">
      <el-button type="primary" icon="el-icon-edit" @click="handleCreate"
        >新增子页面</el-button
      >
    </div>
    <el-table
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%"
    >
      <el-table-column label="页面内容" min-width="180px">
        <template slot-scope="{ row }">
          <span>{{ row.title }}</span>
          <span
            v-if="row.type == 'index'"
            class="el-icon-s-home"
            style="color: blue"
          ></span>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" width="180px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.created_time }}</span>
        </template>
      </el-table-column>
      <el-table-column label="更新时间" width="180px" align="center">
        <template slot-scope="{ row }">
          <span>{{ row.updated_time }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        width="230"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="{ row, $index }">
          <el-button type="warning" size="mini" @click="openDetail(row)">
            装修
          </el-button>
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            编辑
          </el-button>
          <el-popconfirm
            title="是否要删除该记录？"
            @onConfirm="handleDelete(row, $index)"
            style="margin-left: 10px"
          >
            <el-button
              v-if="row.status != 'deleted'"
              size="mini"
              type="danger"
              slot="reference"
              :disabled="!!row.isdefault"
              >删除</el-button
            >
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <pagination
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="90px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="页面标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button
          type="primary"
          @click="dialogStatus === 'create' ? createData() : updateData()"
        >
          提交
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import { fetchMobileList,updateMobile,createMobile } from "@/api/renovation";
export default {
  name: "MobileIndex",
  components: { Pagination },
  data() {
    return {
      listLoading: true,
      list: null, //表格数据
      total: 0,
      listQuery: {
        page: 1,
        limit: 20,
      },
      dialogFormVisible: false,
      dialogStatus: "",
      temp: {
        title: "",
      },
      textMap: {
        update: "编辑",
        create: "新增",
      },
      rules: {
        title: [
          { required: true, message: "页面标题不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    getList() {
      fetchMobileList(this.listQuery).then((response) => {
        // console.log(response);
        this.listLoading = false;
        this.list = response.data.items;
        this.total = response.data.total;
      });
    },
    resetTemp() {
      this.temp = {
        title: "",
      };
    },
    //添加
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(()=>{
          this.$refs['dataForm'].clearValidate()
      })
    },
    createData(){
        this.$refs['dataForm'].validate(valid=>{
            if(valid){
                createMobile(this.temp).then(()=>{                  
                    this.temp.created_time=new Date();
                    console.log(this.temp);
                    this.list.unshift(this.temp)
                    this.dialogFormVisible=false
                    this.$notify({
                        title:'成功',
                        message:"创建成功",
                        type:'success',
                        duration:1000
                    })

                })
            }
        })
    },
    //更新
    handleUpdate(row) {
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.temp = Object.assign({}, row)
      this.temp.updated_time=new Date(this.temp.updated_time)
      this.$nextTick(()=>{
          this.$refs['dataForm'].clearValidate()
      })
    },
    updateData(){
        this.$refs['dataForm'].validate(valid=>{
            if(valid){
                const tempData=Object.assign({}, this.temp)
                updateMobile(this.tempDatap).then(()=>{
                    this.dialogFormVisible=false
                    this.$notify({
                        title:'成功',
                        message:"修改成功",
                        type:'success',
                        duration:1000
                    })
                })
            }
        })
    },
    //装修
    openDetail(row) {
      this.$router.push({
        name: "MobileEdit",
        query: {
          id: row.id,
        },
      });
    },
  },
};
</script>
<style scoped>
.container {
  padding: 20px;
}
.header {
  padding-left: 10px;
  margin-bottom: 20px;
}
</style>