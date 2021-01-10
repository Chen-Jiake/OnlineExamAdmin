<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>考试管理</el-breadcrumb-item>
      <el-breadcrumb-item>试卷管理</el-breadcrumb-item>
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
        <el-button
          icon="el-icon-circle-plus-outline"
          type="primary"
          @click="addTestBefore"
          size="mini"
        >添加试卷</el-button>
      </el-row>
      <!-- 表格 -->
      <el-table :data="tableData" stripe border style="text-align: center">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="testName" label="试卷标题"></el-table-column>
        <el-table-column prop="testBeforetime" label="考试开放时间"></el-table-column>
        <el-table-column prop="testAftertime" label="考试关闭时间"></el-table-column>
        <el-table-column prop="testTimesum" label="考试时长（分钟）"></el-table-column>
        <el-table-column prop="testQuestionsum" label="试题总数"></el-table-column>
        <el-table-column prop="testScores" label="试卷总分"></el-table-column>
        <el-table-column prop="testPass" label="及格分数"></el-table-column>
        <el-table-column prop="testSelectOneSum" label="单选题数"></el-table-column>
        <el-table-column prop="testSelectMoreSum" label="多选题数"></el-table-column>
        <el-table-column prop="testSimpleSum" label="简单题数"></el-table-column>
        <el-table-column prop="testMiddleSum" label="中等题数"></el-table-column>
        <el-table-column prop="testDiffSum" label="高难题数"></el-table-column>
        <el-table-column label="试卷添加时间">
          <template v-slot:default="scope">{{scope.row.testAddtime | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="所属科目">
          <template v-slot:default="scope">{{scope.row.fkSubject.subjectName }}</template>
        </el-table-column>
        <el-table-column label="所属班级">
          <template v-slot:default="scope">{{scope.row.fkGrade.gradeName }}</template>
        </el-table-column>
        <el-table-column label="操作" width="130px;">
          <template v-slot:default="scope">
            <el-button
              @click="updateTest(scope.row)"
              icon="el-icon-edit"
              type="text"
              size="mini"
            >修改</el-button>
            <el-button
              icon="el-icon-delete"
              @click="deleteTest(scope.row.testId)"
              type="text"
              size="mini"
            >删除</el-button>
          </template>
        </el-table-column>
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

    <!-- 添加功能模态框 -->
    <el-dialog title="添加试题" :visible.sync="addTestDialogVisible" width="800px">
      <el-form
        label-position="right"
        label-width="150px"
        :model="addTestForm"
        :rules="testFormRule"
        size="mini"
        ref="addFormRef"
      >
        <el-row>
          <el-form-item label="试卷标题" prop="testName">
            <el-input v-model="addTestForm.testName"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="考试开放时间" prop="testBeforetime">
            <el-date-picker
              v-model="addTestForm.testBeforetime"
              type="datetime"
              placeholder="选择考试开放时间"
              default-time="00:00:00"
              value-format="yyyy-MM-dd HH:mm:ss">
            </el-date-picker>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="考试关闭时间" prop="testAftertime">
            <el-date-picker
              v-model="addTestForm.testAftertime"
              type="datetime"
              placeholder="选择考试关闭时间"
              default-time="00:00:00"
              value-format="yyyy-MM-dd HH:mm:ss">
            </el-date-picker>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="考试时长（分钟）" prop="testTimesum">
            <el-input v-model="addTestForm.testTimesum"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="单选题数" prop="testSelectOneSum">
            <el-input v-model="addTestForm.testSelectOneSum"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="多选题数" prop="testSelectMoreSum">
            <el-input v-model="addTestForm.testSelectMoreSum"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="难易程度" prop="questionLevel">
            <el-select v-model="this.testLevel" placeholder="- 请选择 -">
              <el-option
                v-for="item1 in difficultyLevel"
                :key="item1.level"
                :label="item1.difficulty"
                :value="item1.level"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="所属科目" prop="fkQuestionSubjectId">
            <el-select v-model="addTestForm.fkTestSubjectId" placeholder="- 请选择 -">
              <el-option
                v-for="item in subject"
                :key="item.subjectId"
                :label="item.subjectName"
                :value="item.subjectId"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="所属班级" prop="fkQuestionGradetId">
            <el-select v-model="addTestForm.fkTestGradeId" placeholder="- 请选择 -">
              <el-option
                v-for="item in grade"
                :key="item.gradeId"
                :label="item.gradeName"
                :value="item.gradeId"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
      </el-form>
      <!-- 确定取消按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="addTestYes" size="mini">确 定</el-button>
        <el-button @click="addTestDialogVisible = false" size="mini">取 消</el-button>
      </span>
    </el-dialog>

    <!-- 修改 -->
    <el-dialog title="修改试卷" :visible.sync="updateTestDialogVisible" width="800px">
      <el-form
        label-position="right"
        label-width="150px"
        :model="updateTestForm"
        :rules="testFormRule"
        size="mini"
        ref="updateFormRef"
      >
        <el-row>
          <el-form-item label="试卷标题" prop="testName">
            <el-input v-model="updateTestForm.testName"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="考试开放时间" prop="testBeforetime">
            <el-date-picker
              v-model="updateTestForm.testBeforetime"
              type="datetime"
              placeholder="选择考试开放时间"
              default-time="00:00:00"
              value-format="yyyy-MM-dd HH:mm:ss">
            </el-date-picker>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="考试关闭时间" prop="testAftertime">
            <el-date-picker
              v-model="updateTestForm.testAftertime"
              type="datetime"
              placeholder="选择考试关闭时间"
              default-time="00:00:00"
              value-format="yyyy-MM-dd HH:mm:ss">
            </el-date-picker>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="考试时长（分钟）" prop="testTimesum">
            <el-input v-model="updateTestForm.testTimesum"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="单选题数" prop="testSelectOneSum">
            <el-input v-model="updateTestForm.testSelectOneSum"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="多选题数" prop="testSelectMoreSum">
            <el-input v-model="updateTestForm.testSelectMoreSum"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="难易程度" prop="questionLevel">
            <el-select v-model="this.testLevel" placeholder="- 请选择 -">
              <el-option
                v-for="item1 in difficultyLevel"
                :key="item1.level"
                :label="item1.difficulty"
                :value="item1.level"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="所属科目" prop="fkQuestionSubjectId">
            <el-select v-model="updateTestForm.fkTestSubjectId" placeholder="- 请选择 -">
              <el-option
                v-for="item in subject"
                :key="item.subjectId"
                :label="item.subjectName"
                :value="item.subjectId"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="所属班级" prop="fkQuestionGradetId">
            <el-select v-model="updateTestForm.fkTestGradeId" placeholder="- 请选择 -">
              <el-option
                v-for="item in grade"
                :key="item.gradeId"
                :label="item.gradeName"
                :value="item.gradeId"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
      </el-form>
      <!-- 确定取消按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="updateTestYes()" size="mini">确 定</el-button>
        <el-button @click="updateTestDialogVisible = false" size="mini">取 消</el-button>
      </span>
    </el-dialog>
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
          { level: 0, difficulty: "简单（简单题50%，中等题40%，高难题10%）" },
          { level: 1, difficulty: "中等（简单题40%，中等题40%，高难题20%）" },
          { level: 2, difficulty: "高难（简单题30%，中等题40%，高难题30%）" }
        ],
        testLevel: "",
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
        //添加form
        addTestForm: {
          testName: "",
          testBeforetime: "",
          testAftertime: "",
          testTimesum: "",
          fkTestGradeId: "",
          fkTestSubjectId: "",
          testSelectOneSum: "",
          testSelectMoreSum: "",
          testDiffSum: ""
        },
        updateTestForm: {
          testName: "",
          testBeforetime: "",
          testAftertime: "",
          testTimesum: "",
          fkTestGradeId: "",
          fkTestSubjectId: "",
          testPass: "",
          testSelectOneSum: "",
          testSelectMoreSum: "",
          testSimpleSum: "",
          testMiddleSum: "",
          testDiffSum: ""
        },
        testLevelVo: {
          test: "",
          level: ""
        },
        //添加表单验证规则
        testFormRule: {
          testName: [
            { required: true, message: "请输入试卷标题", trigger: "blur" },
            { min: 1, max: 20, message: "长度在 1 到 20 个字符", trigger: "blur" }
          ],
          testBeforetime: [
            { required: true, message: "请选择考试开放时间", trigger: "blur" },
          ],
          testAftertime: [
            { required: true, message: "请选择考试关闭时间", trigger: "blur" },
          ],
          testTimesum: [
            { required: true, message: "请输入考试时长", trigger: "blur" },
            { min: 1, max: 11, message: "长度在 1 到 11 个字符", trigger: "blur" }
          ],
          testPass: [
            { required: true, message: "请输入考试通过分数", trigger: "blur" },
            { min: 1, max: 11, message: "长度在 1 到 11 个字符", trigger: "blur" }
          ],
          testSelectOneSum: [
            { required: true, message: "请输入单选题数", trigger: "blur" },
            { min: 1, max: 200, message: "长度在 1 到 11 个字符", trigger: "blur" }
          ],
          testSelectMoreSum: [
            { required: true, message: "请输入多选题数", trigger: "blur" },
            { min: 1, max: 200, message: "长度在 1 到 11 个字符", trigger: "blur" }
          ],
          testSimpleSum: [
            { required: true, message: "请输入简单题数", trigger: "blur" },
            { min: 1, max: 200, message: "长度在 1 到 11 个字符", trigger: "blur" }
          ],
          testMiddleSum: [
            { required: true, message: "请输入中等题数", trigger: "blur" },
            { min: 1, max: 200, message: "长度在 1 到 11 个字符", trigger: "blur" }
          ],
          testDiffSum: [
            { required: true, message: "请输入高难题数", trigger: "blur" },
            { min: 1, max: 200, message: "长度在 1 到 11 个字符", trigger: "blur" }
          ],
          fkTestSubjectId: [
            { required: true, message: "请选择试卷所属科目", trigger: "blur" },
          ],
          fkTestGradeId: [
            { required: true, message: "请选择试卷所属班级", trigger: "blur" }
          ]
        },
        //添加模态框是否显示
        addTestDialogVisible: false,
        //修改模态框是否显示
        updateTestDialogVisible: false,
        updateTextId: 0

      };
    },
    //钩子函数，已加载完成
    mounted() {
      //科目和班级列表
      this.getSubjectsAndGrades();
      //加载表格数据
      this.getTests();
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
      getUpdateSubjects() {
        this.$http
          .get("school/subject/findSubjects")
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.subject = res.data;
              // this.updateTestForm.fkTestSubjectId = this.subject[0].subjectId;
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
      getUpdateGrades() {
        this.$http
          .get("school/grade/findGrades")
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.grade = res.data;
              // this.updateTestForm.fkTestGradeId = this.grade[0].gradeId;
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
        this.getTests();
      },
      //清空搜索
      clear() {
        this.queryInfo.page = 1;
        this.queryInfo.questionType = "";
        this.queryInfo.fkTestGradeId = "";
        this.queryInfo.fkTestSubjectId = "";
        this.getTests();
      },
      //每页显示多少条改变
      handleSizeChange(newRows) {
        this.queryInfo.rows = newRows;
        this.getTests();
      },
      //当前页码改变
      handleCurrentChange(newPage) {
        this.queryInfo.page = newPage;
        this.getTests();
      },
      //分页获取表格数据
      getTests() {
        this.$http
          .get("test/test/findTestsByPage", {
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
              title: "pre:AuthorizationFilter" ? "抱歉，您没有权限查看后台用户列表！" : error.respon.data.message
            });
          });
      },
      //添加
      addTestYes() {
        this.testLevelVo.test = this.addTestForm
        this.testLevelVo.level = this.testLevel
        this.$refs.addFormRef.validate(valid => {
          if (valid) {
            //校验通过
            this.$http
              .post("test/test/addTest", this.testLevelVo)
              .then(response => {
                const res = response.data;
                if (res.httpCode === 201) {
                  this.addTestDialogVisible = false;
                  this.getTests();
                  this.$notify.success({
                    title: res.message
                  });
                }
              })
              .catch(error => {
                this.$notify.error({
                  title: error.response.data.message
                });
              });
          }
        });
      },
      updateTestYes(){
        this.testLevelVo.test = this.updateTestForm
        this.testLevelVo.level = this.testLevel
        this.$refs.updateFormRef.validate(valid => {
          if (valid) {
            //校验通过
            this.$http
              .post("test/test/addTest", this.testLevelVo)
              .then(response => {
                const res = response.data;
                if (res.httpCode === 201) {
                  this.$http
                    .delete("test/test/delTestById", {
                      params: {
                        testId: this.updateTextId
                      }
                    })
                  this.updateTestDialogVisible = false;
                  this.updateTextId = 0;
                  this.getTests();
                  this.$notify.success({
                    title: res.message
                  });
                }
              })
              .catch(error => {
                this.$notify.error({
                  title: "pre:AuthorizationFilter" ? "抱歉，您的权限暂未开放，请联系系统管理员！" : error.respon.data.message
                });
              });
          }
        });
      },
      //删除
      deleteTest(testId) {
        this.$confirm("此操作将永久删除该试卷, 并删除所有已参加该考试的记录,是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(() => {
            //删除
            this.$http
              .delete("test/test/delTestById", {
                params: {
                  testId: testId
                }
              })
              .then(response => {
                const res = response.data;
                if (res.httpCode === 204) {
                  this.$notify.success({
                    title: res.message
                  });
                  this.getTests();
                } else if (res.httpCode === 600) {
                  this.$notify.error({
                    title: res.message
                  });
                }
              })
              .catch(error => {
                console.log(error);
                this.$notify.error({
                  title: "pre:AuthorizationFilter" ? "抱歉，您的权限暂未开放，请联系系统管理员！" : error.respon.data.message
                });
              });
          })
          .catch(() => {});
      },
      //
      updateTest(test){
        this.$http
              .get("test/test/canTestUpdate", {
                params: {
                  testId: test.testId
                }
              })
              .then(response => {
                const res = response.data;
                if (res.httpCode === 204) {

                  this.updateTextId = test.testId;
                  this.updateTestBefore(test);
                  // this.$notify.success({
                  //   title: res.message
                  // });
                } else if (res.httpCode === 600) {
                  this.$notify.error({
                    title: res.message
                  });
                }
              })
              .catch(error => {
                console.log(error);
                this.$notify.error({
                  title: "pre:AuthorizationFilter" ? "抱歉，您的权限暂未开放，请联系系统管理员！" : error.respon.data.message
                });
              });




      },
      //修改
      updateUser(userId) {
        this.$http
          .get("user/user/findUserById", {
            params: {
              userId: userId
            }
          })
          .then(response => {
            const res = response.data;
            if (res.httpCode === 200) {
              this.getUpdateRoles();
              this.updateUserForm.userId = userId;
              this.updateUserForm.userAccount = res.data.userAccount;
              this.updateUserForm.userRealname = res.data.userRealname;
              this.updateUserForm.fkUserRoleId = res.data.fkUserRoleId;
              this.updateUserDialogVisible = true;
            }
          })
          .catch(error => {});
      },
      //确认修改
      updateUserYes() {
        this.$refs.updateFormRef.validate(valid => {
          if (valid) {
            //校验通过
            this.$http
              .put("user/user/updateUser", this.updateUserForm)
              .then(response => {
                const res = response.data;
                if (res.httpCode === 201) {
                  this.updateUserDialogVisible = false;
                  this.getTests();
                  this.$notify.success({
                    title: res.message
                  });
                } else if (res.httpCode === 600) {
                  this.$notify.error({
                    title: res.message
                  });
                }
              })
              .catch(error => {
                console.log(error);
                this.$notify.error({
                  title: "pre:AuthorizationFilter" ? "抱歉，您的权限暂未开放，请联系系统管理员！" : error.respon.data.message
                });
              });
          }
        });
      },
      //按下添加试题按钮
      addTestBefore() {
        (this.addTestForm.testTitle = ""),
          (this.addTestForm.testBeforetime = ""),
          (this.addTestForm.testAftertime = ""),
          (this.addTestForm.testTimesum = ""),
          (this.addTestForm.testPass = ""),
          (this.addTestForm.fkTestGradeId = ""),
          (this.addTestForm.fkTestSubjectId = ""),
          (this.addTestForm.testSelectOneSum = ""),
          (this.addTestForm.testSelectMoreSum = ""),
          (this.addTestForm.testSimpleSum = ""),
          (this.addTestForm.testMiddleSum = ""),
          (this.addTestForm.testDiffSum = ""),
          this.testLevel = 0,
          this.getAddSubjects(),
          this.getAddGrades(),
          (this.addTestDialogVisible = true)
      },
      updateTestBefore(test) {

          (this.updateTestForm.testName = test.testName),
          (this.updateTestForm.testBeforetime = test.testBeforetime),
          (this.updateTestForm.testAftertime = test.testAftertime),
          (this.updateTestForm.testTimesum = test.testTimesum+""),
          (this.updateTestForm.testPass = test.testPass),
          (this.updateTestForm.fkTestGradeId = test.fkTestGradeId),
          (this.updateTestForm.fkTestSubjectId = test.fkTestSubjectId),
          (this.updateTestForm.testSelectOneSum = test.testSelectOneSum+""),
          (this.updateTestForm.testSelectMoreSum = test.testSelectMoreSum+""),
          (this.updateTestForm.testSimpleSum = test.testSimpleSum),
          (this.updateTestForm.testMiddleSum = test.testMiddleSum),
          (this.updateTestForm.testDiffSum = test.testDiffSum),
          this.testLevel = 0,
          this.getUpdateSubjects(),
          (this.updateTestForm.fkTestSubjectId = test.fkTestSubjectId),
          this.getUpdateGrades(),
          (this.updateTestForm.fkTestGradeId = test.fkTestGradeId),
          (this.updateTestDialogVisible = true)


      }
    }
  };
</script>

<style lang='less' scoped>
  el-table {
    text-align: center;
  }
</style>
