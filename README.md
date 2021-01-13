# OnlineExamAdmin
在线考试系统——后台管理界面

## 简介
本项目使用vue和springcloud开发，并使用docker部署到阿里云

项目还包括了
1. [在线考试系统——学生考试界面](https://github.com/Chen-Jiake/OnlineExamStudent)
2. [在线考试系统——后端](https://github.com/Chen-Jiake/OnlineExamServer)

本项目在以下两个项目的基础上完成
1. https://github.com/hongfurui2014/exam_online_hongfurui
2. https://gitee.com/davz/yf-exam-lite

## 项目主要功能
### 学生考试
1. 考试中心：参加考试
2. 历史考试：查看考试成绩以及答题情况
### 后台管理
1. 系统管理：对后台用户、考试用户、角色权限进行管理
2. 考试管理：对试题和试卷进行管理，对考生成绩进行统计
3. 学校操作：对班级和科目进行管理
4. 统计日志：查看系统重要操作日志

## 在本机配置后台管理界面——以ubuntu18.04为例
1. 配置环境：nodejs、np
2. 执行npm install
3. 执行npm run dev 

## 利用 docker 将后台管理界面部署到阿里云
1. 执行npm run build打包
2. 把生成的dist文件夹和Dockerfile文件上传到服务器
3. 执行docker build -t exam-admin . 生成镜像
4. 执行docker run -d -p 8003:80 --name exam-admin exam-admin 运行容器
