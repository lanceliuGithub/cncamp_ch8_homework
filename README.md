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

# 应用配置说明

手工编译代码后，应用的二进制会输出到 bin/linux/amd64 目录下
```
bin
└── linux
    └── amd64
        ├── config.json
        └── myhttpserver-1.0
```

同时在相同目录下会生成一份默认配置文件 config.json
```
{
	"server": {
		"host": "0.0.0.0",
		"port": "8888"
	},
	"log": {
		"enable": true,
		"request_header": false
	}
}
```

其中 server.host 是服务器监听的主机，server.port 是服务器监听的端口

log.enable 是记录后台日志的总开关，开启后日志会直接打印在控制台中，默认开启

log.request_header 是细化的日志开关（只有 log.enable 为 true 时才生效），此选项默认关闭

# 应用启动说明

查看启动选项
```
./myhttpserver-1.0 -h
Usage of ./myhttpserver-1.0:
  -c string
    	Specify an alternative config file (default "config.json")
```

目前只有一个选项 -c ，用于指定不同的配置文件供服务器加载
```
./myhttpserver-1.0 -c /etc/another_config.json
```

本HTTP服务器启动后，会模拟两个阶段的耗时
1. 启动耗时，共5s
2. 服务就绪耗时，共10s

启动耗时是从应用启动后，到端口被侦听这段时间，耗时5s

服务器就绪耗时是等启动耗时过去后，再等5s，/healthz接口才能返回成功，否则返回500状态码和failed包体

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

# 使用K8S优雅管理一个Pod

配置文件位于 k8s-plan/graceful-pod.yaml

运行如下命令
```
kubectl apply -f k8s-plan/graceful-pod.yaml
```

观察Pod的状态变化
```
kubectl get pod myhttpserver -w
```

查看HTTP服务器后台日志
```
kubectl logs -f myhttpserver
```


在宿主机上访问HTTP服务

- 首页: http://localhost
- 健康检查页: http://localhost/healthz
- 缺失的页面: http://localhost/no_such_page

移除应用
```
kubectl delete -f k8s-plan/graceful-pod.yaml
```

# 使用K8S维护一个安全且高可用的服务

配置文件位于 k8s-plan/secure-ha-service 目录下

部署所有对象
```
kubectl apply \
  -f 1.config.yaml \
  -f 2.deploy.yaml \
  -f 3.service.yaml \
  -f 4.ingress-nginx-deploy.yaml \
  -f 5.ingress-cert.yaml \
  -f 6.ingress.yaml
```

卸载所有对象
```
kubectl delete \
  -f 6.ingress.yaml \
  -f 5.ingress-cert.yaml \
  -f 4.ingress-nginx-deploy.yaml \
  -f 3.service.yaml \
  -f 2.deploy.yaml \
  -f 1.config.yaml
```

注意：卸载时，如果报如下错误，可以稍等一会再试
```
Error from server (InternalError): error when creating "6.ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": dial tcp 10.105.108.221:443: connect: connection refused
```

发起HTTP访问
```
GATEWAY=`kubectl get -n ingress-nginx svc ingress-nginx-controller -ojson | jq -r '.spec.clusterIP'`
curl -k -H "Host: lancelot.cn" https://$GATEWAY/healthz
```

对象yaml说明：
- 1.config.yaml   HTTP服务器的配置文件对象（ConfigMap）
- 2.deploy.yaml   HTTP服务器的部署对象（Deployment）
- 3.service.yaml  HTTP服务器的服务对象（Service）
- 4.ingress-nginx-deploy.yaml   Nginx实现的Ingress控制器
- 5.ingress-cert.yaml   HTTP服务器的TLS证书（Secret TLS）
- 6.ingress.yaml  HTTP服务器的网关对象（Ingress）

重新生成证书使用命令并同时修改 5.ingress-cert.yaml：
```
cd k8s-plan/secure-ha-service

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-subj "/CN=lancelot.cn/O=lancelot" \
-addext "subjectAltName = DNS:lancelot.cn" \
-keyout lancelot_cn.key -out lancelot_cn.crt
```
