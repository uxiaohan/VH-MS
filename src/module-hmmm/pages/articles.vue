<template>
  <div>
    <el-card class="box-card">
      <!-- 头部搜素框 -->
      <el-row
        :gutter="20"
        type="flex"
        align="center"
        justify="space-between"
        style="margin-left: 20px; width: 100%"
      >
        <el-col>
          <el-form inline>
            <el-form-item label="关键字" style="margin-right: 5%">
              <el-input
                v-model="page.keyword"
                placeholder="根据文章标题搜索"
              ></el-input>
            </el-form-item>

            <el-form-item label="状态">
              <el-select v-model="state" placeholder="请选择">
                <el-option label="启用" value="1"></el-option>
                <el-option label="禁止" value="0"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button size="small" @click="resetForm()">清除</el-button>
              <el-button type="primary" size="small" @click="submitForm()"
                >搜索</el-button
              >
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="6">
          <el-row type="flex" justify="end">
            <el-button
              type="success"
              size="small"
              icon="el-icon-edit"
              @click="showDialog = true"
              >新增技巧</el-button
            >
          </el-row>
        </el-col>
      </el-row>
      <!-- /头部搜素框 -->
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
        <el-table-column
          label="序号"
          type="index"
          :index="getIndex"
          width="80"
        />
        <el-table-column width="400" label="文章标题">
          <template slot-scope="scope">
            <span>{{ scope.row.title }}</span>
            <el-button
              v-if="scope.row.videoURL"
              type="button"
              class="typeface icons"
              icon="el-icon-film"
              @click="videos(scope.row.videoURL)"
            ></el-button>
          </template>
        </el-table-column>
        <el-table-column style="min-width: 80px" prop="visits" label="阅读数">
        </el-table-column>
        <el-table-column style="min-width: 80px" prop="username" label="录入人">
        </el-table-column>
        <el-table-column style="min-width: 80px" label="录入时间">
          <template slot-scope="scope">
            <span>{{
              scope.row.createTime
                | parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")
            }}</span>
          </template>
        </el-table-column>
        <el-table-column style="min-width: 100px" label="状态">
          <template slot-scope="scope">
            <span>{{ scope.row.state === 1 ? `已启用` : `已禁用` }}</span>
          </template>
        </el-table-column>
        <el-table-column width="180" label="操作">
          <template slot-scope="scope">
            <el-button
              type="button"
              class="typeface"
              @click="operation('browse', scope.row.id, scope.row.username)"
              >浏览</el-button
            >
            <el-button
              type="button"
              class="typeface"
              @click="operation('switch', scope.row.id, scope.row.state)"
              >{{ scope.row.state === 1 ? `禁用` : `启用` }}</el-button
            >
            <el-button
              type="button"
              class="typeface"
              :class="{ disableds: scope.row.state === 1 }"
              @click="operation('delete', scope.row.id)"
              :disabled="scope.row.state === 1"
              >修改</el-button
            >
            <el-button
              type="button"
              class="typeface"
              :class="{ disableds: scope.row.state === 1 }"
              @click="operation('modify', scope.row.id)"
              :disabled="scope.row.state === 1"
              >删除</el-button
            >
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
          :page-size="10"
          layout="  prev, pager, next,sizes,jumper"
          :total="page.counts"
        >
        </el-pagination>
      </el-row>
      <!-- /分页 -->
    </el-card>
    <!-- 视屏 -->
    <div class="VideoBox" v-if="Showvideo">
      <el-button
        icon="el-icon-close"
        class="videoIcon"
        circle
        @click="Showvideo = false"
      ></el-button>
      <video
        controls
        ref="video"
        :src="videoUrl"
        autoplay="autoplay"
        loop="loop"
        class="videos"
      ></video>
    </div>
    <!-- 弹出层 -->

    <!-- 编辑 添加 -->
    <ArticlesAdd
      ref="JNPFForm"
      :showDialog.sync="showDialog"
      @addDepts="lists"
    />
    <!-- 浏览 -->
    <ArticlesPreview :showDialogP.sync="showDialogP" ref="detail" />
    <!-- 弹出层 -->
  </div>
</template>
<script>
import { list, changeState, remove } from "@/api/hmmm/articles";
import ArticlesAdd from "@/module-hmmm/components/articles-add.vue";
import ArticlesPreview from "@/module-hmmm/components/articles-preview.vue";

export default {
  components: { ArticlesPreview, ArticlesAdd },
  props: {},
  data() {
    return {
      showDialog: false, //新增技巧弹出层
      showDialogP: false, //浏览
      list: [], //文章列表
      page: {
        page: 1, //当前页数
        pagesize: 10, //页尺寸
        counts: 0,

        keyword: "", //关键字
      },
      state: "", //状态 1 开启 0 屏蔽 启用 禁止
      Showvideo: false, //视屏盒子开关
      videoUrl: "", //视屏地址
    };
  },
  created() {
    this.lists();
  },

  methods: {
    // 文章列表获取
    async lists() {
      const { data } = await list(this.page);
      console.log(data);
      this.page.counts = data.counts;
      this.list = data.items;
    },

    // 分页功能
    handleSizeChange(val) {
      this.page.pagesize = val;
      this.lists();
    },
    handleCurrentChange(val) {
      this.page.page = val;
      this.lists();
    },
    // 点击视屏
    videos(url) {
      this.videoUrl = url;
      this.Showvideo = true;
    },
    // 清空表单
    resetForm() {
      this.state = "";
      this.page.keyword = "";
      delete this.page.state;
    },
    //搜索
    submitForm() {
      // stae === "" ? (tate = 1) : state;

      if (this.state !== "") {
        const state = this.state;
        this.page = { ...this.page, state };
        console.log(this.state);
        this.lists();
      } else {
        this.lists();
      }
    },
    // 状态 操作 增删改查
    async operation(val, id, state) {
      if (val === "browse") {
        this.showDialogP = true;
        // 浏览
        this.$refs.detail.detail(id, state);
      } else if (val === `switch`) {
        //启用 或禁用
        console.log(state);
        try {
          await changeState({
            id,
            state: !state ? 1 : 0,
          });
          this.lists();
          this.$message({
            showClose: true,
            message: "操作成功",
            type: "success",
            duration: 1500,
          });
        } catch (err) {
          this.$message.error({
            message: "操作失败",
            showClose: true,
            duration: 1500,
          });
        }
        this.lists();
        // console.log("启用", id);
      } else if (val === "delete") {
        // 修改
        this.showDialog = true;
        this.$refs.JNPFForm.delete(id);
      } else if (val === "modify") {
        // 删除
        this.$confirm("此操作将永久删除该文章, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }).then(async () => {
          console.log(id);
          try {
            await remove({
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
            this.lists();
          } catch (err) {}
        });
      }
    },
  },
  computed: {
    // 序号
    getIndex() {
      return (this.page.page - 1) * this.page.pagesize + 1;
    },
  },

  watch: {},
};
</script>
<style lang="scss" scoped>
.box-card {
  margin: 20px;
  .typeface {
    margin-left: 10px;
    color: #409eff;
    border: none;
    margin-left: 10px;
    padding: 0;
  }
  .disableds {
    color: #c0c4cc;
  }
  .icons {
    color: blue;
    font-size: 18px;
  }
}

.VideoBox {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  margin: auto;
  width: 100vw;
  height: 100vh;
  z-index: 9999;
  background: rgba(0, 0, 0, 0.3);
  .videos {
    position: fixed;
    top: 0;
    right: 0;
    left: 0;
    bottom: 0;
    margin: auto;
    width: 800px;
    height: 600px;
  }
  .videoIcon {
    position: fixed;
    right: 50%;
    top: 30px;
    z-index: 9999;
    font-size: 30px;
    background: rgba(0, 0, 0, 0.5);
    border: none;
    color: #fff;
  }
}
</style>
