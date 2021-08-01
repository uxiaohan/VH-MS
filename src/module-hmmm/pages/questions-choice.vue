<template>
  <div class="container">
    <el-card shadow="never">
      <el-form ref="form" :model="form" label-width="80px">
        <div class="filter-container">
          <el-row :gutter="20">
            <el-col :span="6" :offset="18">
              <el-button
                @click="addQBtn"
                class="fr filter-item"
                icon="el-icon-edit"
                type="success"
                >新增试题</el-button
              >
            </el-col>
            <el-col :span="6">
              <el-form-item label="学科" prop="subjectID">
                <el-select
                  @focus="getSubjectSimpleList"
                  v-model="form.subjectID"
                  placeholder="请选择"
                >
                  <el-option
                    v-for="item in subjectSimpleList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="二级目录" prop="catalogID">
                <el-select
                  @focus="getDirectorysSimpleList"
                  v-model="form.catalogID"
                  placeholder="请选择"
                >
                  <el-option
                    v-for="item in directorysSimpleList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="标签" prop="tags">
                <el-select
                  @focus="getTagsSimpleList"
                  v-model="form.tags"
                  placeholder="请选择"
                >
                  <el-option
                    v-for="item in tagsSimpleList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="关键字" prop="keyword">
                <el-input v-model="form.keyword"></el-input> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="试题类型" prop="questionType">
                <el-select v-model="form.questionType" placeholder="请选择">
                  <el-option
                    v-for="item in questionTypeList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="难度" prop="difficulty">
                <el-select v-model="form.difficulty" placeholder="请选择">
                  <el-option
                    v-for="item in difficultyList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value"
                  ></el-option>
                </el-select> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="方向" prop="direction">
                <el-select v-model="form.direction" placeholder="请选择">
                  <el-option
                    v-for="(item, index) in directionList"
                    :key="index"
                    :label="item"
                    :value="index"
                  ></el-option>
                </el-select> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="录入人" prop="creatorID">
                <el-select
                  @focus="getUsersSimple"
                  v-model="form.creatorID"
                  placeholder="请选择"
                >
                  <el-option
                    v-for="item in usersSimpleList"
                    :key="item.value"
                    :label="item.username"
                    :value="item.id"
                  ></el-option>
                </el-select> </el-form-item
            ></el-col>
            <el-col :span="6">
              <el-form-item label="题目备注" prop="remarks">
                <el-input v-model="form.remarks"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="6">
              <el-form-item label="企业简称" prop="shortName">
                <el-input v-model="form.shortName"></el-input> </el-form-item
            ></el-col>
            <el-col :span="6"
              ><div class="grid-content bg-purple provCity">
                <el-form-item label="城市"></el-form-item>
                <el-form-item
                  label-width="0"
                  prop="province"
                  class="provinceSelect"
                >
                  <el-select
                    @focus="getProvincesList"
                    v-model="form.province"
                    placeholder="请选择"
                  >
                    <el-option
                      v-for="(item, index) in provincesList"
                      :key="index"
                      :label="item"
                      :value="item"
                    ></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label-width="0" prop="city" class="citySelect">
                  <el-select
                    @focus="getCitysList"
                    v-model="form.city"
                    placeholder="请选择"
                  >
                    <el-option
                      v-for="(item, index) in citysList"
                      :key="index"
                      :label="item"
                      :value="item"
                    ></el-option>
                  </el-select>
                </el-form-item></div
            ></el-col>
            <el-col :span="6">
              <el-button
                @click="searchBtn"
                type="primary"
                class="fr filter-item"
                >搜索</el-button
              >
              <el-button @click="clearBtn" class="fr filter-item"
                >清除</el-button
              >
            </el-col>
          </el-row>
        </div>
      </el-form>
      <el-tabs type="card" v-model="form.chkState" @tab-click="handleClick">
        <el-alert
          v-if="alertText !== ''"
          :title="alertText"
          type="info"
          :closable="false"
          show-icon
        ></el-alert>
        <!-- name="all2134234564234" 防止重名 -->
        <el-tab-pane label="全部" name="all2134234564234"></el-tab-pane>
        <el-tab-pane label="待审核" name="0"></el-tab-pane>
        <el-tab-pane label="已审核" name="1"></el-tab-pane>
        <el-tab-pane label="已拒绝" name="2"></el-tab-pane>
      </el-tabs>
      <el-table :data="tableData" style="width: 100%" border>
        <el-table-column prop="number" label="试题编号" width="150">
        </el-table-column>
        <el-table-column prop="subject" label="学科" width="150">
        </el-table-column>
        <el-table-column prop="catalog" label="目录" width="150">
        </el-table-column>
        <el-table-column
          :formatter="questionTypeFormatter"
          prop="questionType"
          label="题型"
          width="150"
        >
        </el-table-column>
        <el-table-column label="题干" width="350">
          <template slot-scope="scope">
                        <span v-html="scope.row.question"></span>
                      </template
          >
        </el-table-column>
        <el-table-column label="录入时间" width="250">
            <template slot-scope="scope">
                        <span>{{
              scope.row.addDate | parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")
            }}</span>
                      </template
          >
        </el-table-column>
        <el-table-column
          :formatter="difficultyFormatter"
          prop="difficulty"
          label="难度"
          width="150"
        >
        </el-table-column>
        <el-table-column prop="creator" label="录入人" width="150">
        </el-table-column>
        <el-table-column
          :formatter="chkStateFormatter"
          prop="chkState"
          label="审核状态"
          width="150"
        >
        </el-table-column>
        <el-table-column prop="chkRemarks" label="审核意见" width="150">
        </el-table-column>
        <el-table-column prop="chkUser" label="审核人" width="150">
        </el-table-column>
        <el-table-column
          :formatter="publishStateFormatter"
          prop="publishState"
          label="发布状态"
          width="150"
        >
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="200" type="index">
          <template slot-scope="scope">
            <el-button @click="previewBtn(scope.row)" type="text" size="small"
              >预览</el-button
            >
            <el-button
              :disabled="scope.row.chkState === 1"
              @click="checkBtn(scope.row)"
              type="text"
              size="small"
              >审核</el-button
            >
            <el-button
              :disabled="scope.row.publishState === 1"
              @click="editBtn(scope.row)"
              type="text"
              size="small"
              >修改</el-button
            >
            <el-button @click="downBtn(scope.row)" type="text" size="small">{{
              !scope.row.publishState ? "上架" : "下架"
            }}</el-button>
            <el-button
              :disabled="scope.row.publishState === 1"
              @click="deleteBtn(scope.row)"
              type="text"
              size="small"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="form.page"
        :page-sizes="[1, 2, 3, 4, 5]"
        :page-size="form.pagesize"
        layout="prev, pager, next, sizes, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 题目预览弹出层 -->
    <QuestionPreview
      v-if="showPrevDialog"
      :showPrevDialog.sync="showPrevDialog"
      :currentData="currentData"
    ></QuestionPreview>
    <!-- 试题审核弹出层 -->
    <QuestionCheck
      v-if="showCheckDialog"
      :showCheckDialog.sync="showCheckDialog"
      :currentData="currentData"
      @renderTableData="loadQuestions"
    ></QuestionCheck>
  </div>
</template>

<script>
import { choice, choicePublish, remove, detail } from "@/api/hmmm/questions.js";
import { simple as subjectsSimple } from "@/api/hmmm/subjects.js";
import { simple as directorysSimple } from "@/api/hmmm/directorys.js";
import { simple as tagsSimple } from "@/api/hmmm/tags.js";
import { simple as usersSimple } from "@/api/base/users.js";
import { questionType, difficulty, direction } from "@/api/hmmm/constants.js";
import { provinces, citys } from "@/api/hmmm/citys.js";
import QuestionPreview from "../components/questions-preview.vue";
import QuestionCheck from "../components/questions-check.vue";

export default {
  data() {
    return {
      form: {
        subjectID: "", //学科
        catalogID: "", //二级目录
        tags: "", // 标签
        keyword: "", // 关键字
        questionType: "", // 试题类型
        difficulty: "", // 难度
        direction: "", // 方向
        creatorID: "", // 录入人
        remarks: "", // 题目备注
        shortName: "", // 企业简称
        province: "", // 企业所在地省份
        city: "", // 企业所在城市
        // ----------------------------------------------------
        chkState: "all2134234564234", // 审核状态
        // ----------------------------------------------------
        page: 1, // 当前页数
        pagesize: 5, // 每页条目数
      },
      total: 0, // 总条目数
      alertText: "", //数据总条数
      tableData: [], // 试题数据列表
      subjectSimpleList: [], // 学科简单列表
      directorysSimpleList: [], // 目录简单列表
      tagsSimpleList: [], // 标签简单列表
      questionTypeList: questionType, // 试题类型列表
      difficultyList: difficulty, // 难度列表
      directionList: direction, // 方向列表
      usersSimpleList: [], // 录入人的简单列表
      provincesList: [], // 城市列表
      citysList: [], // 区域列表
      // ----------------------------------
      showPrevDialog: false, // 显示预览弹出层
      currentData: {}, // 当前试题数据
      showCheckDialog: false, // 显示尸体审核弹出层
      currentId: "", // 当前试题id
    };
  },
  components: {
    QuestionPreview,
    QuestionCheck,
  },

  methods: {
    // 获取学科的简单列表
    async getSubjectSimpleList() {
      // 获取学科前清空二级目录和标签的值
      this.form.catalogID = "";
      this.form.tags = "";
      const { data } = await subjectsSimple();
      this.subjectSimpleList = data;
    },
    // 根据学科获取目录的简单列表
    async getDirectorysSimpleList() {
      const { data } = await directorysSimple(this.form.subjectID);
      this.directorysSimpleList = data;
    },
    // 根据学科获取标签的简单列表
    async getTagsSimpleList() {
      const { data } = await tagsSimple({
        subjectID: this.form.subjectID,
      });
      this.tagsSimpleList = data;
    },
    // 获取录入人的简单列表
    async getUsersSimple() {
      const { data } = await usersSimple();
      this.usersSimpleList = data;
    },
    //获取城市列表
    getProvincesList() {
      this.provincesList = provinces();
    },
    //获取城市所在区域列表
    getCitysList() {
      this.citysList = citys(this.form.province);
    },
    // 新增试题按钮
    addQBtn() {
      this.$router.push("/questions/new");
    },
    //搜索按钮
    searchBtn() {
      this.form.page = 1;
      this.loadQuestions();
    },
    // 清除按钮
    clearBtn() {
      this.$refs.form.resetFields();
    },
    // 切换选项卡
    handleClick(tab) {
      this.loadQuestions();
    },
    // 每页条目数变化时
    handleSizeChange(pagesize) {
      this.form.page = 1;
      this.form.pagesize = pagesize;
      this.loadQuestions();
    },
    // 切换页面时
    handleCurrentChange(page) {
      this.form.page = page;
      this.loadQuestions();
    },
    // *********************************渲染数据列表*********************************
    async loadQuestions() {
      // 筛选form中有值的属性
      let validForm = {};
      for (let key in this.form) {
        if (this.form[key] !== "" && this.form[key] !== "all2134234564234") {
          validForm[key] = this.form[key];
        }
      }
      const { data } = await choice(validForm);
      this.tableData = data.items;
      console.log(this.tableData);
      this.alertText = `数据一共${data.counts}条`;
      this.total = data.counts;
    },
    // *****************************************************************************

    // 将后台的数字转化成对应的字符串
    questionTypeFormatter(row) {
      if (row.questionType === "1") {
        return "单选";
      } else if (row.questionType === "2") {
        return "多选";
      } else if (row.questionType === "3") {
        return "简答";
      }
    },
    difficultyFormatter(row) {
      if (row.difficulty === "1") {
        return "简单";
      } else if (row.difficulty === "2") {
        return "一般";
      } else if (row.difficulty === "3") {
        return "困难";
      }
    },
    chkStateFormatter(row) {
      if (row.chkState === 0) {
        return "待审核";
      } else if (row.chkState === 1) {
        return "已审核";
      } else if (row.chkState === 2) {
        return "已拒绝";
      }
    },
    publishStateFormatter(row) {
      if (row.publishState === 0 && row.chkState === 1) {
        return "已下架";
      } else if (row.publishState === 1 && row.chkState === 1) {
        return "已发布";
      } else if (row.chkState === 0 || row.chkState === 2) {
        return "待发布";
      }
    },
    // 预览按钮
    async previewBtn(row) {
      const { data } = await detail({ id: row.id });
      this.currentData = data;
      this.showPrevDialog = true;
      console.log("文章详情预览", this.currentData);
    },
    // 审核按钮
    async checkBtn(row) {
      this.currentData = row;
      this.showCheckDialog = true;
      console.log("当前行数据", this.currentData);
    },
    // 修改按钮
    async editBtn(row) {
      // 携带试题ID跳转到试题录入
      this.$router.push("/questions/new/?id=" + row.id);
    },
    // 上架&下架按钮
    downBtn(row) {
      console.log();
      let text = row.publishState === 1 ? "下架" : "上架";
      this.$confirm("您确认" + text + "这道题目?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          await choicePublish({
            publishState: row.publishState ? 0 : 1,
            id: row.id,
          });
          this.loadQuestions();
          this.$message({
            type: "success",
            message: text + "成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消" + text,
          });
        });
    },
    // 删除按钮
    deleteBtn(row) {
      this.$confirm("此操作将永久删除题目,是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          // 基础题库删除api
          await remove(row);
          this.loadQuestions();
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
  },
  created() {
    // 页面加载渲染一次
    this.loadQuestions();
  },
};
</script>

<style scoped lang='scss'>
.container {
  .el-select {
    width: 100%;
  }
  .provCity {
    display: flex;
    .provinceSelect,
    .citysSelect {
      .el-form-item__content {
        margin-left: unset;
      }
    }
    .provinceSelect {
      margin-right: 10px;
    }
  }
}
</style>
