<template>
  <div class="container">
    <!-- 弹层 -->
    <el-dialog
      title="浏览文章"
      :visible="showDialogP"
      width="800px"
      @close="btnCancel"
    >
      <div>
        <h2 style=" padding: 0; margin: 0;">{{ list.title }}</h2>
        <div style="  margin-top: 10px; padding-bottom: 10px;">
          <span style="margin-left: 10px;">{{
            list.createTime | parseTimeByString("{y}-{m}-{d} {h}:{i}:{s}")
          }}</span>
          <span style="margin-left: 10px;">{{ username }}</span>
          <span style="margin-left: 10px;">
            <i class="el-icon-view" style="margin-right: 10px;"></i>
            {{ list.visits }}
          </span>
        </div>
        <div class="articleBody" v-html="list.articleBody"></div>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { detail } from "@/api/hmmm/articles";

export default {
  components: {},
  props: {
    showDialogP: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      list: [],
      username: "",
    };
  },
  created() {},

  methods: {
    btnCancel() {
      this.$emit("update:showDialogP", false); // 关闭
    },
    async detail(id, name) {
      const { data } = await detail({ id });
      this.list = data;
      this.username = name;
      // console.log(`---`, data);
    },
  },

  computed: {},

  watch: {},
};
</script>
<style lang="scss" scoped>
.container {
  .articleBody {
    background-color: #f5f5f5;
    padding: 10px;
    border-top: 1px dashed #ccc;
  }
}
</style>
