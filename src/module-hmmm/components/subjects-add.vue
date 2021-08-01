<template>
  <div class="container">
    <!-- 添加学科弹层 -->
    <el-dialog
      center
      width="400px"
      :visible="showDialog"
      @close="btnCancel"
      :title="row.id ? '修改学科' : '新增学科'"
    >
      <!-- 表单 -->
      <el-form
        ref="addSubject"
        label-width="80px"
        :model="formData"
        :rules="rules"
      >
        <el-form-item label="学科名称" prop="subjectName">
          <el-input
            style="width: 280px"
            placeholder="请输入学科名称"
            v-model="formData.subjectName"
          />
        </el-form-item>
        <el-form-item label="是否显示" prop="isFrontDisplay">
          <el-switch
            v-model="formData.isFrontDisplay"
            active-value="1"
            inactive-value="0"
          >
            >
          </el-switch>
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
        style="display: flex; justify-content: flex-end"
      >
        <el-button @click="btnCancel">取 消</el-button>
        <el-button type="primary" @click="btnOK">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { add, update } from "../../api/hmmm/subjects";

export default {
  name: "addSubject",
  props: {
    showDialog: {
      type: Boolean,
      default: true,
    },
    row: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      // 校验规则
      formData: {
        subjectName: "",
        isFrontDisplay: "1",
      },
      rules: {
        subjectName: [
          { required: true, message: "学科名称不能为空", trigger: "blur" },
          {
            min: 1,
            max: 10,
            message: "用户姓名为1-10位",
          },
        ],
      },
    };
  },
  created() {
    if (this.row.subjectName) {
      this.formData.subjectName = this.row.subjectName;
    }
  },

  methods: {
    // 点击确定时 校验整个表单
    async btnOK() {
      try {
        await this.$refs.addSubject.validate();
        console.log(this.row.id);
        // 调用修改接口
        if (this.row.id) {
          await update(this.row);
          this.$parent.showDialog = false;
          this.row.subjectName = this.formData.subjectName;
        } else {
          await add(this.formData); // 新增学科
          // 告诉父组件更新数据
          // this.$parent 可以直接调用到父组件的实例 实际上就是父组件this
          // this.$emit
          this.$parent.getSubjectList();
          this.$parent.showDialog = false;
        }
      } catch (error) {
        console.log(error);
      }
    },
    btnCancel() {
      // 重置原来的数据
      this.formData = {
        subjectname: "",
        isFrontDisplay: "",
      };
      this.$refs.addSubject.resetFields(); // 重置校验结果
      this.$emit("update:showDialog", false);
    },
  },
};
</script>

<style scoped lang='less'></style>
