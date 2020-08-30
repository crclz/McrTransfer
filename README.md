# 在中国大陆安全访问MCR（微软容器镜像源）

## 基础信息

MCR github主页： https://github.com/microsoft/containerregistry

中国大陆访问mcr非常慢、不稳定。并且微软下架了dockerhub的dotnet导致无法使用加速站点。

安全：看怎么理解“安全”了。我认为，个人（除自己外）的镜像都是不太放心的，所以得用自己的。（当然，感谢那些为大家搭建镜像的个人，我没有贬低你们的意思）


## 思路
mcr ---> 阿里云容器镜像服务 ---> 你的电脑


## 找到自己想要的镜像
如果熟悉这些镜像名称的，可以到 [DockerHub .Net 索引页](https://hub.docker.com/_/microsoft-dotnet-core) 上面去找。然后在相关页面的`Tags`里面找想要的tag，就行了。

对于不熟悉这些镜像的，可以用 Visual Studio 添加docker支持（记得把除了Dockerfile、dockerignore之外的生成的东西给删除）。得到Dockerfile后，你就知道你想要哪些镜像了。

## 建立github仓库，添加 Dockerfile
建立github仓库，里面用于存放dockerhub + 作为构建的代码仓库

Dockerfile参考本仓库的。例如 [aspnet3.1-alpine.Dockerfile](aspnet3.1-alpine.Dockerfile)

## 开通阿里云容器镜像服务 + 设置
阿里云的是免费的。不知道腾讯云有没有。

开通之后，到 https://cr.console.aliyun.com/

让我们来看一个地址：

`registry.cn-hangzhou.aliyuncs.com/hello/webapp:6.0`

解析： `地区.aliyuncs.com/命名空间/仓库名称:TAG`

注意，容器镜像服务分成不同的区的。不同区的命名空间没有关联。

所以，首先选择一个地区（左上角），然后创建命名空间，然后创建仓库。

然后进入仓库。按照“操作指南”可以进行登录，登陆了才有push、pull (private仓库)的权限。

## 构建
点击左侧的“构建”，然后点击右侧的“添加规则”。

这个就是本仓库的某个dockerfile的设置： 

| 类型   | branch/tag | Dockerfile目录 | Dockerfile文件名                 | 镜像版本              |
|--------|------------|----------------|----------------------------------|-----------------------|
| branch | master     | /              | aspnet3.1-buster-slim.Dockerfile | aspnet3.1-buster-slim |

这个对应的地址就是：`地区.aliyuncs.com/命名空间/仓库名称:aspnet3.1-buster-slim`

然后，左上角的`代码变更自动构建镜像`关闭。

`海外机器构建关闭`我是关闭的，构建挺快的（应该是有缓存的原因）。如果你构建过慢，就开着。

然后再点击`构建规则设置`项目的屏幕最右方的`构建`，再等待一会儿，就构建成功了。

最后，就把Dockerfile里面所有用到 mcr 的地方，换成你的镜像，例如换成 `地区.aliyuncs.com/命名空间/仓库名称:aspnet3.1-buster-slim`

## 更新
定期到阿里云更新一下（为每个构建规则点击“构建”）。别忘了本地的image也要重新pull。

如果懒的话，可以`代码变更自动构建镜像`开启，然后把构建规则设置到另一个分支上去，想更新了，就在push到构建分支。