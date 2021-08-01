<template>
  <div class="randoms">
    <el-card class="box-card">
      <!-- 表单 -->
      <el-row>
        <el-form ref="form" inline :model="page">
          <el-row
            :gutter="20"
            type="flex"
            align="center"
            justify="space-between"
            style="margin-left: 20px; width: 100%"
          >
            <el-col>
              <el-form-item label="关键字">
                <el-input
                  placeholder="根据编号搜索"
                  v-model="page.keyword"
                ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-row type="flex" justify="end">
                <el-form-item>
                  <el-button size="small" @click="page.keyword = ''"
                    >清楚</el-button
                  >
                  <el-button type="primary" size="small" @click="search"
                    >搜索</el-button
                  >
                </el-form-item>
              </el-row>
            </el-col>
          </el-row>
        </el-form>
      </el-row>
      <!-- /表单 -->
      <!-- 显示数据总条数 -->
      <el-alert
        :title="`数据一共 ${page.counts} 条`"
        type="info"
        show-icon
        :closable="false"
      />
      <!-- /显示数据总条数 -->

      <!-- 表格 -->
      <el-table :data="list" style="margin-top: 20px">
        <el-table-column label="编号" prop="id" width="220" />
        <el-table-column
          width="80"
          :formatter="questionTypeFormatter"
          label="题型"
          prop="questionType"
        >
          <!-- <template slot-scope="scope">
            <span>{{ scope.row.questionType == 3 ? `多选` : `简答` }}</span>
          </template> -->
        </el-table-column>
        <el-table-column width="220" label="题目编号">
          <template slot-scope="scope">
            <div v-for="(item, ind) in scope.row.questionIDs" :key="ind">
              <a
                href="#"
                style="color: rgb(0, 153, 255)"
                @click="setRecords(item.id)"
                >{{ item.number }}</a
              >
            </div>
          </template>
        </el-table-column>
        <el-table-column width="180" label="录入时间">
          <template slot-scope="scope">
            <span>
              {{
                scope.row.addTime | parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")
              }}
            </span>
          </template>
        </el-table-column>
        <el-table-column
          style="min-width: 80px"
          label="答题时间(s)"
          prop="totalSeconds"
        >
        </el-table-column>
        <el-table-column
          style="min-width: 80px"
          prop="accuracyRate"
          label="正确率(%)"
        >
        </el-table-column>
        <el-table-column style="min-width: 80px" prop="userName" label="录入人">
        </el-table-column>
        <el-table-column style="min-width: 80px" label="操作">
          <template slot-scope="scope">
            <el-button
              class="iconbut"
              size="small "
              icon="el-icon-delete"
              circle
              @click="removeRandoms(scope.row.id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- /表格 -->
      <!-- 分页 -->
      <el-row type="flex" justify="end" align="middle" style="height: 60px">
        <el-pagination
          background
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="page.page"
          :page-sizes="[5, 10, 20, 50]"
          :page-size="20"
          layout="  prev, pager, next,sizes,jumper"
          :total="page.counts"
        >
        </el-pagination>
      </el-row>
      <!-- /分页 -->
    </el-card>
    <QuestionsPreview
      :showPrevDialog.sync="showPrevDialog"
      :currentData="currentData"
    />
  </div>
</template>
<script>
import { randoms, removeRandoms, detail } from "@/api/hmmm/questions";
import QuestionsPreview from "../components/questions-preview.vue";
import { questionType, difficulty, direction } from "@/api/hmmm/constants.js";
export default {
  components: { QuestionsPreview },
  props: {},
  data() {
    return {
      list: [{ id: 123 }],
      page: {
        page: 1, //当前页数
        pagesize: 20, //页尺寸
        counts: 0,
        keyword: "", //关键字
      },
      questionTypeList: questionType, // 试题类型列表
      difficultyList: difficulty, // 难度列表
      showPrevDialog: false, //关闭弹层
      currentData: {},
    };
  },
  created() {
    this.randoms();
  },

  methods: {
    // 获取组题列表
    async randoms() {
      const { data } = await randoms(this.page);

      this.list = data.items;
      this.page.counts = data.counts;
      console.log(this.list);
    },
    // 组题列表删除
    async removeRandoms(id) {
      console.log(id);
      this.$confirm("此操作将永久删除该题组, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          console.log(id);
          try {
            await removeRandoms({
              id,
            });
            if (this.list.length === 1 && this.page.page !== 1) {
              this.page.page--;
            }
            this.$message({
              showClose: true,
              duration: 1500,
              type: "success",
              message: "删除成功!",
            });
            this.randoms();
          } catch (err) {}
        })
        .catch((err) => {});
    },
    search() {
      this.randoms();
    },

    // 分页功能
    handleSizeChange(val) {
      this.page.pagesize = val;
      this.randoms();
    },
    handleCurrentChange(val) {
      this.page.page = val;
      this.randoms();
    },
    // 题目编辑浏览
    async setRecords(id) {
      const data = {
        id: id,
      };
      this.showPrevDialog = true;
      const date = await detail(data);

      this.currentData = date.data;
      console.log(this.currentData);
    },
    questionTypeFormatter(row) {
      switch (row.questionType) {
        case "1":
          row.questionType = "单选";
          break;
        case "2":
          row.questionType = "多选";
          break;
        case "3":
          row.questionType = "简答";
          break;
      }
      return row.questionType;
    },
    difficultyFormatter(row) {
      switch (row.difficulty) {
        case "1":
          row.difficulty = "简单";
          break;
        case "2":
          row.difficulty = "一般";
          break;
        case "3":
          row.difficulty = "困难";
          break;
      }
      return row.difficulty;
    },
  },

  computed: {},

  watch: {},
};
</script>
<style lang="scss" scoped>
.randoms {
  padding: 20px;
  .iconbut {
    color: #f56c6c;
    background: #fef0f0;
    border-color: #fbc4c4;
  }
}
</style>
