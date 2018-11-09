liveness and readiness

kubelet 通过liveness知道什么时候需要重启pod

kubelet 通过readiness 知道什么时候容器业务处于ready 状态，可以对外提供服务


## 部署 command 探测器
执行livenessProbe中的命令，如果返回值为0，成功，非0，失败

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

## HTTP probe
使用HTTP get 请求， 200 ~ 400 之间的值成功，其他值失败  

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```


## TCP liveness probe
kubelet 会尝试创建连接到指定的端口，如果能够创建连接，就说明服务正常

```
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

`periodSeconds` 探测周期，在示例中每隔5秒钟探测一次  
`initialDelaySeconds` 初始化时间等待时间
`timeoutSeconds`   
`successThreshold `  
`failureThreshold` 当pod第一次probe失败后，重试failureThreshold 次，如果还是失败，那就放弃probe，然后重启pod


## HTTP probe 参数
`host` 连接的主机名字，默认是pod ip
`scheme` 连接到host的方法（http， https）， 默认是http  
`path` http 请求地址  
`httpHeaders` 用户定义http header  
`port` 端口号 1 ~ 65535   
