<template>
  <div class="container">
    <el-dialog
      title="新增目录"
      :visible.sync="orDiaShow"
      width="400px"
      center
      :before-close="diaClose"
    >
      <el-form
        :model="theform"
        :rules="formRuls"
        ref="theform"
        label-width="80px"
        size="small"
      >
        <el-form-item v-if="!routeId" label="所属学科" prop="subjectName">
          <el-select
            v-model="theform.subjectName"
            placeholder="请选择"
            @focus="getSelData"
          >
            <el-option
              v-for="itm in seleData"
              :label="itm.subjectName"
              :key="itm.id"
              :value="itm.subjectName"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="目录名称" prop="files">
          <el-input v-model="theform.files" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="diaClose">取 消</el-button>
        <el-button type="primary" @click="diaSuccess">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { list } from "@/api/hmmm/subjects";
export default {
  props: {
    orDiaShow: {
      type: Boolean,
      default: false,
    },
    oldData: {
      type: Object,
      defaule: null,
    },
    orUpdate: {
      type: Boolean,
      required: true,
    },
    routeId: {
      type: [Number, String],
      default: null,
    },
  },
  data() {
    return {
      theform: this.oldData || {
        subjectName: "",
        files: "",
        theId: "",
      },
      formRuls: {
        subjectName: [
          { required: true, message: "请选择学科", trigger: "blur" },
        ],
        files: [
          {
            required: true,
            message: "请输入目录名称",
            trigger: "blur",
          },
        ],
      },
      seleData: [],
    };
  },
  watch: {
    routeId: {
      handler(val) {
        console.log(val);
      },
      immediate: true,
    },
    oldData: {
      async handler(val) {
        if (val.name.length > 0) {
          await this.getSelData();
          this.theform.subjectName = val.subid;
          this.theform.files = val.name;
          this.theform.theId = val.id;
        }
      },
      deep: true,
    },
    orUpdate(val) {
      console.log(val);
    },
  },
  created() {},
  methods: {
    async getSelData() {
      const { data: res } = await list();
      this.seleData = res.items;
    },
    diaClose() {
      this.theform = {
        subjectName: "",
        files: "",
        theId: "",
      };
      this.$refs.theform.resetFields();
      this.$emit("disAbl", false);
    },
    diaSuccess() {
      let fnName = "addData";
      this.orUpdate === true && (fnName = "updateData");
      this.$refs.theform
        .validate()
        .then((res) => {
          const idx = this.seleData.findIndex(
            (itm) => itm.subjectName === this.theform.subjectName
          );

          this.$emit(fnName, {
            name: this.theform.files,
            subid: Number(this.routeId) || this.seleData[idx].id,
            id: this.theform.theId,
          });
        })
        .catch((err) => {
          console.log(err, "错误");
        });
    },
  },
};
</script>
<style>
.el-dialog__header {
  text-align: left !important;
}
</style>
<style scoped>
</style>