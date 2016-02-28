# 常见问题

1. 为什么 meteor 命令找不到？

   		需要在meteor项目目录内运行，运行meteor命令时需要读取 .meteor/packages 文件才能启动。

   		*注意命令的输出*

2. 如何连接 MongoDB？

   		首先要启动Meteor命令（会自动启动MongoDB），其次注意MongoDB的端口通常是Meteor运行的端口+1；
   		推荐办法：meteor mongo （连接的时候会显示端口)

3. 有没有什么工具能生成localmarket 一样的目录结构初始项目？

		meteor create —example localmarket
		cd localmarket
		find . -type f -exec rm -rf {} \;
		tree .
