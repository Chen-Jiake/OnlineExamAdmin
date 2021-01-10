<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>考试管理</el-breadcrumb-item>
      <el-breadcrumb-item>考试统计</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card body-style="padding: 5px 10px;">
      <!-- 搜索表单 -->
      <el-form label-position="right" label-width="80px" :model="queryInfo" size="mini">
        <el-row :gutter="10">
          <el-col :xs="12" :sm="8" :md="6" :lg="6" :xl="4">
            <el-form-item label="考试班级">
              <el-select v-model="queryInfo.fkTestGradeId" placeholder="- 请选择 -">
                <el-option
                  v-for="item in grade"
                  :key="item.gradeId"
                  :label="item.gradeName"
                  :value="item.gradeId"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :xs="12" :sm="8" :md="6" :lg="6" :xl="4">
            <el-form-item label="考试科目">
              <el-select v-model="queryInfo.fkTestSubjectId" placeholder="- 请选择 -">
                <el-option
                  v-for="item in subject"
                  :key="item.subjectId"
                  :label="item.subjectName"
                  :value="item.subjectId"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <!-- 按钮 -->
      <el-row :gutter="10" class="importexport" style="margin-left: 0px;">
        <el-button icon="el-icon-search" @click="searchTests" type="success" size="mini">搜索</el-button>
        <el-button icon="el-icon-refresh" @click="clear" size="mini">清空搜索</el-button>
      </el-row>
      <!-- 表格 -->
      <el-table :data="tableData" stripe border style="text-align: center">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="姓名">
          <template v-slot:default="scope">{{scope.row.fkUserQ.userQRealname }}</template>
        </el-table-column>
        <el-table-column label="班级">
          <template v-slot:default="scope">{{scope.row.fkTest.fkGrade.gradeName }}</template>
        </el-table-column>
        <el-table-column label="试卷标题">
          <template v-slot:default="scope">{{scope.row.fkTest.testName }}</template>
        </el-table-column>
        <el-table-column label="试卷总分值">
          <template v-slot:default="scope">{{scope.row.fkTest.testScores }}</template>
        </el-table-column>
        <el-table-column label="试卷通过分数">
          <template v-slot:default="scope">{{scope.row.fkTest.testPass }}</template>
        </el-table-column>
        <el-table-column label="科目">
          <template v-slot:default="scope">{{scope.row.fkTest.fkSubject.subjectName }}</template>
        </el-table-column>
        <el-table-column prop="testUserQStarttime" label="用户开始考试时间"></el-table-column>
        <el-table-column prop="testUserQCommittime" label="用户结束考试时间"></el-table-column>
        <el-table-column prop="testUserQScore" label="用户考试成绩"></el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.page"
        :page-sizes="[5, 10, 20, 50, 100]"
        :page-size="queryInfo.rows"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        //试题类型
        questionType: [
          { type: 0, question: "单选" },
          { type: 1, question: "多选" },
        ],
        //难易程度
        difficultyLevel: [
          { level: 0, difficulty: "简单" },
          { level: 1, difficulty: "中等" },
          { level: 2, difficulty: "高难" }
        ],
        //表格数据
        tableData: [],
        grade: [],
        subject: [],
        //总条数
        total: 0,
        //分页查询参数
        queryInfo: {
          page: 1, //当前页数
          rows: 5, //每页显示条数
          fkTestGradeId: "",
          fkTestSubjectId: ""
        },
      };
    },
    //钩子函数，已加载完成
    mounted() {
      //科目和班级列表
      this.getSubjectsAndGrades();
      //加载表格数据
      this.getResults();
    },
    methods: {
      //科目和班级列表
      getSubjectsAndGrades() {
        this.$http
          .get("school/subject/findSubjects")
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.subject = res.data;
            }
          })
          .catch(error => {
            console.log(error);
            this.$notify.error({
              title: error.response.data.message
            });
          });
        this.$http
          .get("school/grade/findGrades")
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.grade = res.data;
            }
          })
          .catch(error => {
            this.$notify.error({
              title: error.response.data.message
            });
          });
      },
      //为添加表单添加科目列表
      getAddSubjects() {
        this.$http
          .get("school/subject/findSubjects")
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.subject = res.data;
              this.addTestForm.fkTestSubjectId = this.subject[0].subjectId;
            }
          })
          .catch(error => {
            console.log(error);
            this.$notify.error({
              title: error.response.data.message
            });
          });
      },
      getAddGrades() {
        this.$http
          .get("school/grade/findGrades")
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.grade = res.data;
              this.addTestForm.fkTestGradeId = this.grade[0].gradeId;
            }
          })
          .catch(error => {
            this.$notify.error({
              title: error.response.data.message
            });
          });
      },
      //为修改表单添加角色列表
      getUpdateRoles() {
        this.$http
          .get("user/role/findRoles")
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.updateUserForm.role = res.data;
            }
          })
          .catch(error => {
            console.log(error);
            this.$notify.error({
              title: error.response.data.message
            });
          });
      },
      //搜索
      searchTests() {
        this.queryInfo.page = 1;
        this.getResults();
      },
      //清空搜索
      clear() {
        this.queryInfo.page = 1;
        this.queryInfo.fkTestSubjectId = "";
        this.queryInfo.fkTestGradeId = "";
        this.getResults();
      },
      //每页显示多少条改变
      handleSizeChange(newRows) {
        this.queryInfo.rows = newRows;
        this.getResults();
      },
      //当前页码改变
      handleCurrentChange(newPage) {
        this.queryInfo.page = newPage;
        this.getResults();
      },
      //分页获取表格数据
      getResults() {
        this.$http
          .get("test/testUserQ/findTestUserQsByPage", {
            params: {
              page: this.queryInfo.page,
              rows: this.queryInfo.rows,
              questionLevel: this.queryInfo.fkTestGradeId,
              fkQuestionSubjectId: this.queryInfo.fkTestSubjectId
            }
          })
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.tableData = res.data.list;
              this.total = res.data.total;
            }
          })
          .catch(error => {
            this.$notify.error({
              title: "pre:AuthorizationFilter" ? "抱歉，您咱没有权限查看后台用户列表！" : error.respon.data.message
            });
          });
      },
    }
  };
</script>

<style lang='less' scoped>
  el-table {
    text-align: center;
  }
</style>
