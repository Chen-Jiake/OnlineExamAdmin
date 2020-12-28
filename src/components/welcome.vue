<template>
  <div>
    <el-card shadow="always" body-style="padding: 5px 20px;">
      <el-tag type="success" style="width: 100%;">主要功能</el-tag>
      <p>1. 系统管理：对后台用户、考试用户、角色权限进行管理</p>
      <p>2. 考试管理：对试题和试卷进行管理，对考生成绩进行统计</p>
      <p>3. 学校操作：对班级和科目进行管理</p>
      <p>4. 统计日志：查看系统重要操作日志</p>
    </el-card>
  </div>
</template>


<script>
export default {
  data() {
    return {};
  },
  mounted() {},
  methods: {}
};
</script>

<style lang='less' scoped>
.el-card {
  padding-top: 15px;
}

p {
  color: #555;
  font-size: 13px;
  text-indent: 28px;
}
</style>
