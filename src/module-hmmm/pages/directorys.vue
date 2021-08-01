<template>
  <div class="container">
    <el-card class="box-card">
      <div class="head_ser">
        <section class="head_ipt">
          <el-form
            :inline="true"
            :rules="fmListRul"
            :model="fmList"
            ref="fmList"
          >
            <!-- 目录名称数组列表 -->
            <el-form-item label="目录名称" prop="directoryName">
              <el-input v-model="fmList.directoryName"></el-input>
            </el-form-item>
            <el-form-item label="状态" prop="state">
              <el-select v-model="fmList.state" placeholder="请选择">
                <el-option label="已启用" :value="1"></el-option>
                <el-option label="已禁用" :value="0"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button @click="clearFm">清除</el-button>
              <el-button type="primary" @click="SearchFm">搜索</el-button>
            </el-form-item>
          </el-form>
        </section>
        <section class="head_btn">
          <el-button
            icon="el-icon-back"
            @click="$router.push('/subjects/list')"
            type="text"
            >返回学科</el-button
          >
          <el-button @click="disAbl(true)" type="success" icon="el-icon-edit"
            >新增目录</el-button
          >
        </section>
      </div>
      <!-- 数据显示标签 -->
      <section class="list_con">
        <div class="tips">
          <el-alert :title="`数据一共${thepage.total}条`" type="info" show-icon>
          </el-alert>
        </div>
        <!-- Table数据显示 -->
        <el-table :data="listTableData" style="width: 100%">
          <el-table-column type="index" prop="address" label="序号">
          </el-table-column>
          <el-table-column prop="subjectName" label="所属学科">
          </el-table-column>
          <el-table-column prop="directoryName" label="目录名称">
          </el-table-column>
          <el-table-column prop="username" label="创建者"> </el-table-column>
          <el-table-column prop="addDate" label="创建日期"> </el-table-column>
          <el-table-column prop="totals" label="面试题数量"> </el-table-column>
          <el-table-column prop="state" label="状态"> </el-table-column>
          <el-table-column align="right">
            <template slot="header">
              <span>操作</span>
            </template>
            <template slot-scope="scope">
              <el-button type="text" @click="tableDis(scope.row)">{{
                scope.row.state ? "禁用" : "启用"
              }}</el-button>
              <el-button
                type="text"
                @click="tableEditFn(scope.row)"
                :disabled="Boolean(scope.row.state)"
                >修改</el-button
              >
              <el-button
                type="text"
                @click="tableDel(scope.row)"
                :disabled="Boolean(scope.row.state)"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <!-- 底部分页功能 -->
        <div class="pages">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="thepage.page"
            :page-sizes="[5, 10, 20, 50]"
            :page-size="thepage.pagesize"
            layout=" prev, pager, next,sizes, jumper"
            :total="thepage.total"
          >
          </el-pagination>
        </div>
      </section>
    </el-card>
    <!-- 弹出框功能 -->
    <DirectAdd
      ref="diradd"
      :orDiaShow="orDiaShow"
      :oldData="oldData"
      :orUpdate.sync="orUpdate"
      @disAbl="disAbl"
      @addData="addTableData"
      @updateData="tableEdit"
      :routeId="$route.query.id || null"
    ></DirectAdd>
  </div>
</template>

<script>
import { list, add, remove, changeState, update } from "@/api/hmmm/directorys";
export default {
  components: {
    DirectAdd: () => import("../components/directorys-add"),
  },
  props: {
    // 获取传来的关键词
    keyword: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      // 新增修改弹出层显示或者隐藏参数
      orDiaShow: false,
      // 判断是否是更新数据
      orUpdate: false,
      // 判断是否有原始数据
      oldData: null,
      // 待提交的Form表单数据
      fmList: {
        directoryName: "",
        state: "",
      },
      // 表单校验规则
      fmListRul: {
        directoryName: [
          {
            required: true,
            trigger: "blur",
            message: "请输入目录名称",
          },
        ],
        state: [
          {
            required: true,
            trigger: "blur",
            message: "请选择状态",
          },
        ],
      },
      // Table表单数据
      listTableData: [],
      // 分页功能数据
      thepage: {
        page: 1,
        pagesize: 5,
        total: 0,
        directoryName: "",
        state: null,
        subjectID: this.$route.query.id || null,
      },
    };
  },
  created() {
    // 初始化table数据
    this.getTheListData();
  },
  watch: {
    // 监控Form表单数据变化
    fmList: {
      handler(val) {
        this.thepage.directoryName = val.directoryName || "";
        this.thepage.state = val.state || null;
      },
      deep: true,
      immediate: true,
    },
  },
  methods: {
    // 显示新增弹框
    disAbl(or) {
      this.orDiaShow = or;
    },
    // 请求获取数据
    async getTheListData() {
      const { data: res } = await list(this.thepage);
      this.thepage.total = res.counts;
      this.listTableData = res.items;
    },
    // 清空输入框
    clearFm() {
      if (this.fmList.directoryName !== "" && this.fmList.state != "") {
        this.fmList = {};
        this.getTheListData();
      }
    },
    // 提交输入框搜索
    SearchFm() {
      this.$refs.fmList
        .validate()
        .then(() => {
          this.getTheListData();
        })
        .catch((err) => {
          console.log(err, "错误");
        });
    },
    // 新增数据条目
    async addTableData({ name, subid }) {
      const { data: res } = await add({
        subjectID: subid,
        directoryName: name,
      });
      if (res.id) {
        this.$message.success("添加成功");
        this.getTheListData();
      } else {
        this.$message.fail("添加失败");
      }
      this.$refs.diradd.diaClose();
      // this.disAbl(false);
    },
    // 禁用当前数据状态
    async tableDis(row) {
      const { data: res } = await changeState({
        id: row.id,
        state: !row.state ? 1 : 0,
      });
      if (res.success === true) {
        this.$message.success("修改成功");
        this.listTableData[this.listTableData.indexOf(row)].state = !row.state
          ? 1
          : 0;
      } else {
        this.$message.fail("修改失败");
      }
    },
    // 删除当前数据条目
    tableDel(row) {
      this.$confirm("此操作将永久删除该目录, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          const { data: res } = await remove({ id: row.id });
          if (res.success) {
            // this.listTableData.splice(this.listTableData.indexOf(row), 1);
            this.$message({
              type: "success",
              message: "删除成功!",
            });
            this.getTheListData();
          } else {
            this.$message({
              type: "danger",
              message: "删除失败!",
            });
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    // 修改当前数据条目
    tableEditFn({ directoryName, subjectName, id }) {
      this.orUpdate = true;
      console.log({ directoryName, subjectName, id });
      this.oldData = {
        name: directoryName,
        subid: subjectName,
        id: id,
      };
      this.orDiaShow = true;
    },
    // 数据修改事件
    async tableEdit({ subid, name, id }) {
      const { data: res } = await update({
        subjectID: subid,
        directoryName: name,
        id: id,
      });
      if (res.success) {
        this.$message({
          type: "success",
          message: "修改成功!",
        });
        this.getTheListData();
      } else {
        this.$message({
          type: "danger",
          message: "修改失败!",
        });
      }
      this.orUpdate = false;
      this.$refs.diradd.diaClose();
    },
    // 一页最多数据变化事件
    handleSizeChange(val) {
      this.thepage.pagesize = val;
      this.getTheListData();
    },
    // 分页或者跳转数值变化时事件
    handleCurrentChange(val) {
      this.thepage.page = val;
      this.getTheListData();
    },
  },
};
</script>

<style scoped>
.container {
  margin-bottom: 36px;
}
.head_ser {
  display: flex;
  justify-content: space-between;
}
.head_ser .head_ipt {
  flex: 1;
}
.head_ser .head_btn button {
  margin: 0 0 0 30px;
}
.list_con .tips {
  margin: 0px 0 16px;
}
.list_con .pages {
  margin: 26px 0;
  float: right;
}
</style>
