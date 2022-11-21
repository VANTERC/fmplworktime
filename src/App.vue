<template>
  <div id="app">
    <div id="fm-content">
      <el-button
        type="primary"
        v-if="showbtn"
        style="
          z-index: 2000;
          position: absolute;
          right: 0;
          top: 0;
          margin-right: 150px;
        "
        @click="showDrawer"
        >查看本月未填写工时</el-button
      >
      <el-button
        type="primary"
        v-if="showbtn1"
        style="
          z-index: 2000;
          position: absolute;
          right: 0;
          top: 0;
          margin-right: 310px;
        "
        @click="showDrawer1"
        >批量填写工时</el-button
      >
      <el-dialog
        :visible.sync="dialogVisible"
        title="请输入非码账号密码"
        width="30%"
      >
        <div>
          账号：<el-input
            v-model="fmaccount"
            placeholder="请输入账号"
          ></el-input>
        </div>
        <div>
          密码：<el-input
            v-model="fmpassword"
            placeholder="请输入密码"
            type="password"
          ></el-input>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="handleOk">确 定</el-button>
        </span>
      </el-dialog>
      <el-drawer
        style="z-index: 99999"
        title="本月未填写工时"
        :visible.sync="drawer"
        :direction="direction"
        :before-close="drawerHandleClose"
      >
        <el-table
          v-loading="loading"
          :data="tableData"
          style="width: 100%"
          border
        >
          <el-table-column fixed prop="ownerName" label="姓名">
          </el-table-column>
          <el-table-column fixed prop="departmentName" label="部门">
          </el-table-column>
          <el-table-column fixed prop="count" label="未填写工时总天数">
          </el-table-column>
          <el-table-column prop="dateCells" label="未填写工时日期">
            <template>
              <el-tooltip placement="top">
                <div slot="content">{{ stsd }}</div>
                <span>{{ stsd }}</span>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </el-drawer>

      <el-drawer
        style="z-index: 99999"
        :visible.sync="drawer1"
        :direction="direction"
        size="60%"
        :before-close="drawerHandleClose1"
      >
        <div slot="title">
          <div style="display: flex">
            <div>
              <div style="color: red; display: flex; align-items: center">
                * 工时日期、工作描述可填可不填、项目编号、商户必填
              </div>
              <div style="color: red; display: flex; align-items: center">
                * 在tapd右上角<img
                  style="width: 36px; height: 36px"
                  src="https://vanterc.oss-cn-beijing.aliyuncs.com/WX20221121-143713.png"
                />内设置项目需求显示字段把项目编号勾选可不用手动填写项目编号
              </div>
            </div>
            <div>
              <img
                style="width: 300px; height: 100px"
                src="https://vanterc.oss-cn-beijing.aliyuncs.com/WX20221121-141734.png"
              />
            </div>
          </div>
          <div>
            <el-button type="primary" @click="batchok" :loading="btloading"
              >批量填写</el-button
            >
            商户：
            <el-select
              style="width: 200px"
              filterable
              v-model="partnersId"
              placeholder="请选择商户"
            >
              <el-option
                v-for="item in partnersList"
                :key="item.id"
                :label="item.name"
                :value="item.name"
              >
              </el-option>
            </el-select>
          </div>
          <div style="padding: 20px 20px">
            <el-progress
              :text-inside="true"
              :stroke-width="26"
              :percentage="percentage"
            ></el-progress>
          </div>
        </div>
        <div style="padding: 20px 20px" class="demo-drawer__content">
          <el-table
            border
            height="350"
            :data="currentPageTask"
            @selection-change="handleSelectionChange"
            style="width: 100%"
          >
            <el-table-column type="selection" width="55"> </el-table-column>
            <el-table-column fixed width="400" prop="taskname" label="需求任务">
            </el-table-column>
            <el-table-column width="300" prop="owner" label="处理人">
            </el-table-column>
            <el-table-column width="80" prop="developer" label="开发人员">
            </el-table-column>
            <el-table-column width="80" prop="testuser" label="测试人员">
            </el-table-column>
            <el-table-column width="300" prop="developer" label="项目编号">
              <template slot-scope="scope">
                <el-select
                  style="width: 250px"
                  filterable
                  v-model="scope.row.projectCode"
                  placeholder="请选择项目编号"
                >
                  <el-option
                    v-for="item in promData"
                    :key="item.projectCode"
                    :label="item.name + '-' + item.projectCode"
                    :value="item.projectCode"
                  >
                  </el-option>
                </el-select>
              </template>
            </el-table-column>
            <el-table-column
              width="220"
              prop="workTime"
              label="工作日期(可不填自动带入)"
            >
              <template slot-scope="scope">
                <el-date-picker
                  v-model="scope.row.workTime"
                  type="date"
                  placeholder="选择日期"
                >
                </el-date-picker>
              </template>
            </el-table-column>
            <el-table-column width="180" prop="consumeTime" label="消耗工时">
              <template slot-scope="scope">
                <el-input-number
                  size="mini"
                  v-model="scope.row.consumeTime"
                  :min="0"
                  :max="8"
                  label="消耗工时"
                ></el-input-number>
              </template>
            </el-table-column>
            <el-table-column width="180" prop="status" label="状态">
              <template slot-scope="scope">
                <el-select v-model="scope.row.status" placeholder="请选择状态">
                  <el-option
                    v-for="item in statusList"
                    :key="item.id"
                    :label="item.value"
                    :value="item.id"
                  >
                  </el-option>
                </el-select>
              </template>
            </el-table-column>
            <el-table-column
              width="220"
              prop="taskRemark"
              label="任务描述(可不填自动带入)"
            >
              <template slot-scope="scope">
                <el-input
                  v-model="scope.row.taskRemark"
                  placeholder="请输入任务描述"
                ></el-input>
              </template>
            </el-table-column>
          </el-table>
        </div>
      </el-drawer>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
export default {
  name: "#fmapp",
  data() {
    return {
      dialogVisible: false,
      fmaccount: "",
      fmpassword: "",
      showbtn: true,
      showbtn1: true,
      drawer: false,
      drawer1: false,
      direction: "ttb",
      tableData: [],
      resourceInfos: [],
      stsd: "",
      loading: false,
      endMon: "",
      startMon: "",
      currentPageTask: [],
      promData: [],
      selectData: [],
      statusList: [
        { id: 1, value: "计划中" },
        { id: 2, value: "执行中" },
        { id: 3, value: "已完成" },
        { id: 4, value: "取消" },
      ],
      partnersList: [],
      partnersId: "",
      percentage: 0,
      btloading: false,
      dateCells: [],
    };
  },
  created() {
    this.startMon = moment().add("month", 0).format("YYYY-MM") + "-01";
    let vEndM = moment(this.startMon).add("month", 1).add("days", -1);
    this.endMon = moment(vEndM).format("YYYY-MM-DD");

    if (!sessionStorage.fmWorklogToken) {
      this.dialogVisible = true;
      this.showbtn = false;
      this.showbtn1 = false;
    }
  },
  mounted() {},
  methods: {
    moment,
    getPartners() {
      let config = {
        url:
          "https://fm-worklog.sandload.cn/worklog/partners/v1.0/" +
          moment().valueOf(),
        data: {},
        method: "get",
      };
      this.fetchFun(config).then((res) => {
        this.partnersList = res.result;
        this.partnersId = "顶新_全家";
      });
    },
    createTask(data, index) {
      if (data.workTime === "" || data.workTime === null) {
        data.workTime = this.dateCells[0];
        this.dateCells.splice(0, 1);
      }
      if (data.taskRemark === "" || data.taskRemark === null) {
        data.taskRemark = data.taskname + "完成";
      }
      let statusdsc = this.statusList.filter((e) => {
        return e.id === data.status;
      })[0].value;
      let config = {
        url: "https://fm-worklog.sandload.cn/worklog/task/create",
        data: {
          projectCode: data.projectCode,
          taskName: data.taskname,
          toolUrl: data.url,
          startTime: moment(data.workTime).format("YYYY-MM-DD 09:30:00"),
          consumeTime: data.consumeTime,
          status: data.status,
          taskRemark: data.taskRemark,
          ver: "V1.0",
          timestamp: moment().valueOf(),
          partnerName: this.partnersId,
          statusDesc: statusdsc,
          source: "NORMAL",
        },
        method: "post",
      };
      this.fetchFun(config).then((res) => {
        if (res.code === "100") {
          if (index != undefined) {
            this.percentage = ((index + 1) / this.selectData.length) * 100;
            if (this.percentage === 100) {
              this.btloading = false;
              this.selectData = [];
              let that = this;
              setTimeout(() => {
                that.showDrawer1();
              }, 1000);
              this.$message.success({
                message: "批量填写工时完成！",
              });
            }
          }
        } else {
          this.btloading = false;
          this.selectData = [];
          this.$message.error({
            message: res.msg,
          });
        }
      });
    },
    batchok() {
      let _this = this;
      if (
        this.selectData.some((e) => {
          return e.projectCode === "";
        })
      ) {
        this.$message({
          message: "请先选择项目编号！",
        });
        return false;
      }
      if (!this.partnersId) {
        this.$message({
          message: "请先选择商户！",
        });
        return false;
      }
      if (this.selectData.length === 0) {
        this.$message({
          message: "请先勾选需要填写的需求任务！",
        });
        return false;
      }
      this.btloading = true;
      this.selectData.map((r, i) => {
        setTimeout(fn(r, i), i * 1000);
      });
      function fn(r, i) {
        return function () {
          _this.createTask(r, i);
        };
      }
    },
    getProjects() {
      let config = {
        url:
          "https://fm-worklog.sandload.cn/worklog/projects/v1.0/" +
          moment().valueOf(),
        data: {},
        method: "get",
      };
      this.fetchFun(config).then((res) => {
        this.promData = res.result;
      });
    },
    handleSelectionChange(e) {
      this.selectData = e;
    },
    handleOk() {
      this.dialogVisible = false;
      this.getToken();
    },
    drawerHandleClose1() {
      this.drawer1 = false;
    },
    drawerHandleClose() {
      this.drawer = false;
    },
    getToken() {
      let _this = this;
      let config = {
        url: "https://fm-worklog.sandload.cn/worklog/user/login",
        data: {
          password: this.fmpassword,
          timestamp: 1664160581993,
          userName: this.fmaccount,
          ver: "v1.0",
        },
        method: "POST",
      };
      this.fetchFun(config)
        .then((res) => {
          sessionStorage.setItem("fmUserName", res.result.name);
          sessionStorage.setItem("fmWorklogToken", res.result.token);
          _this.showbtn = true;
          _this.showbtn1 = true;
          _this.getFmUserWorklogData();
        })
        .catch((e) => {
          _this.$message({
            message: e.msg,
          });
        });
    },
    showDrawer1() {
      this.getProjects();
      this.getPartners();
      this.getFmUserWorklogData();
      this.partnersId = "";
      this.percentage = 0;
      this.currentPageTask = [];
      var _xquds =
        document.getElementsByTagName("table")[0].children[1].children;
      for (var i = 2; i < _xquds.length; i++) {
        let rowdata = {};
        for (var t = 0; t < _xquds[i].children.length; t++) {
          if (
            _xquds[i].children[t].getAttribute("data-editable-field") ===
            "owner"
          ) {
            let peoplename = _xquds[i].children[t].getAttribute(
              "data-editable-value"
            );
            if (peoplename.indexOf(sessionStorage.fmUserName) != -1) {
              rowdata["people"] = sessionStorage.fmUserName;
              rowdata["owner"] = peoplename;
            } else {
              rowdata["people"] = "";
              rowdata["owner"] = peoplename;
            }
          } else if (
            _xquds[i].children[t].getAttribute("data-editable-field") ===
            "developer"
          ) {
            let peoplename = _xquds[i].children[t].getAttribute(
              "data-editable-value"
            );
            rowdata["developer"] = _xquds[i].children[t].getAttribute(
              "data-editable-value"
            );
            if (peoplename.indexOf(sessionStorage.fmUserName) != -1) {
              rowdata["people"] = sessionStorage.fmUserName;
              rowdata["developer"] = peoplename;
            }
          } else if (
            _xquds[i].children[t].getAttribute("data-editable-field") ===
            "custom_field_100"
          ) {
            let peoplename = _xquds[i].children[t].getAttribute(
              "data-editable-value"
            );
            rowdata["testuser"] = _xquds[i].children[t].getAttribute(
              "data-editable-value"
            );
            if (peoplename.indexOf(sessionStorage.fmUserName) != -1) {
              rowdata["people"] = sessionStorage.fmUserName;
              rowdata["testuser"] = peoplename;
            }
          } else if (
            _xquds[i].children[t].getAttribute("data-editable-field") ===
            "custom_field_three"
          ) {
            rowdata["projectCode"] = _xquds[i].children[t]
              .getAttribute("data-editable-value")
              .split("_")[1];
          }
          if (
            _xquds[i].children[t].getAttribute("data-editable-field") === "name"
          ) {
            rowdata["taskname"] = _xquds[i].children[t].getAttribute(
              "data-editable-value"
            );
            rowdata["url"] =
              "https://www.tapd.cn/" +
              _xquds[i].children[t].children[0].children[0]
                .getElementsByClassName(
                  "namecol editable-value j-iteration-title-link"
                )[0]
                .getAttribute("href");
          }
        }
        if (rowdata.people === sessionStorage.fmUserName) {
          rowdata["workTime"] = "";
          rowdata["consumeTime"] = 8;
          rowdata["status"] = 3;
          rowdata["taskRemark"] = "";
          this.currentPageTask.push(rowdata);
        }
      }
      console.log(this.currentPageTask);
      this.drawer1 = true;
    },
    showDrawer() {
      this.drawer = true;
      this.getFmUserWorklogData();
    },
    getFmUserWorklogData() {
      let _this = this;
      this.loading = true;
      let config = {
        url: "https://fm-worklog.sandload.cn/worklog/report/queryResourceRangeNotEntered",
        data: {
          organizationNodes: [],
          endTime: _this.endMon,
          startTime: _this.startMon,
          timestamp: 1664162559285,
          ver: "V1.0",
        },
        method: "POST",
      };
      this.fetchFun(config).then((res) => {
        this.dateCells = res.result.dateCells;
        this.stsd =
          res.result.dateCells.length === 0
            ? []
            : res.result.dateCells.join("，");
        console.log(this.stsd);
        this.resourceInfos = res.result.resourceInfos;
        if (this.resourceInfos.length > 0) {
          this.resourceInfos[0].dateCells = res.result.dateCells;
        }
        this.tableData = this.resourceInfos;
        console.log(this.tableData);
        this.loading = false;
      });
    },
    fetchFun(options) {
      return new Promise((resolve, reject) => {
        const instance = axios.create({
          headers: {
            token: sessionStorage.fmWorklogToken
              ? sessionStorage.fmWorklogToken
              : null,
          },
        });
        instance(options)
          .then((response) => {
            console.log(response);
            if (response.data.code == "100" || response.status == 200) {
              resolve(response.data);
            } else if (response.data.code == "403") {
              sessionStorage.removeItem("fmWorklogToken");
              this.$message({
                message: "登录已过期，请重新刷新页面！",
              });
            } else {
              reject(response.data);
            }
          })
          .catch((error) => {
            reject(error);
          });
      });
    },
  },
};
</script>

<style>
</style>
