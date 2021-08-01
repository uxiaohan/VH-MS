<template>
  <div class="container">
    <el-card class="page-tools">
      <el-row type="flex" justify="space-between" align="middle">
        <el-col>
          <div class="before">
            <!-- 定义前面得插槽 -->
            <el-form label-width="80px">
              <el-form-item label="学科名称">
                <el-input
                  v-model="input"
                  style="width: 200px; heigth: 20px"
                  id="inputId"
                  ref="inputValue"
                  clearable
                ></el-input>
                <el-button style="margin-left: 10px" @click="clear"
                  >清除</el-button
                >
                <el-button type="primary" @click="search">搜索</el-button>
              </el-form-item>
            </el-form>
            <slot name="before" />
          </div>
        </el-col>
        <el-col>
          <el-row type="flex" justify="end" class="after">
            <!-- 定义后面的插槽 -->
            <el-button
              type="success"
              icon="el-icon-edit"
              @click="addSubjects"
              label="新增"
              >新增学科</el-button
            >
            <slot name="after" />
          </el-row>
        </el-col>
      </el-row>
      <el-alert
        :title="'数据一共' + page.counts + '条'"
        type="info"
        show-icon
        :closable="false"
      >
      </el-alert>

      <!-- 放置表格与分页组件 -->
      <el-table border style="margin-top: 20px" :data="list">
        <el-table-column
          label="序号"
          sortable=""
          type="index"
          :index="getIndex"
        />
        <el-table-column label="学科名称·" sortable="" prop="subjectName" />
        <el-table-column label="创建者" sortable="" prop="username" />
        <el-table-column label="创建日期" sortable="" prop="addDate">
          <template slot-scope="obj">
            {{ obj.row.addDate | parseTimeByString }}
          </template>
        </el-table-column>

        <el-table-column
          label="前台是否显示"
          sortable=""
          prop="isFrontDisplay"
          :formatter="formatSubjects"
        />

        <el-table-column
          label="二级目录"
          sortable=""
          prop="twoLevelDirectory"
        />
        <el-table-column label="标签" sortable="" prop="tags" />
        <el-table-column label="题目数量" sortable="" prop="totals" />
        <el-table-column label="操作" sortable="" fixed="right" width="280">
          <template slot-scope="{ row }">
            <el-button
              type="text"
              size="small"
              @click="
                $router.push(
                  `/subjects/directorys?id=${row.id}&name=${row.subjectName}`
                )
              "
              >学科分类</el-button
            >
            <el-button
              type="text"
              size="small"
              @click="
                $router.push(
                  `/subjects/tags?id=${row.id}&name=${row.subjectName}`
                )
              "
              >学科标签</el-button
            >
            <el-button type="text" size="small" @click="editSubject(row)"
              >修改</el-button
            >
            <el-button type="text" size="small" @click="delSubject(row)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页组件 -->
      <el-row type="flex" justify="center" align="middle" style="height: 60px">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :page-sizes="[10, 20]"
          :page-size="10"
          layout="total, prev, pager, next,sizes, jumper"
          :total="page.counts"
        >
        </el-pagination>
      </el-row>
    </el-card>
    <!-- 新增学科弹层 -->
    <AddSubject
      v-if="showDialog"
      :show-dialog.sync="showDialog"
      :row="row"
    ></AddSubject>
  </div>
</template>

<script>
import { list, remove } from "@/api/hmmm/subjects";
import { hireType } from "@/api/hmmm/constants";
import AddSubject from "../components/subjects-add.vue";
export default {
  data() {
    return {
      input: "",
      list: [],
      page: {
        page: 1,
        pagesize: 10,
        counts: 0,
      },
      showDialog: false, //默认关闭弹层,
      row: {},
    };
  },
  components: {
    AddSubject,
  },
  created() {
    this.getSubjectList();
  },
  methods: {
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    // newPage最新页
    async handleCurrentChange(newPage) {
      this.page.page = newPage;
      const {
        data: { items },
      } = await list({ page: this.page.page });
      console.log(items);
      this.list = items;
    },
    // 获取学科列表
    async getSubjectList() {
      const { data } = await list();
      this.list = data.items;
      this.page.counts = data.counts;
      console.log(data);
    },

    // 是否前台显示
    formatSubjects(row, column, cellValue, index) {
      // 要去找所对应的值
      const obj = hireType.find((item) => item.id === cellValue);
      return obj ? obj.value : "未知";
    },
    // 删除学科
    async delSubject(row) {
      this.$confirm("您确定删除该学科吗", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          await remove(row);
          this.getSubjectList();
          this.$message({
            type: "success",
            message: "删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    // 修改学科
    async editSubject(row) {
      this.row = row;
      this.showDialog = true;
    },
    // 新增学科
    addSubjects() {
      this.showDialog = true;
      this.row = {};
    },
    //  清空按纽
    clear() {
      this.input = "";
      document.getElementById("inputId").focus();
    },
    //搜索列表
    async search() {
      const {
        data: { items },
      } = await list({ subjectName: this.input });
      console.log(items);
      this.list = items;
    },
  },
  computed: {
    // 序号排序
    getIndex() {
      return (this.page.page - 1) * this.page.pagesize + 1;
    },
  },
};
</script>

<style scoped lang='scss'>
.page-tools {
  margin: 10px 0;
  padding: none;
}
.before {
  line-height: 34px;
}
.after {
  margin-top: -20px;
  color: white;
}
</style>
