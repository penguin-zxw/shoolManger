<template>
  <div class="container">
    <!-- 头部 -->
    <div class="header">
      <el-dropdown>
        <el-button type="danger">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item>禁止评论</el-dropdown-item>
          <el-dropdown-item>禁止访问</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <div style="float: right">
        <el-input
          v-model="listQuery.title"
          @keyup.enter.native="handleFilter"
          placeholder="用户名"
          style="width: 200px"
        ></el-input>
        <el-button type="primary" icon="el-icon-search" @click="handleFilter"
          >搜索</el-button
        >
      </div>
    </div>

    <!-- 表格 -->
    <el-table
      :data="list"
      border
      style="width: 100%"
    >
        <el-table-column
        type="selection"
        width="55">
        </el-table-column>
      <el-table-column label="用户">
          <template slot-scope="{row}">
              <div style="display:flex;align-items: center;">
                  <img style="border-radius:50%;height:50px;margin-right:10px" :src="row.user.avatar" alt="">
                  <span>{{row.user.username}}</span>
              </div>
              
          </template>
      </el-table-column>
      <el-table-column
        prop="total_consume"
        label="消费总额"
        align="center"
        width="100"
      >
      </el-table-column>
      <el-table-column
        prop="created_time"
        label="创建时间"
        align="center"
        width="150"
      >
      </el-table-column>
      <el-table-column width="300" label="操作" align="center">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" @click="handleUpdate(scope.row)"
            >详情</el-button
          >
          <el-button
            size="mini"
            type="success"
            @click="handleChangeStatus(scope.row)"
            >联系用户</el-button
          >
          <el-dropdown>
        <el-button type="danger" size="mini" style="margin-left:10px">
          黑名单设置<i class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item>禁止评论</el-dropdown-item>
          <el-dropdown-item>禁止访问</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
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
    <!-- 编辑与添加面板 -->
    
    
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import Dropzone from "@/components/Dropzone";
import Tinymce from "@/components/Tinymce";
import { fetchList } from "@/api/user";
export default {
  name: "media",
  components: { Pagination, Dropzone, Tinymce },
  data() {
    return {
      list: null, //表格数据
      total: 0,
      listQuery: {
          title:'',
        page: 1,
        limit: 20,
      },
      statusOptions: [0, 1],
      temp: {
        id: "",
        title: "",
        cover: "",
        try: "",
        content: "",
        price: "0",
        sub_count: "",
        status: 0,
      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "Edit",
        create: "Create",
      },
      rules: {
        title: [{ required: true, message: "标题不能为空", trigger: "blur" }],
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    handleFilter() {
      this.listQuery.page = 1;
      this.getList();
    },
    //获取表格内容
    getList() {
      fetchList(this.listQuery).then((response) => {
        console.log(response);
        this.list = response.data.items;
        this.total = response.data.total;
      });
    },
    handleEdit(index, row) {
      console.log(index, row);
    },
    //上架下架
    handleChangeStatus(row) {
      if (row.status === 0) {
        row.status = 1;
        this.$message({
          message: "商品已上架",
          type: "success",
        });
      } else if (row.status === 1) {
        row.status = 0;
        this.$message({
          message: "商品已下架",
          type: "success",
        });
      }
    },
    //删除
    handleDelete(index, row) {
      
    },
    //添加
    handleCreate() {
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
    },
    //上传图片
    dropzoneS(file) {
      console.log(file);
      this.$message({ message: "Upload success", type: "success" });
    },
    dropzoneR(file) {
      console.log(file);
      this.$message({ message: "Delete success", type: "success" });
    },
    //新增数据
    createData() {
      
    },
    //编辑
    updateData() {
      
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
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