<template>
  <div class="container">
    <!-- 头部 -->
    <div class="header">
      <el-button type="primary" icon="el-icon-edit" @click="handleCreate"
        >新增音频</el-button
      >
      <div style="float: right">
        <el-select
          v-model="value"
          placeholder="请选择"
          style="margin-right: 10px; width: 100px"
        >
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
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

    <!-- 表格 -->
    <el-table
      :data="list"
      border
      fit
      highlight-current-row
      @sort-change="sortChange"
      style="width: 100%"
    >
      <el-table-column prop="id" label="ID" sortable width="80" align="center">
      </el-table-column>
      <el-table-column label="音频内容">
        <template slot-scope="scope">
          <div style="display: flex; flex-direction: row">
            <img style="height: 50px" :src="scope.row.cover" alt="" />
            <div style="margin-left: 10px">
              <span>{{ scope.row.title }}</span
              ><br />
              <span style="color: red">￥{{ scope.row.price }}</span>
            </div>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        prop="sub_count"
        label="订阅量"
        align="center"
        width="100"
      >
      </el-table-column>
      <el-table-column label="状态" align="center" width="100">
        <template slot-scope="scope">
          <el-tag :type="scope.row.status === 1 ? 'success' : 'danger'">
            {{ scope.row.status === 0 ? "已下架" : "已上架" }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="created_time"
        label="创建时间"
        align="center"
        width="150"
      >
      </el-table-column>
      <el-table-column label="操作" align="center" width="250">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="primary"
             @click="handleUpdate(scope.row)"
            >编辑</el-button
          >
          <el-button
            size="mini"
            :type="scope.row.status === 1 ? '' : 'success'"
            @click="handleChangeStatus(scope.row)"
            >{{ scope.row.status === 1 ? "下架" : "上架" }}</el-button
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
    </el-table>
    <!-- 分页 -->
    <pagination
      :total="total"
      :page.sync="listQuery.page"
      :limit.sync="listQuery.limit"
      @pagination="getList"
    />
    <!-- 编辑与添加面板 -->
    <el-dialog
      width="1000px"
      :title="textMap[dialogStatus]"
      :visible.sync="dialogFormVisible"
    >
      <el-form
        ref="dataForm"
        :rules="rules"
        :model="temp"
        label-position="left"
        label-width="70px"
        style="width: 400px; margin-left: 50px"
      >
        <el-form-item label="标题" prop="title">
          <el-input v-model="temp.title" />
        </el-form-item>
        <el-form-item label="封面">
          <div class="editor-container">
            <dropzone
              v-model="temp.cover"
              id="myVueDropzone"
              url="https://httpbin.org/post"
              @dropzone-removedFile="dropzoneR"
              @dropzone-success="dropzoneS"
            />
          </div>
        </el-form-item>
        <el-form-item label="试看内容" prop="trySee">
          <tinymce v-model="temp.try" :width="600" :height="300" />
        </el-form-item>
        <el-form-item label="课程内容">
          <tinymce v-model="temp.content" :width="600" :height="300" />
        </el-form-item>
        <el-form-item label="课程价格">
          <el-input-number v-model="temp.price"></el-input-number>
        </el-form-item>
        <el-form-item label="Status">
           <el-radio-group v-model="temp.status">
            <el-radio :label="1">上架</el-radio>
            <el-radio :label="0">下架</el-radio>
        </el-radio-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false"> 取消 </el-button>
        <el-button
          type="primary"
          @click="dialogStatus==='create'?createData():updateData()"
        >
          提交
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import Pagination from "@/components/Pagination";
import Dropzone from "@/components/Dropzone";
import Tinymce from "@/components/Tinymce";
import { fetchList,createAudio,updateAudio,deleteAudio} from "@/api/audio";
export default {
  name: "audio",
  components: { Pagination, Dropzone, Tinymce },
  data() {
    return {
      options: [
        {
          value: "选项1",
          label: "已上架",
        },
        {
          value: "选项2",
          label: "已下架",
        },
      ],
      value: "",
      list: null, //表格数据
      total: 0,
      listQuery: {
        page: 1,
        limit: 20,
      },
      statusOptions: [0, 1],
      temp: {
        id: '',
        title: "",
        cover: "",
        try:'',
        content:'',
        price:'0',
        sub_count:'',
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
    sortChange(data) {
      const { prop, order } = data;
      if (prop === "id") {
        this.sortByID(order);
      }
    },
    sortByID(order) {
      if (order === "ascending") {
        this.listQuery.sort = "+id";
      } else {
        this.listQuery.sort = "-id";
      }
      this.handleFilter();
    },
    //获取表格内容
    getList() {
      fetchList(this.listQuery).then((response) => {
        // console.log(response);
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
      deleteMedia(row.id).then((response) => {
        console.log(response);
        if (response.code === 20000) {
          this.$notify({
            title: "Success",
            message: "删除成功",
            type: "success",
            duration: 2000,
          });
          this.list.splice(index, 1);
        }
      });
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
    createData(){
        createMedia(this.listQuery).then((response) => {
            // console.log(response);
            if (response.code === 20000) {
                this.dialogFormVisible = false
                this.$notify({
                title: 'Success',
                message: '新增成功',
                type: 'success',
                duration: 2000
                })
            }
      });
    },
    //编辑
    updateData(){
        updateMedia(this.listQuery).then((response) => {
        console.log(response);
        if (response.code === 20000) {
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: '添加成功',
              type: 'success',
              duration: 2000
            })
        }
      });
    },
    handleUpdate(row){
      this.temp = Object.assign({}, row)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    }
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