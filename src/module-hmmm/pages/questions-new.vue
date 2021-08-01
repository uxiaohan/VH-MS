<template>
  <div class="container">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>试题录入</span>
      </div>
      <div>
        <el-form
          ref="quesForm"
          :model="quesForm"
          :rules="quesFormRuls"
          label-width="120px"
        >
          <!-- 学科数组列表 -->
          <el-form-item label="学科：" prop="subjectID">
            <el-select
              @focus="getSubData"
              v-model="quesForm.subjectID"
              placeholder="请选择"
              class="han400"
            >
              <el-option
                v-for="itm in subDataArr"
                :label="itm.label"
                :key="itm.value"
                :value="itm.value"
              />
            </el-select>
          </el-form-item>
          <!-- 目录数组列表 -->
          <el-form-item label="目录：" prop="catalogID">
            <el-select
              @focus="getDirData"
              v-model="quesForm.catalogID"
              placeholder="请选择"
              class="han400"
            >
              <el-option
                v-for="itm in dirDataArr"
                :label="itm.label"
                :key="itm.value"
                :value="itm.value"
              />
            </el-select>
          </el-form-item>
          <!-- 企业数组列表 -->
          <el-form-item label="企业：" prop="enterpriseID">
            <el-select
              @focus="getCompnyData"
              v-model="quesForm.enterpriseID"
              placeholder="请选择"
              class="han400"
            >
              <el-option
                v-for="itm in compnyDataArr"
                :label="itm.shortName"
                :key="itm.id"
                :value="itm.id"
              />
            </el-select>
          </el-form-item>
          <!-- 城市数组列表 -->
          <div style="display: flex; overflow: hidden">
            <el-form-item label="城市：" prop="province">
              <el-select v-model="quesForm.province" placeholder="请选择">
                <el-option
                  v-for="(itm, idx) in provinceDataArr"
                  :label="itm"
                  :key="idx"
                  :value="itm"
                />
              </el-select>
            </el-form-item>
            <el-form-item label-width="14px" prop="city">
              <el-select
                v-model="quesForm.city"
                placeholder="请选择"
                @focus="getCitysArr"
              >
                <el-option
                  v-for="(itm, idx) in citysDataArr"
                  :label="itm"
                  :key="idx"
                  :value="itm"
                />
              </el-select>
            </el-form-item>
          </div>
          <!-- 方向数组列表 -->
          <el-form-item label="方向：" prop="direction">
            <el-select
              v-model="quesForm.direction"
              placeholder="请选择"
              class="han400"
            >
              <el-option
                v-for="(itm, idx) in directionDataArr"
                :label="itm"
                :key="idx"
                :value="itm"
              />
            </el-select>
          </el-form-item>
          <!-- 题型单选框 -->
          <el-form-item label="题型：" prop="questionType">
            <el-radio-group @change="questionTypeChan" v-model="questionType">
              <el-radio label="单选"></el-radio>
              <el-radio label="多选"></el-radio>
              <el-radio label="简答"></el-radio>
            </el-radio-group>
          </el-form-item>
          <!-- 难度单选框 -->
          <el-form-item label="难度：" prop="difficulty">
            <el-radio-group v-model="difficulty">
              <el-radio label="简单"></el-radio>
              <el-radio label="一般"></el-radio>
              <el-radio label="困难"></el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="题干：" prop="question">
            <quill-editor
              class="hanEditTool"
              :options="editorOption"
              v-model="quesForm.question"
              @change="editorRulsFn('question')"
              @blur="editorRulsFn('question')"
            />
          </el-form-item>
          <el-form-item
            label="选项"
            prop="difficulty"
            v-if="quesForm.questionType === '2'"
          >
            <el-checkbox-group v-model="checkAnswer" class="hanRadio">
              <div class="hanIpt" v-for="(itm, idx) in answerArr" :key="idx">
                <el-checkbox :label="idx"
                  >{{ itm.code }}： <el-input v-model="itm.title" />
                </el-checkbox>
                <el-upload
                  class="avatar-uploader"
                  action="https://jsonplaceholder.typicode.com/posts/"
                  :show-file-list="false"
                  :on-success="handleAvatarSuccess(itm)"
                >
                  <img v-if="itm.img" :src="itm.img" class="avatar" />
                  <span v-else>上传图片</span>

                  <i
                    class="el-icon-circle-close avatar-i"
                    @click.stop="cleaImg(itm)"
                  ></i>
                </el-upload>
              </div>
            </el-checkbox-group>
            <div>
              <el-button type="danger" size="small" @click="addAnser"
                >+增加选项与答案</el-button
              >
            </div>
          </el-form-item>
          <el-form-item label="选项：" v-if="quesForm.questionType === '1'">
            <el-radio-group v-model="radioAnser" class="hanRadio">
              <div class="hanIpt" v-for="(itm, idx) in answerArr" :key="idx">
                <el-radio :label="idx"
                  >{{ itm.code }}： <el-input v-model="itm.title" />
                </el-radio>
                <el-upload
                  class="avatar-uploader"
                  action="https://jsonplaceholder.typicode.com/posts/"
                  :show-file-list="false"
                  :on-success="handleAvatarSuccess(itm)"
                >
                  <img v-if="itm.img" :src="itm.img" class="avatar" />
                  <span v-else>上传图片</span>

                  <i
                    class="el-icon-circle-close avatar-i"
                    @click.stop="cleaImg(itm)"
                  ></i>
                </el-upload>
              </div>
            </el-radio-group>
            <div>
              <el-button disabled type="danger" size="small" @click="addAnser"
                >+增加选项与答案</el-button
              >
            </div>
          </el-form-item>
          <el-form-item label="解析视频：">
            <el-input v-model="quesForm.videoURL" class="han400"></el-input>
          </el-form-item>
          <el-form-item label="答案解析：" prop="answer">
            <quill-editor
              class="hanEditTool"
              :options="editorOption"
              v-model="quesForm.answer"
              @blur="editorRulsFn('answer')"
              @change="editorRulsFn('answer')"
            />
          </el-form-item>
          <el-form-item label="题目备注：">
            <el-input
              class="han400"
              type="textarea"
              :rows="4"
              v-model="quesForm.remarks"
            />
          </el-form-item>
          <el-form-item label="试题标签：">
            <el-select
              class="han400"
              @focus="getTagsData"
              v-model="theTags"
              multiple
              filterable
              allow-create
              default-first-option
              placeholder="请选择"
            >
              <el-option
                v-for="(item, idx) in tagsDataArr"
                :key="idx"
                :label="item.label"
                :value="item.label"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item size="large">
            <el-button
              :type="this.$route.query.id ? 'success' : 'primary'"
              @click="onSubmit"
              >{{ this.$route.query.id ? "确认修改" : "确认提交" }}</el-button
            >
          </el-form-item>
        </el-form>
      </div>
    </el-card>
  </div>
</template>

<script>
import { simple as subSmb } from "@/api/hmmm/subjects";
import { simple as dirSmb } from "@/api/hmmm/directorys";
import { simple as tagSmb } from "@/api/hmmm/tags";
import { list as compnyList } from "@/api/hmmm/companys";
import {
  detail as quesInfo,
  add as quesAdd,
  update as quesUpdate,
} from "@/api/hmmm/questions";
import { provinces as provincesArr, citys as citysArr } from "@/api/hmmm/citys";
import { direction as directionDataArr } from "@/api/hmmm/constants";
import {
  questionType as questionTypeFn,
  difficulty as difficultyFn,
} from "@/api/hmmm/constants";
export default {
  data() {
    return {
      // 待提交的Form数据
      quesForm: {
        subjectID: "",
        catalogID: "",
        enterpriseID: "",
        province: "",
        city: "",
        questionType: "",
        difficulty: "",
        question: "",
        direction: "",
        videoURL: "",
        answer: "",
        remarks: "",
        tags: [],
        options: [],
      },
      // 单选答案
      radioAnser: "",
      // 多选答案
      checkAnswer: [],
      // 学科数组列表
      subDataArr: [],
      // 目录数组列表
      dirDataArr: [],
      // 公司数组列表
      compnyDataArr: [],
      // 城市数组列表
      provinceDataArr: [],
      // 区域数组列表
      citysDataArr: [],
      // 方向数组列表
      directionDataArr: [],
      // 标签数组列表
      tagsDataArr: [],
      // Editor富文本输入框配置
      editorOption: {
        modules: {
          toolbar: [
            ["bold", "italic", "underline", "strike"], // toggled buttons
            [{ list: "ordered" }, { list: "bullet" }], //列表
            ["blockquote", "code-block", "image", "link"],
          ],
        },
        placeholder: "",
      },
      // 题目答案数组
      answerArr: [
        {
          code: "A",
          img: "",
          isRight: false,
          title: "",
        },
        {
          code: "B",
          img: "",
          isRight: false,
          title: "",
        },
        {
          code: "C",
          img: "",
          isRight: false,
          title: "",
        },
        {
          code: "D",
          img: "",
          isRight: false,
          title: "",
        },
      ],
      // 待提交的表单数据校验规则
      quesFormRuls: {
        subjectID: [{ required: true, trigger: "blur", message: "参数不完整" }],
        catalogID: [{ required: true, trigger: "blur", message: "参数不完整" }],
        enterpriseID: [
          { required: true, trigger: "blur", message: "参数不完整" },
        ],
        province: [{ required: true, trigger: "blur", message: "参数不完整" }],
        city: [{ required: true, trigger: "blur", message: "参数不完整" }],
        questionType: [
          { required: true, trigger: "blur", message: "参数不完整" },
        ],
        difficulty: [
          { required: true, trigger: "blur", message: "参数不完整" },
        ],
        question: [{ required: true, trigger: "blur", message: "参数不完整" }],
        direction: [{ required: true, trigger: "blur", message: "参数不完整" }],
        answer: [{ required: true, trigger: "blur", message: "参数不完整" }],
      },
    };
  },
  created() {
    this.$route.query.id && this.quesInfoFn(this.$route.query.id);
    // 初始化城市数组数据
    this.provinceDataArr = provincesArr();
    // 初始化方向数组数据
    this.directionDataArr = directionDataArr;
  },
  methods: {
    // 根据ID获取问题详情
    async quesInfoFn(id) {
      const { data: res } = await quesInfo({ id });
      this.quesForm = res;
      this.answerArr = res.options;
      console.log(this.answerArr);
      let checkOrradioAnser = "";
      switch (res.questionType) {
        case "1":
          checkOrradioAnser = "";
          this.answerArr.forEach((itm, idx) => {
            itm.isRight && (checkOrradioAnser = idx);
          });
          this.radioAnser = checkOrradioAnser;
          break;
        case "2":
          checkOrradioAnser = [];
          this.answerArr.forEach((itm, idx) => {
            itm.isRight && checkOrradioAnser.push(idx);
          });
          this.checkAnswer = checkOrradioAnser;
          break;
        default:
          break;
      }
    },
    // 获取城市City数据
    getCitysArr() {
      this.citysDataArr = citysArr(this.quesForm.province);
    },
    // 单选多选等变化事件
    questionTypeChan() {
      this.checkAnswer = [];
      this.radioAnser = "";
    },
    // 获取学科数组数据
    async getSubData() {
      const { data: res } = await subSmb();
      this.subDataArr = res;
    },
    // 获取目录列表  
    async getDirData() {
      const { data: res } = await dirSmb(this.quesForm.subjectID);
      this.dirDataArr = res;
    },
    // 获取公司列表
    async getCompnyData() {
      const { data: res } = await compnyList();
      this.compnyDataArr = res.items;
    },
    // 获取标签列表
    async getTagsData() {
      const { data: res } = await tagSmb();
      this.tagsDataArr = res;
    },
    // 表单提交事件
    onSubmit() {
      this.$refs.quesForm
        .validate()
        .then(async () => {
          const { data: res } = this.$route.query.id
            ? await quesUpdate(this.quesForm)
            : await quesAdd(this.quesForm);
          this.$route.query.id
            ? res.success && this.$message.success("修改成功")
            : res.id && this.$message.success("添加成功成功");
          this.$router.push("/questions/list");
        })
        .catch((err) => {
          console.log(err, "错误");
        });
    },
    // Editor富文本校验事件
    editorRulsFn(key) {
      this.$refs["quesForm"].validateField(key);
    },
    // 图片上传成功事件
    handleAvatarSuccess(itm) {
      // bind原理
      return (...fn) => {
        itm.img = URL.createObjectURL(fn[1].raw);
      };
    },
    // 上传图片按钮右上角清除图片事件
    cleaImg(itm) {
      itm.img = "";
    },
    // 多选题新增答案事件
    addAnser() {
      const itm = this.answerArr;
      itm.push({
        code: String.fromCharCode(65 + itm.length),
        img: "",
        isRight: false,
        title: "",
      });
    },
  },
  computed: {
    // 标签数组转字符串与字符串转数组事件（为了方便最终提交）
    theTags: {
      get(val) {
        return this.quesForm.tags != "" ? this.quesForm.tags.split(",") : "";
      },
      set(val) {
        this.quesForm.tags = val.join(",");
      },
    },
    // 字符串与value值的转换（为了方便最终提交）
    questionType: {
      set(val) {
        this.quesForm.questionType = questionTypeFn
          .filter((itm) => itm.label === val)[0]
          ["value"].toString();
      },
      get() {
        let _itms = questionTypeFn.filter(
          (itm) => itm.value === Number(this.quesForm.questionType)
        );
        return _itms.length > 0 ? _itms[0]["label"] : "";
      },
    },
    // 字符串与value值的转换（为了方便最终提交）
    difficulty: {
      set(val) {
        this.quesForm.difficulty = difficultyFn
          .filter((itm) => itm.label === val)[0]
          ["value"].toString();
      },
      get() {
        let _itms = (_itms = difficultyFn.filter(
          (itm) => itm.value === Number(this.quesForm.difficulty)
        ));
        return _itms.length > 0 ? _itms[0]["label"] : "";
      },
    },
  },
  watch: {
    // 答案单选框改变时，更改提交的答案的isRight
    radioAnser(val) {
      this.answerArr.forEach((itm) => {
        itm.isRight = false;
      });
      val != "" && (this.answerArr[val].isRight = true);
    },
    // 答案多选框改变时，更改提交的答案的isRight
    checkAnswer(val) {
      this.answerArr.forEach((itm) => {
        itm.isRight = false;
      });
      val.forEach((itm) => {
        this.answerArr[itm].isRight = true;
      });
    },
    // 答案选项数据改变时赋给待提交的Form表单 （为了方便最终提交）
    answerArr: {
      handler(val) {
        this.quesForm.options = val;
      },
      deep: true,
    },
  },
};
</script>
<style>
.avatar-i {
  position: absolute;
  right: 0;
  top: 0;
  -webkit-transform: translate(50%, -50%);
  transform: translate(50%, -50%);
  background: #fff;
  font-size: 18px;
  color: #999;
}
.avatar-uploader {
  display: inline-flex;
  align-items: center;
}
.hanRadio span {
  font-size: 14px;
}
.hanRadio .avatar-uploader {
  margin-left: 28px;
}

.hanRadio .hanIpt {
  margin-bottom: 18px;
  align-items: center;
  display: flex;
}
</style>
<style scope>
.avatar-uploader .el-upload {
  vertical-align: middle;
  line-height: 1;
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  width: 100px;
  height: 60px;
  line-height: 60px;
}
.avatar-uploader .el-upload:hover {
  border-color: #409EFF;
}
.avatar {
  width: 100px;
  height: 60px;
  display: block;
  overflow: hidden;
}
.box-card {
  margin: 20px;
}
.han400 {
  width: 400px;
}
.hanEditTool {
  height: 200px;
  margin-bottom: 68px;
}
</style>
