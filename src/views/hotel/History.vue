<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>酒店管理</el-breadcrumb-item>
      <el-breadcrumb-item>酒店历史列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getHistoryList">
            <el-button slot="append" icon="el-icon-search" @click="getHistoryList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加历史</el-button>
        </el-col>
      </el-row>

      <!-- 酒店历史列表数据 -->
      <el-table :data="historylist" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="historyIntro" label="酒店历史简介"></el-table-column>
        <el-table-column prop="historyPicture" label="酒店历史图片">
          <template slot-scope="scope">
            <img :src="'http://101.132.135.179:8080'+scope.row.historyPicture" width="150px" height="100px"/>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showHistoryDialog(scope.row.id)"
            >编辑</el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeHistoryById(scope.row.id)"
            >删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.page"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.limit"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>

      <el-row :gutter="20">
        <el-col :span="4">
          <el-button type="danger" style="margin:10px 0 0 0">批量删除</el-button>
        </el-col>
      </el-row>
    </el-card>

    <!-- 添加酒店历史的对话框 -->
    <el-dialog
      title="添加酒店历史信息"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="历史简介" prop="historyIntro">
          <el-input v-model="addForm.historyIntro"></el-input>
        </el-form-item>
        <el-form-item label="酒店历史图片" prop="file">
          <el-upload
            ref="upload"
            action="http://101.132.135.179:8088/hotel/history/insertHistory"
            :limit="limitNum"
            :auto-upload="false"
            accept=".jpg"
            :before-upload="beforeUploadFile"
            :on-change="fileChange"
            :on-exceed="exceedFile"
            :file-list="fileList"
          >
            <el-button size="small" plain>选择文件</el-button>
            <div slot="tip" class="el-upload__tip" style="color:grey;">只能上传jpg文件，且不超过10M</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addHistory">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑酒店历史的对话框 -->
    <el-dialog
      title="修改酒店历史信息"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="历史简介" prop="historyIntro">
          <el-input v-model="editForm.historyIntro"></el-input>
        </el-form-item>
        <el-form-item label="酒店历史图片" prop="file">
          <el-upload
            ref="upload"
            action="http://101.132.135.179:8088/hotel/history/updateHistory"
            :limit="limitNum"
            :auto-upload="false"
            accept=".jpg"
            :before-upload="beforeUploadFile"
            :on-change="fileChange"
            :on-exceed="exceedFile"
            :file-list="fileList"
          >
            <el-button size="small" plain>选择文件</el-button>
            <div slot="tip" class="el-upload__tip" style="color:grey;">只能上传jpg文件，且不超过10M</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editHistoryInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        // query: "",
        // 当前的页数
        page: 1,
        // 当前每页显示多少条数据
        limit: 5
      },
      total: 0,
      historylist: [],
      addDialogVisible: false,
      // 图片上传
      limitNum: 1,
      // 添加酒店历史的表单数据
      addForm: {
        file: "",
        historyIntro: ""
      },
      fileList: [],
      // 添加表单的验证规则对象
      addFormRules: {
        historyIntro: [
          { required: true, message: "请输入酒店历史简介", trigger: "blur" }
        ]
      },
      // 控制修改酒店历史信息对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的酒店历史订息对象
      editForm: {},
      // 修改酒店历史表单的验证规则对象
      editFormRules: {
        historyIntro: [
          { required: true, message: "请输入酒店历史简介", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getHistoryList();
  },
  methods: {
    async getHistoryList() {
      const { data: res } = await this.$http.get(
        "http://101.132.135.179:8088/hotel/select/getHistoryList",
        {
          params: this.queryInfo
        }
      );

      if (res.code !== 0) {
        return this.$message.error("获取酒店历史列表失败！");
      }
      this.$message.success("获取酒店历史列表成功！");

      console.log(res);
      this.total = res.count;
      this.historylist = res.data;
    },
    handleSizeChange(newSize) {
      this.queryInfo.page = newSize;
      this.getHistoryList();
    },
    handleCurrentChange(newPage) {
      this.queryInfo.limit = newPage;
      this.getHistoryList();
    },
    // 监听添加酒店历史对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
      this.$refs.upload.clearFiles();
    },
    // 添加酒店历史的对话框
    addHistory() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return;
        let formData = new FormData();
        formData.append("file", this.addForm.file);
        // 可以发起添加酒店历史的网络请求
        const {
          data: res
        } = await this.$http.post(
          "http://101.132.135.179:8088/hotel/history/insertHistory?" +
            "historyIntro=" +
            this.addForm.historyIntro ,
          formData,
          { "Content-Type": "multipart/form-data" }
        );

        if (res.code == 0) {
          this.$message.success("添加酒店历史信息成功！");
        } else if (res.code == 1000) {
          this.$message.warning("对不起，你无权访问，请升级权限");
        } else {
          this.$message.error("添加酒店历史信息失败！");
        }

        // 隐藏添加酒店历史的对话框
        this.addDialogVisible = false;
        // 重新获取酒店历史列表数据
        this.getHistoryList();
      });
    },
    // 展示编辑酒店历史信息的对话框
    async showHistoryDialog(id) {
      console.log(id);
      const { data: res } = await this.axios.post(
        "http://101.132.135.179:8088/hotel/select/getHistoryById?id=" + id
      );

      console.log("根据id查询数据", res);
      if (res.code != 0) {
        return this.$message.success("查询酒店历史信息失败！");
      }
      this.$message.success("查询酒店历史信息成功！");
      this.editForm = res.data;
      this.editForm.password = "";
      console.log("酒店历史数据", res.data);
      this.editDialogVisible = true;
    },
    // 监听修改酒店历史对话框的关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
      this.$refs.upload.clearFiles();
    },
    // 修改酒店历史信息并提交
    editHistoryInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return;
        // 发起修改酒店历史信息的数据请求
        let formData1 = new FormData();
        formData1.append("file", this.editForm.file);
        const { data: res } = this.$http
          .post(
            "http://101.132.135.179:8088/hotel/history/updateHistory?id=" +
              this.editForm.id +
              "&historyIntro=" +
              this.editForm.historyIntro,
            formData1,
            { "Content-Type": "multipart/form-data" }
          )
          .then(res => {
            if (res.data.code !== 0) {
              return this.$message.error("更新酒店历史信息失败！");
            }
            // 提示修改成功
            this.$message.success("更新酒店历史信息成功！");
            // 关闭对话框
            this.editDialogVisible = false;
            // 刷新数据列表
            this.getHistoryList();
          })
          .catch(err => {});
      });
    },
    // 根据Id删除对应酒店历史信息
    async removeHistoryById(id) {
      console.log(id);
      // 弹框询问酒店历史信息是否删除数据
      const confirmResult = await this.$confirm(
        "此操作将永久删除该酒店历史数据, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);

      // 如果酒店历史信息确认删除，则返回值为字符串 confirm
      // 如果酒店历史信息取消了删除，则返回值为字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== "confirm") {
        return this.$message.info("已取消删除");
      }

      const { data: res } = await this.axios
        .post(
          "http://101.132.135.179:8088/hotel/history/recycleHistory?id=" + id
        )
        .then(res => {
          console.log("res", res);
          if (res.code != 0) {
            return this.$message.success("删除酒店历史成功！");
          }
          this.$message.error("删除酒店历史失败！");
        });
      this.getHistoryList();
    },
    // 文件超出个数限制时的钩子
    exceedFile(files, fileList) {
      this.$notify.warning({
        title: "警告",
        message: `只能选择 ${
          this.limitNum
        } 个文件，当前共选择了 ${files.length + fileList.length} 个`
      });
    },
    // 文件状态改变时的钩子
    fileChange(file, fileList) {
      console.log("change");
      console.log(file);
      this.addForm.file = file.raw;
      console.log(this.addForm.file);
      console.log(fileList);
    },
    // 上传文件之前的钩子, 参数为上传的文件,若返回 false 或者返回 Promise 且被 reject，则停止上传
    beforeUploadFile(file) {
      console.log("before upload");
      console.log(file);
      let extension = file.name.substring(file.name.lastIndexOf(".") + 1);
      let size = file.size / 1024 / 1024;
      if (extension !== "jpg") {
        this.$notify.warning({
          title: "警告",
          message: `只能上传jpg（即后缀是.jpg）的文件`
        });
      }
      if (size > 10) {
        this.$notify.warning({
          title: "警告",
          message: `文件大小不得超过10M`
        });
      }
    }
    // // 文件上传成功时的钩子
    // handleSuccess(res, file, fileList) {
    //   this.$notify.success({
    //     title: "成功",
    //     message: `文件上传成功`
    //   });
    // },
    // // 文件上传失败时的钩子
    // handleError(err, file, fileList) {
    //   this.$notify.error({
    //     title: "错误",
    //     message: `文件上传失败`
    //   });
    // }
  }
};
</script>

<style lang="stylus">
.el-cascader {
  width: 100%;
}
</style>
