<template>
  <div class="add-form">
    <el-dialog
      :title="`${titleText}企业`"
      :visible.sync="childDialog"
      width="50%"
      class="create-company"
      @close="changeCompanyDialogClosed"
    >
      <el-form
        :model="companyForm"
        :rules="companyFormRules"
        ref="companyFormRef"
        label-width="150px"
        style="width:80%"
      >
        <el-form-item label="企业名称" prop="shortName">
          <el-input v-model="companyForm.shortName"></el-input>
        </el-form-item>
        <el-form-item label="所属公司" prop="company">
          <el-input v-model="companyForm.company"></el-input>
          <p>https://www.tianyancha.com （在此可查询所属公司全称及简称）</p>
        </el-form-item>
        <el-form-item label="城市地区" prop="province">
          <el-select
            v-model="companyForm.province"
            placeholder="请选择"
            size="medium"
            @change="getcitys"
            style="width:49%; margin-right:2%"
          >
            <el-option
              v-for="(item, index) in citySelect.province"
              :key="index"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
          <el-select
            v-model="companyForm.city"
            placeholder="请选择"
            size="medium"
            style="width:49%"
          >
            <el-option
              v-for="(item, index) in citySelect.cityDate"
              :key="index"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="方向（企业标签）" prop="tags">
          <el-input v-model="companyForm.tags"></el-input>
        </el-form-item>
        <el-form-item label="备注" prop="remarks">
          <el-input
            type="textarea"
            :rows="2"
            placeholder="请输入内容"
            v-model="companyForm.remarks"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="$emit('closeDialog', false)">取 消</el-button>
        <el-button type="primary" @click="submitHandle">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { add, update } from "@/api/hmmm/companys.js";
import { provinces, citys } from "@/api/hmmm/citys.js";
export default {
  name: "CompanysAdd",
  props: {
    companyDialog: {
      type: Boolean,
      required: true,
    },
    companyForm: {
      type: Object,
      required: true,
    },
    // 弹出层标题文本
    titleText: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      citySelect: {
        province: [], // 所有城市
        cityDate: [], // 城市下的地区
      },
      childDialog: false,
      companyFormRules: {
        shortName: [
          { required: true, message: "企业简称不能为空", trigger: "blur" },
        ],
        company: [
          { required: true, message: "所属公司不能为空", trigger: "blur" },
        ],
        province: [{ required: true, message: "请选择省份", trigger: "blur" }],
        tags: [{ required: true, message: "请输入标签", trigger: "blur" }],
        remarks: [{ required: true, message: "备注不能为空", trigger: "blur" }],
      },
    };
  },
  watch: {
    // 定一个计算属性，监听 companyDialog 的值发生改变并返回更新到子组件的数据
    companyDialog: function() {
      this.childDialog = this.companyDialog
    },
    // childDialog 的数据发送到父组件，在父组件监听这个自定义的事件，来接收子组件传过来的数据
    childDialog: function() {
      this.$emit('closeDialog', this.childDialog)
    }
  },
  created() {
    this.getprovinces()
  },
  methods:{
    changeCompanyDialogClosed() {
      this.$refs.companyFormRef.resetFields()
      // 获取城市下地区数据，以清空该表单
      this.getcitys()
      this.childDialog = false
    },
     getprovinces() {
      this.citySelect.province = provinces()
    },
     getcitys() {
      this.citySelect.cityDate = citys(this.companyForm.province)
      this.companyForm.city = this.citySelect.cityDate[0]
    },
    submitHandle() {
      this.$refs.companyFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.info('请输入企业管理的基本信息')
        } else {
          const data = { ...this.companyForm, isFamous: true }
          console.log(data)

          if (this.companyForm.id) {
            const res = await update(data)

            if (res.status !== 200) return this.$message.error('更新失败！')
            this.$message.success('更新成功！')
            // 刷新列表
            this.$emit('getCompanysList')
            // 关闭弹出层
            this.$emit('closeDialog', false)
          } else {
            const res = await add(data)

            if (res.status !== 200) return this.$message.error('添加失败！')
            this.$message.success('添加成功！')
            // 刷新列表
            this.$emit('getCompanysList')
            // 关闭弹出层
            this.$emit('closeDialog', false)
          }
        }
      })
    }
  }
};
</script>

<style lang="scss" scoped>
.dialog-footer{

  text-align: center;
    //  margin-right:180px;
}
</style>
