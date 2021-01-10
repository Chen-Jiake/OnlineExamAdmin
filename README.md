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

## 如何将后台管理界面部署到阿里云
1. 执行npm run build打包
2. 把生成的dist文件夹和Dockerfile文件上传到服务器
3. 执行docker build -t exam-admin . 生成镜像
4. 执行docker run -d -p 8003:80 --name exam-admin exam-admin 运行容器
