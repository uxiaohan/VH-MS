<template>
  <div class="container">
    <el-dialog
      title="题目预览"
      :visible="showPrevDialog"
      width="60%"
      @close="closePrevDialog"
      close-on-click-modal
    >
      <el-row :gutter="20">
        <el-col :span="6">【题型】: {{ currentData.questionType }}</el-col>
        <el-col :span="6">【编号】: {{ currentData.id }}</el-col>
        <el-col :span="6">【难度】: {{ currentData.difficulty }}</el-col>
        <el-col :span="6">【标签】: {{ currentData.tags }}</el-col>
        <el-col :span="6">【学科】: {{ currentData.subjectName }}</el-col>
        <el-col :span="6">【目录】: {{ currentData.directoryName }}</el-col>
        <el-col :span="6">【方向】: {{ currentData.direction }}</el-col>
        <el-col :span="24"><el-divider></el-divider></el-col>
        <el-col :span="24">【题干】: </el-col>
        <el-col :span="24"
          ><span v-html="currentData.question" style="color: blue"></span
        ></el-col>
        <el-col :span="24" v-if="currentData.questionType !== '简答'"
          >{{ currentData.questionType }}题 选项:
          (以下选中的选项为正确答案)</el-col
        >
        <div v-if="currentData.questionType === '单选'" class="radio">
          <el-col
            :span="24"
            v-for="(obj, ind) in currentData.options"
            :key="ind"
          >
            <el-radio v-model="radio" :label="obj.title"></el-radio>
          </el-col>
        </div>
        <div v-if="currentData.questionType === '多选'" class="checkbox">
          <el-checkbox-group v-model="checkList">
            <el-col
              :span="24"
              v-for="(obj, ind) in currentData.options"
              :key="ind"
            >
              <el-checkbox :label="obj.title"></el-checkbox>
            </el-col>
          </el-checkbox-group>
        </div>
        <el-col :span="24"><el-divider></el-divider></el-col>
        <el-col :span="24"
          >【参考答案】:
          <el-button type="danger" @click="showAnswerVideo = true" size="small"
            >视频答案预览</el-button
          ></el-col
        >
        <el-col :span="24" style="margin-top: 24px">
          <video
            controls
            @click="videoAnswerBtn"
            class="answerVideo"
            v-if="showAnswerVideo"
            :src="currentData.videoURL"
            style="width: 50%"
          ></video>
        </el-col>
        <el-col :span="24"><el-divider></el-divider></el-col>
        <el-col :span="24"
          >【答案解析】: <span v-html="currentData.answer || '无'"></span
        ></el-col>
        <el-col :span="24"><el-divider></el-divider></el-col>
        <el-col :span="24"
          >【题目备注】: <span v-html="currentData.remarks || '无'"></span
        ></el-col>
      </el-row>

      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="closePrevDialog">关 闭</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      showAnswerVideo: false, // 显示视频答案预览
      // radio:0, // 单选题
      // checkList: [], // 多选题
    };
  },
  props: {
    showPrevDialog: {
      type: Boolean,
      default: false,
    },
    currentData: {
      type: Object,
      required: true,
    },
  },
  methods: {
    closePrevDialog() {
      this.$emit("update:showPrevDialog", false);
    },

    videoAnswerBtn() {
      this.showAnswerVideo = true;
      var video = document.querySelector(".answerVideo");
      if (video.paused) {
        video.play();
      } else {
        video.pause();
      }
    },
  },
  created() {},
  computed: {
    radio: {
      get() {
        const radios = this.currentData.options;
        const ppp = radios.filter((item) => item.isRight !== 0);
        if (ppp === []) {
          return "";
        } else {
          return ppp[0].title;
        }
      },
      set() {
        return 0;
      },
    },
    checkList: {
      get() {
        const radios = this.currentData.options;
        const ppp = radios.filter((item) => item.isRight !== 0);
        if (ppp === []) {
          return "";
        } else {
          const val = [];
          ppp.forEach((item) => {
            val.push(item.title);
          });
          return val;
        }
      },
      set() {
        return 0;
      },
    },
  },
  watch: {
    "currentData.questionType": {
      handler() {
        if (this.currentData.questionType === "1") {
          return (this.currentData.questionType = "单选");
        } else if (this.currentData.questionType == "2") {
          return (this.currentData.questionType = "多选");
        } else if (this.currentData.questionType == "3") {
          return (this.currentData.questionType = "简答");
        }
      },
      immediate: true,
    },
    "currentData.difficulty": {
      handler() {
        if (this.currentData.difficulty === "1") {
          return (this.currentData.difficulty = "简单");
        } else if (this.currentData.difficulty == "2") {
          return (this.currentData.difficulty = "一般");
        } else if (this.currentData.difficulty == "3") {
          return (this.currentData.difficulty = "困难");
        }
      },
      immediate: true,
    },
  },
};
</script>

<style scoped lang="scss">
.container {
  .el-dialog__footer {
    text-align: center !important;
  }
}
</style>
