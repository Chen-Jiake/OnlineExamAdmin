<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/admin' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>考试管理</el-breadcrumb-item>
      <el-breadcrumb-item>试题管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card body-style="padding: 5px 10px;">
      <!-- 搜索表单 -->
      <el-form label-position="right" label-width="80px" :model="queryInfo" size="mini">
        <el-row :gutter="10">
          <el-col :xs="12" :sm="8" :md="6" :lg="6" :xl="4">
            <el-form-item label="试题类型">
              <el-select v-model="queryInfo.questionType" placeholder="- 请选择 -">
                <el-option
                  v-for="item in questionType"
                  :key="item.type"
                  :label="item.question"
                  :value="item.type"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :xs="12" :sm="8" :md="6" :lg="6" :xl="4">
            <el-form-item label="难易程度">
              <el-select v-model="queryInfo.questionLevel" placeholder="- 请选择 -">
                <el-option
                  v-for="item in difficultyLevel"
                  :key="item.level"
                  :label="item.difficulty"
                  :value="item.level"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :xs="12" :sm="8" :md="6" :lg="6" :xl="4">
            <el-form-item label="所属科目">
              <el-select v-model="queryInfo.fkQuestionSubjectId" placeholder="- 请选择 -">
                <el-option
                  v-for="item in queryInfo.subject"
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
        <el-button icon="el-icon-search" @click="searchQuestion" type="success" size="mini">搜索</el-button>
        <el-button icon="el-icon-refresh" @click="clear" size="mini">清空搜索</el-button>
        <el-button
          icon="el-icon-circle-plus-outline"
          type="primary"
          @click="addQuestionBefore"
          size="mini"
        >添加试题</el-button>
      </el-row>
      <!-- 表格 -->
      <el-table :data="tableData" stripe border style="text-align: center">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="questionTitle" label="试题题目"></el-table-column>
        <el-table-column prop="questionType" label="试题类型"></el-table-column>
        <el-table-column prop="questionSelectA" label="选项A描述"></el-table-column>
        <el-table-column prop="questionSelectB" label="选项B描述"></el-table-column>
        <el-table-column prop="questionSelectC" label="选项C描述"></el-table-column>
        <el-table-column prop="questionSelectD" label="选项D描述"></el-table-column>
        <el-table-column prop="questionYesanswer" label="正确答案"></el-table-column>
        <el-table-column prop="questionScore" label="分值"></el-table-column>
        <el-table-column prop="questionLevel" label="难易程度"></el-table-column>
        <el-table-column label="注册时间">
          <template v-slot:default="scope">{{scope.row.questionAddtime | dateFormat}}</template>
        </el-table-column>
        <el-table-column label="所属角色">
          <template v-slot:default="scope">{{scope.row.fkSubject.subjectName }}</template>
        </el-table-column>
        <el-table-column label="操作" width="130px;">
          <template v-slot:default="scope">
            <el-button
              @click="updateQuestion(scope.row.questionId)"
              icon="el-icon-edit"
              type="text"
              size="mini"
            >修改</el-button>
            <el-button
              icon="el-icon-delete"
              @click="deleteQuestion(scope.row.questionId)"
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
    <el-dialog title="添加试题" :visible.sync="addQuestionDialogVisible" width="800px">
      <el-form
        label-position="right"
        label-width="150px"
        :model="addQuestionForm"
        :rules="questionFormRule"
        size="mini"
        ref="addFormRef"
      >
        <el-row>
          <el-form-item label="试题题目" prop="questionTitle">
            <el-input v-model="addQuestionForm.questionTitle"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="试题类型" prop="userPassword">
            <el-select v-model="addQuestionForm.questionType" placeholder="- 请选择 -">
              <el-option
                v-for="item in questionType"
                :key="item.type"
                :label="item.question"
                :value="item.type"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="选项A描述" prop="questionSelectA">
            <el-input v-model="addQuestionForm.questionSelectA"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="选项B描述" prop="questionSelectA">
            <el-input v-model="addQuestionForm.questionSelectB"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="选项C描述" prop="questionSelectC">
            <el-input v-model="addQuestionForm.questionSelectC"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="选项D描述" prop="questionSelectD">
            <el-input v-model="addQuestionForm.questionSelectD"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="正确答案" prop="questionYesanswer">
            <el-input v-model="addQuestionForm.questionYesanswer"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="分值" prop="questionScore">
            <el-input v-model="addQuestionForm.questionScore"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="难易程度" prop="questionLevel">
            <el-select v-model="addQuestionForm.questionLevel" placeholder="- 请选择 -">
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
          <el-form-item label="试题所属科目" prop="fkQuestionSubjectId">
            <el-select v-model="addQuestionForm.fkQuestionSubjectId" placeholder="- 请选择 -">
              <el-option
                v-for="item in addQuestionForm.subject"
                :key="item.subjectId"
                :label="item.subjectName"
                :value="item.subjectId"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
      </el-form>
      <!-- 确定取消按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="addQuestionYes" size="mini">确 定</el-button>
        <el-button @click="addQuestionDialogVisible = false" size="mini">取 消</el-button>
      </span>
    </el-dialog>
    <!-- 修改功能模态框 -->
    <el-dialog title="修改用户" :visible.sync="updateUserDialogVisible" width="300px">
      <el-form
        label-position="right"
        label-width="80px"
        :model="updateUserForm"
        :rules="questionFormRule"
        size="mini"
        ref="updateFormRef"
      >
        <el-row>
          <el-form-item label="登录账户" prop="userAccount">
            <el-input disabled v-model="updateUserForm.userAccount"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="真实姓名" prop="userRealname">
            <el-input disabled v-model="updateUserForm.userRealname"></el-input>
          </el-form-item>
        </el-row>
        <el-row>
          <el-form-item label="所属角色" prop="fkUserRoleId">
            <el-select v-model="updateUserForm.fkUserRoleId" placeholder="- 请选择 -">
              <el-option
                v-for="item in updateUserForm.role"
                :key="item.roleId"
                :label="item.roleName"
                :value="item.roleId"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-row>
      </el-form>
      <!-- 确定取消按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="updateUserYes" size="mini">确 定</el-button>
        <el-button @click="updateUserDialogVisible = false" size="mini">取 消</el-button>
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
        { level: 0, difficulty: "简单" },
        { level: 1, difficulty: "中等" },
        { level: 2, difficulty: "高难" }
      ],
      //表格数据
      tableData: [],
      //总条数
      total: 0,
      //分页查询参数
      queryInfo: {
        page: 1, //当前页数
        rows: 5, //每页显示条数
        questionType: "",
        questionLevel: "",
        subject: [],
        fkQuestionSubjectId: "",
      },
      //添加form
      addQuestionForm: {
        questionTitle: "",
        questionType: "",
        questionSelectA: "",
        questionSelectB: "",
        questionSelectC: "",
        questionSelectD: "",
        questionYesanswer: "",
        questionScore: "",
        questionLevel: "",
        subject: [],
        fkQuestionSubjectId: ""
      },
      //修改form
      updateUserForm: {
        questionId: "",
        questionTitle: "",
        questionType: "",
        questionSelectA: "",
        questionSelectB: "",
        questionSelectC: "",
        questionSelectD: "",
        questionYesanswer: "",
        questionScore: "",
        questionLevel: "",
        subject: [],
        fkQuestionSubjectId: ""
      },
      //添加表单验证规则
      questionFormRule: {
        questionTitle: [
          { required: true, message: "请输入试题题目", trigger: "blur" },
          { min: 5, max: 500, message: "长度在 5 到 500 个字符", trigger: "blur" }
        ],
        questionType: [
          { required: true, message: "请选择试题类型", trigger: "blur" },
        ],
        questionSelectA: [
          { required: true, message: "请输入选择题选项A描述", trigger: "blur" },
          { min: 1, max: 200, message: "长度在 1 到 200 个字符", trigger: "blur" }
        ],
        questionSelectB: [
          { required: true, message: "请输入选择题选项B描述", trigger: "blur" },
          { min: 1, max: 200, message: "长度在 1 到 200 个字符", trigger: "blur" }
        ],
        questionSelectC: [
          { required: true, message: "请输入选择题选项C描述", trigger: "blur" },
          { min: 1, max: 200, message: "长度在 1 到 200 个字符", trigger: "blur" }
        ],
        questionSelectD: [
          { required: true, message: "请输入选择题选项D描述", trigger: "blur" },
          { min: 1, max: 200, message: "长度在 1 到 200 个字符", trigger: "blur" }
        ],
        questionYesanswer: [
          { required: true, message: "请输入正确答案", trigger: "blur" },
          { min: 1, max: 1000, message: "长度在 1 到 1000 个字符", trigger: "blur" }
        ],
        questionScore: [
          { required: true, message: "请输入分值", trigger: "blur" },
          { min: 1, max: 11, message: "长度在 1 到 11 个字符", trigger: "blur" }
        ],
        questionLevel: [
          { required: true, message: "请选择难易程度", trigger: "blur" },
        ],
        fkQuestionSubjectId: [
          { required: true, message: "请选择试题所属科目", trigger: "blur" }
        ]
      },
      //添加模态框是否显示
      addQuestionDialogVisible: false,
      //修改模态框是否显示
      updateUserDialogVisible: false
    };
  },
  //钩子函数，已加载完成
  mounted() {
    //为搜索表单添加课程列表
    this.getSearchSubjects();
    //加载表格数据
    this.getQuestions();
  },
  methods: {
    //为搜索表单添加角色列表
    getSearchSubjects() {
      this.$http
        .get("school/subject/findSubjects")
        .then(response => {
          const res = response.data;
          if (res.httpCode === 200) {
            this.queryInfo.subject = res.data;
          }
        })
        .catch(error => {
          console.log(error);
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
            this.addQuestionForm.subject = res.data;
            this.addQuestionForm.fkQuestionSubjectId = this.addQuestionForm.subject[0].subjectId;
          }
        })
        .catch(error => {
          console.log(error);
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
    searchQuestion() {
      this.queryInfo.page = 1;
      this.getQuestions();
    },
    //清空搜索
    clear() {
      this.queryInfo.page = 1;
      this.queryInfo.questionType = "";
      this.queryInfo.questionLevel = "";
      this.queryInfo.fkQuestionSubjectId = "";
      this.getQuestions();
    },
    //每页显示多少条改变
    handleSizeChange(newRows) {
      this.queryInfo.rows = newRows;
      this.getQuestions();
    },
    //当前页码改变
    handleCurrentChange(newPage) {
      this.queryInfo.page = newPage;
      this.getQuestions();
    },
    //分页获取表格数据
    getQuestions() {
      this.$http
        .get("test/question/findQuestionsByPage", {
          params: {
            page: this.queryInfo.page,
            rows: this.queryInfo.rows,
            questionType: this.queryInfo.questionType,
            questionLevel: this.queryInfo.questionLevel,
            fkQuestionSubjectId: this.queryInfo.fkQuestionSubjectId
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
    //添加
    addQuestionYes() {
      this.$refs.addFormRef.validate(valid => {
        if (valid) {
          //校验通过
          this.$http
            .post("test/question/addQuestion", this.addQuestionForm)
            .then(response => {
              const res = response.data;
              if (res.httpCode === 201) {
                this.addQuestionDialogVisible = false;
                this.getQuestions();
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
    deleteUser(userId) {
      this.$confirm("此操作将永久删除该用户, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          //删除
          this.$http
            .delete("user/user/delUserById", {
              params: {
                userId: userId
              }
            })
            .then(response => {
              const res = response.data;
              if (res.httpCode === 204) {
                this.$notify.success({
                  title: res.message
                });
                this.queryInfo.page = 1;
                this.getQuestions();
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
                this.getQuestions();
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
    addQuestionBefore() {
        (this.addQuestionForm.questionTitle = ""),
        (this.addQuestionForm.questionType = 0),
        (this.addQuestionForm.questionSelectA = ""),
        (this.addQuestionForm.questionSelectB = ""),
        (this.addQuestionForm.questionSelectC = ""),
        (this.addQuestionForm.questionSelectD = ""),
        (this.addQuestionForm.questionYesanswer = ""),
        (this.addQuestionForm.questionScore = ""),
        (this.addQuestionForm.questionLevel = 0),
        this.getAddSubjects(),
          (this.addQuestionDialogVisible = true)
    }
  }
};
</script>

<style lang='less' scoped>
el-table {
  text-align: center;
}
</style>
