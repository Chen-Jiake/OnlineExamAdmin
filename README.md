# OnlineExamAdmin
在线考试系统——后台管理界面

## 将项目部署到阿里云
1. 执行npm run build打包
2. 把生成的dist文件夹和Dockerfile文件上传到服务器
3. 执行docker build -t exam-admin . 生成镜像
4. 执行docker run -d -p 8003:80 --name exam-admin exam-admin 运行容器
