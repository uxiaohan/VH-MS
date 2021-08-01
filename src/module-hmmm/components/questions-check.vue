<template>
  <div class="container">
    <el-dialog
      title="题目审核"
      :visible="showCheckDialog"
      width="30%"
      @close="closeCheckDialog"
      close-on-click-modal
    >
      <el-form ref="form" :model="checkForm" label-width="80px">
        <el-form-item prop="chkState">
          <el-radio-group v-model="checkForm.chkState">
            <el-radio label="通过" value="1"></el-radio>
            <el-radio label="拒绝" value="2"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item prop="chkRemarks">
          <el-input
            type="textarea"
            placeholder="请输入审核意见"
            v-model="checkForm.chkRemarks"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="closeCheckDialog">取 消</el-button>
        <el-button type="primary" @click="checkBtn">确 定</el-button>
      </span></el-dialog
    >
  </div>
</template>

<script>
import { choiceCheck } from "@/api/hmmm/questions.js";
export default {
  props: {
    showCheckDialog: {
      type: Boolean,
      default: false,
    },
    currentData: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      checkForm: {
        chkState: "通过", // 1通过2拒接
        chkRemarks: "", // 原因(审核意见)
      },
    };
  },
  methods: {
    closeCheckDialog() {
      this.$emit("update:showCheckDialog", false);
       console.log(this.currentData);
    },
    async checkBtn() {
      let validForm = {
        id: this.currentData.id,
        chkState: this.checkForm.chkState === "通过" ? 1 : 2,
        chkRemarks: this.checkForm.chkRemarks,
      };
      if (this.checkForm.chkRemarks === "") {
        this.$message({
          showClose: true,
          message: "请输入审核意见!",
          type: "warning",
        });
      } else {
        await choiceCheck(validForm);
        this.$emit("update:showCheckDialog", false);
        this.$emit("renderTableData");
      }
    },
  },
};
</script>

<style scoped lang='less'></style>
