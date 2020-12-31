# 通用yaml模板
> 此开源图书由[thaiq](https://github.com/ithaiq)原创，创作不易转载请注明出处

*   创建namespace

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: xxx
```

*  创建ConfigMap

``` yaml
apiVersion: v1
data:
  config: |- #key可以定义多个
      #具体config
kind: ConfigMap
metadata:
  name: xxx
```

* 创建PV(使用nfs)

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: xxx
spec:
  capacity:
    storage: 5Gi #磁盘容量
  accessModes:
    - ReadWriteMany #多主机读写
  nfs:
    server: 127.0.0.1 #nfs内网地址
    path: "xxx" #nfs共享path
```

* 创建PVC

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: xxx
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 5Gi
```

* 创建deployment(包含命令行参数、环境变量、configmap挂载数据卷、PVC挂载)

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: xxx
spec:
  selector:
    matchLabels:
      app: xxx
  replicas: 1
  template:
    metadata:
      labels:
        app: xxx
    spec:
      containers:
        - args:
            - sh
            - -c
            - "sleep 2000" #调试
          env:
            - name: xxx
              value: xxx
          image: xxx
          imagePullPolicy: Always #IfNotPresent
          name: xxx
          ports:
            - containerPort: 80 #容器应用端口
              protocol: TCP
          volumeMounts:
            - mountPath: /app/config #挂载容器目录
              name: cfg
            - mountPath: /xxx/xxx #容器完整目录
              name: pvc
              subPath: xxx/xxx #以pvc为根目录的路径
          workingDir: /app #容器工作目录
      imagePullSecrets:
        - name: xxx #拉取私有镜像
      volumes:
        - configMap:
            defaultMode: 0655 #权限
            items:
              - key: config #configmap的key
                path: config.yaml #映射容器里的文件名
            name: xxx #configmap的名字
          name: cfg
        - name: pvc
          persistentVolumeClaim:
            claimName: pvcname
```

* 创建service

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: xxx
spec:
  selector:
    app: xxx #deployment定义的标签
  ports:
    - protocol: TCP
      port: 80 #容器端口
      targetPort: 80 #目标端口
  type: ClusterIP #还有nodeport hostport loadbalance
#clusterip模式可以让外部通过servicename.namespace:targetPort访问
```

* 创建ingress

``` yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: xxx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: "/$2" #路径重写
spec:
  rules:
    - host: www.xxx.xxx
      http:
        paths:
          - path: /api/(\/?)(.*) #路径
            backend:
              serviceName: xxx
              servicePort: 80 #service targetport
```
