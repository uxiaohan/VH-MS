<template>
  <div class="container">
    <el-card>
      <el-form
        ref="searchFormRef"
        :model="query"
        :inline="true"
        v-model="formInline"
        class="demo-form-inline"
      >
        <el-row>
          <el-form-item class="search-input" prop="tags" label="标签名称">
            <el-input
              size="small"
              class="search"
              v-model="query.tags"
              placeholder="请输入"
              @keyup.enter.native="searchHandle"
            ></el-input>
          </el-form-item>
          <el-form-item class="search-input" prop="province" label="城市">
            <el-select
              class="search"
              v-model="query.province"
              size="small"
              @change="getcitys"
              @keyup.enter.native="searchHandle"
              placeholder="请选择"
            >
              <el-option
                v-for="(item, index) in citySelect.province"
                :key="index"
                :label="item"
                :value="item"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item class="search-input" prop="city" label="地区">
            <el-select
              class="search"
              v-model="query.city"
              size="small"
              @keyup.enter.native="searchHandle"
              placeholder="请选择"
            >
              <el-option
                v-for="(item, index) in citySelect.cityArea"
                :key="index"
                :label="item"
                :value="item"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item class="search-input" prop="shortName" label="企业简称">
            <el-input
              class="search"
              v-model="query.shortName"
              size="small"
              @keyup.enter.native="searchHandle"
            ></el-input>
          </el-form-item>
          <el-form-item class="zhuangtai" prop="state" label="状态">
            <el-select
              class="search-zhuangtai"
              v-model="query.state"
              size="small"
              @keyup.enter.native="searchHandle"
              placeholder="请选择"
            >
              <el-option
                v-for="item in getstatus"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button
              class="btn-clear"
              size="small"
              type="default"
              @click="clearForm"
              >清除</el-button
            >
          </el-form-item>
          <el-form-item>
            <el-button
              size="small"
              type="primary"
              @click="searchHandle"
              @keyup.enter.native="searchHandle"
              >搜索</el-button
            >
          </el-form-item>
          <el-button
            type="success"
            size="small"
            @click="showDialog('add')"
            class="btn-submit"
            icon="el-icon-edit"
            >新增企业</el-button
          >
        </el-row>
      </el-form>
      <!-- 记录提示信息 -->
      <el-alert
        show-icon
        :closable="false"
        class="himt"
        :title="`共${total}条记录`"
        type="info"
      >
      </el-alert>
      <!-- 表格区域 -->
      <el-table :data="companysList">
        <el-table-column
          align="center"
          prop="id"
          label="序号"
        ></el-table-column>
        <el-table-column
          align="center"
          prop="number"
          label="企业编号"
        ></el-table-column>
        <el-table-column
          align="center"
          prop="shortName"
          label="企业简称"
        ></el-table-column>
        <el-table-column
          align="center"
          prop="tags"
          label="标签"
        ></el-table-column>
        <el-table-column
          align="center"
          prop="creatorID"
          label="创建者"
        ></el-table-column>
        <el-table-column
          align="center"
          prop="addDate"
          label="创建日期"
        ></el-table-column>
        <el-table-column
          align="center"
          prop="remarks"
          label="备注"
        ></el-table-column>
        <el-table-column align="center" prop="state" label="状态">
          <template slot-scope="scope">
            <span>{{ scope.row.state ? `启用` : `禁止` }}</span>
            <!-- <span>{{ scope.row.state }}</span> -->
          </template>
        </el-table-column>
        <el-table-column align="center" label="操作" width="220px">
          <template slot-scope="scope">
            <el-button
              size="medium"
              plain
              circle
              type="primary"
              icon="el-icon-edit"
              @click="showDialog(scope.row)"
            ></el-button>
            <el-tooltip
              class="item"
              effect="dark"
              :content="`${scope.row.state === 0 ? '启用' : '禁用'}`"
              placement="top"
            >
              <el-button
                size="medium"
                plain
                circle
                :type="`${scope.row.state === 0 ? 'success' : 'warning'}`"
                :icon="`el-icon-${scope.row.state ? 'check' : 'close'}`"
                @click="changeState(scope.row.id, scope.row.state)"
              >
              </el-button>
            </el-tooltip>
            <el-button
              size="medium"
              plain
              circle
              type="danger"
              icon="el-icon-delete"
              @click="deleteCompany(scope.row)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 底层分页部分 -->
      <el-row class="page-bottom" type="flex" justify="end" align="middle">
        <el-pagination
          background
          :current-page="query.page"
          :page-sizes="[10, 20, 30, 50]"
          :page-size="query.pagesize"
          layout="prev, pager, next,sizes,jumper"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        >
        </el-pagination>
      </el-row>
    </el-card>
    <companys-add
      :company-dialog="companyDialog"
      :company-form="companyForm"
      :titleText="titleText"
      @closeDialog="closeDialog"
      @getCompanysList="getCompanysList"
    ></companys-add>
  </div>
</template>

<script>
import { list, detail, remove, disabled } from "@/api/hmmm/companys.js";
import { provinces, citys } from "@/api/hmmm/citys.js";
import { status } from "@/api/hmmm/constants.js";
import CompanysAdd from "@/module-hmmm/components/companys-add.vue";
import dayjs from "dayjs";

export default {
  name: "CompanysList",
  data() {
    return {
      formInline: {
        user: "",
        region: "",
      },
      companysList: [],
      query: {
        page: 1,
        pagesize: 10,
      },
      total: 0,
      citySelect: {
        province: [],
        cityArea: [],
      },
      companyDialog: false,
      companyForm: {
        isFamous: "",
        shortName: "",
        company: "",
        province: "",
        city: "",
        tags: "",
        remarks: "",
      },
      titleText: "",
    };
  },
  created() {
    var that = this;
    this.getCompanysList();
    this.getprovinces();

    document.onkeydown = function (e) {
      if (window.event.keycode == 13) {
        that.searchHandle();
      }
    };
  },
  methods: {
    // 获取企业列表
    async getCompanysList() {
      const res = await list(this.query);
      if (res.status != 200)
        return this.$message.error("获取企业管理列表数据失败");

      res.data.items.forEach((item) => {
        item.addDate = dayjs(item.addDate).format("YYYY-MM-DD HH:mm:ss");
      });

      this.companysList = res.data.items;
      this.total = res.data.counts;
    },
    // 每页显示数量
    handleSizeChange(pagesize) {
      this.query.pagesize = pagesize;
      this.getCompanysList();
    },
    // 跳转下一页
    handleCurrentChange(page) {
      this.query.page = page;
      this.getCompanysList();
    },
    // 获取省份
    getprovinces() {
      this.citySelect.province = provinces();
    },
    // 获取城市
    getcitys() {
      this.citySelect.cityArea = citys(this.query.province);
      this.companyForm.city = this.citySelect.cityArea[0];
    },
    // 清除状态
    clearForm() {
      this.$refs.searchFormRef.resetFields();
      this.getcitys();
      this.getCompanysList();
    },
    closeDialog(val) {
      this.companyDialog = val;
      // 重置清空
      this.companyForm = {
        isFamous: "",
        shortName: "",
        company: "",
        province: "",
        city: "",
        tags: "",
        remarks: "",
      };
    },
    showDialog(data) {
      if (data === "add") {
        this.titleText = "添加";
      } else {
        this.titleText = "编辑";
        this.getCompanyInfo(data);
      }
      this.companyDialog = true;
    },
    async getCompanyInfo(data) {
      const res = await detail(data);
      if (res.status != 200) return this.$message.error("获取详情失败!");
      this.companyForm = res.data;
    },
    async deleteCompany(data) {
      const confirmResult = await this.$confirm(
        "此操作将永久删除该企业, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);

      if (confirmResult !== "confirm") {
        return this.$message.info("您取消了删除");
      }

      const res = await remove(data);
      if (res.status !== 200) return this.$message.error("删除失败！");
      this.$message.success("删除成功！");
      this.getCompanysList();
    },
    // 查询
    searchHandle() {
      this.query.page = 1;
      this.getCompanysList();
    },
    // 改变状态
    changeState(id, value) {
      var data = {
        id: id,
        state: value ? 0 : 1,
      };

      this.$confirm(`是否确定修改?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(async () => {
          await disabled(data);
          this.getCompanysList();
        })
        .catch((err) => err);
    },
  },
  computed: {
    getstatus() {
      return status;
    },
  },
  components: {
    CompanysAdd,
  },
};
</script>

<style lang="scss" scoped>
.container {
  position: relative;
  background-color: #fff;
  margin: 20px 20px;
  .search-input {
    margin-left: 15px;
    .search {
      width: 330px;
    }
  }
  .zhuangtai {
    margin-left: 45px;
    .search-zhuangtai {
      width: 330px;
    }
  }
  .btn-clear {
    margin-left: 27px;
  }
  .btn-submit {
    position: absolute;
    top: 70px;
    right: 40px;
  }
  .himt {
    margin-bottom: 20px;
  }
  .page-bottom {
    margin-top: 20px;
  }
}
</style>
