<template>
  <!-- <div class='container'>添加文章对话框</div> -->
  <!-- 弹层 -->
  <el-dialog
    :title="title"
    :visible="showDialog"
    width="800px"
    @close="btnCancel"
  >
    <!-- 文章标题 -->
    <el-form ref="form" :model="form" :rules="rules" label-width="85px">
      <el-form-item label="文章标题" prop="title">
        <el-input placeholder="请输入文章内容" v-model="form.title" />
      </el-form-item>
      <!-- 富文本 -->
      <el-form-item label="文章内容" prop="articleBody">
        <quill-editor
          v-model="form.articleBody"
          ref="myQuillEditor"
          class="ql-editor-class"
          :options="editorOption"
        >
        </quill-editor>
        <el-upload
          style="display: none"
          class="quill-picture-uploader"
          action="http://localhost:8080/project/uploadPic"
        >
          <el-button size="small" type="primary"></el-button>
          <div slot="tip" class="el-upload__tip">
            <!-- 只能上传jpg/png文件，且不超过500kb -->
          </div>
        </el-upload>
      </el-form-item>
      <!-- 视屏地址 -->
      <el-form-item label="视屏地址" style="margin-top:25px;">
        <el-input placeholder="请输入视屏地址" v-model="form.videoURL" />
      </el-form-item>

      <el-form-item>
        <!-- 底部 -->
        <el-row type="flex" justify="end">
          <el-button size="medium " @click="btnCancel">取消</el-button>
          <el-button size="medium " type="primary" @click="btnOK"
            >确定</el-button
          >
        </el-row>
      </el-form-item>
    </el-form>
  </el-dialog>

  <!-- /弹层 -->
</template>
<script>
import { quillEditor } from "vue-quill-editor";
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";
import "quill/dist/quill.bubble.css";
import { add, detail, update } from "@/api/hmmm/articles";
export default {
  components: { quillEditor },
  props: {
    showDialog: {
      type: Boolean,
      required: true,
    },
  },

  // 上传图片部分
  data() {
    const toolbarOptions = [
      // 工具栏配置, 默认是全部
      ["bold"],
      ["italic"],
      ["underline"],
      ["strike"],
      [{ list: "ordered" }, { list: "bullet" }],
      ["blockquote"],
      ["code-block", "image", "link"],
    ];
    return {
      // 表单 数据
      form: {
        title: "",
        articleBody: "",
        videoURL: "",
      },
      rules: {
        title: [{ required: true, message: "请输入文章内容", trigger: "blur" }],
        articleBody: [
          { required: true, message: "请输入文章标题", trigger: "blur" },
        ],
      },
      //图片url
      urlList: [],
      //正文
      content: "",
      //富文本配置
      editorOption: {
        placeholder: "",
        theme: "snow",
        modules: {
          toolbar: {
            container: toolbarOptions, //自定义工具栏，略
          },
        },
      },
    };
  },

  created() {},

  methods: {
    // 弹出层取消
    btnCancel() {
      this.form = {
        title: "",
        articleBody: "",
        videoURL: "",
      };
      this.$refs.form.resetFields(); // 重置校验字段
      this.$emit("update:showDialog", false); // 关闭
    },
    // 弹出层确定
    btnOK() {
      this.$refs.form.validate(async (isOK) => {
        if (isOK) {
          if (this.form.id) {
            console.log(`修改`);
            // 编辑

            await update(this.form);
          } else {
            console.log(`新增`);
            // 新增调用
            console.log(`新增`, this.form);
            await add(this.form);
          }
          // update:props名称
          this.$emit(`addDepts`);
          this.$message({
            showClose: true,
            duration: 1500,
            type: "success",
            message: "操作成功",
          });
          this.$emit("update:showDialog", false);
        }
      });
    },
    // 编辑
    async delete(id) {
      const { data } = await detail({ id });

      this.form = data;
    },
  },

  computed: {
    title() {
      return this.form.id ? `修改文章` : `编辑文章`;
    },
  },

  watch: {},
};
</script>
<style lang="scss" scoped>
::v-deep .ql-editor-class {
  .ql-toolbar {
    padding: 0;
    padding-left: 10px;
  }
  .ql-container {
    height: 200px;
  }
}
</style>
