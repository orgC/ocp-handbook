# oc new-app 

通过指定源码，模板或者镜像来创建应用  
该命令会尝试利用 镜像， 模板，代码来创建应用以及相关组件，它会从本地，docker registry，image stream以及template 中寻找镜像  

... 

如果你提供了一个source code 地址，那么会自动创建一个新的build，你可以使用`oc status`去查看这个过程


```
Usage:  
  oc new-app (IMAGE | IMAGESTREAM | TEMPLATE | PATH | URL ...) [flags]
```

# Example
列出所有可以使用的template和image stream  
`oc new-app --list`


基于当前git repository 创建一个应用  
` oc new-app . --docker-image=repo/langimage`

按照指定的 镜像:源码 组合形式创建ruby应用  
`oc new-app centos/ruby-22-centos7~https://github.com/openshift/ruby-ex.git`
