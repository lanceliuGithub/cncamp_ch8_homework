# 项目介绍

本项目用于创建一个实验性质的HTTP服务器，仅可用于学习
https://github.com/lanceliuGithub/cncamp_ch8_homework.git

# 编译二进制可执行文件

建议在 Linux 环境运行如下编译命令，Windows平台请先安装 Cygwin
```
make
```
或
```
make build
```

# 制作容器镜像

生成容器镜像
```
make release
```
请注意release动作包括了make，只不过编译动作是在容器中完成的。
如果只想单独编译出二进制，请使用 make build

生成容器镜像并推送到 Docker Hub 公开仓库
```
make push
```

如果推送时报错 `denied: requested access to the resource is denied` ，请先登录 docker.com
```
docker login
```

# 使用Docker启动应用

运行如下命令
```
docker run -d --name myhttpserver -e VERSION=1.0 -p 80:8888 lanceliu2022/myhttpserver:1.0
```
其中的 VERSION 环境变量可以省略，默认是1.0

打开方式（localhost可以更换为任意服务端IP）

- 首页: http://localhost
- 健康检查页: http://localhost/healthz
- 缺失的页面: http://localhost/no_such_page

