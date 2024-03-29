# 从Jenkins到Drone

## 我与Jenkins战斗日子

2014年~2018年我在的公司使用使用[Jenkins](https://www.jenkins.io/) 与 [rundeck]([http://rundeck.org](http://rundeck.org/))实现CI/CD.   前期主力是`Jenkins`.  使用`Jenkins`经历了几个阶段:

1. 使用Web UI点点. 复杂的逻辑用`shell`脚本实现,由`Jenkins`调用
2. 使用`Jenkins`的[pipeline](https://www.jenkins.io/doc/book/pipeline/)功能,通过[groovy](https://groovy-lang.org/)脚本封装公共逻辑,减少重复配置
3. CI/CD阶段. 实现`git2jenkins`, 用户将代码更新后,触发自动构建,发布到测试环境. 部署包存放到指定目录. 通过`rundeck`部署在线环境

### 阶段1. 使用Jenkins插件加shell脚本

**why**:刚开始使用`Jenkins`. 从网上找教程,跟大牛学习配置流水线. 有些复杂功能用`shell`实现会简单点. 比如: 上传文件到某主机指定目录.然后执行重启操作.

**how**: 整个编译发布过程有以下节点:

* 节点1. 从SCM下载源码
* 节点2. 代码编译
* 节点3. 单元测试. 可选
* 节点4. 部署到指定环境. CI/STG/PRD
* 节点5. `sonar`扫描. 可选

### 阶段2. 使用pipeline功能

**w h y** : 后来公司开始做外部项目,需要配置的东西多.流行 `Configuration as a code` . 正好有时间,所以就研究了`pipeline`功能.

**how**: 使用 `pipeline`实现了:

- 支持通过[shared-libraries](https://jenkins.io/doc/book/pipeline/shared-libraries/)复用代码
- 在SCM工具中保存了`pipeline`配置,可以看到历史版本. 方便回退. 不用担心`jenkins` 配置丢失了
- 封装封装... 方便在不同项目部署Jenkins

最终效果:

```groovy
//file: dev_publish_web_bss.groovy. Jenkins直接调用
@Library('my_tools') _

// 封装的函数. 将下载、编译、发布全封装了. 根据入参选不同的配置 
// 封装了发送通知模块.实现通过叮叮发送通知,告知发布进度
projectxxWebDelivery(
	Job_Name: "web-bss", 
	Job_Desc: "开发环境前端-bss",
	Deploy_To: "dev",
)
```

p.s. 太复杂了.走火入魔的感觉 😓

### 阶段3, CI&CD

**why**: 偷懒,不想每次手工点`jenkins`. 当然还是有时间. 

**how**: 实现功能有:

1. `git2jenkins`功能. 用户提交代码可以触发`Jenkins`构建. 这样提交代码就会自动做单元测试,成功后可以发布到CI环境
2. 每个版本构建的结果保存起来.`war`包或`docker image` .  并将版本号记录到`DB` 中
3. 部署时,可以从`db`中选择指定的版本部署. 不用再重新编译. 通过`rundeck`操作

其中`git2jenkins`功能实现流程:

1. `gitee`上配置`webhook` 有人`push`代码时,调用`rundeck`. 见我的项目: [gitee2rundeck-adapter](https://github.com/cf2012/gitee2rundeck-adapter)
2. `rundeck` 收到请求时,根据逻辑调用`jenkins`构建

p.s. 使用`rundeck`中转的原因:  `rundeck`的权限控制做的比较好

### 反思

`pipeline`功能很强大, 但[阶段2](#阶段2. 使用pipeline功能[)追求功能复用导致新人接手门槛高. 如果再做, 我会采用通过`python`程序用模版生成`pipeline`脚本来实现. 😄

## 为什么选择 Drone

**w h y**:  使用`golang`编写,比较酷.  配置比`jenkins`的`pipeline`简单易懂. 使用`docker`技术,方便使用`k8s`资源

## 用Drone 如何解决之前的问题

需要有以下功能:

* 流水线内部支持串行与并行
* 节点下如果有多个子节点,支持跳过若干个
* 二次开发能力 & 开发插件简单

## Drone 架构

## 写Drone插件

## 使用Drone exec练手

## 参考资料

B站视频[【用 Drone 改善團隊自動化測試及部署流程】吳柏毅 (R2-Day1) MOPCON 2018](https://www.bilibili.com/video/BV1H741137Uy?from=search&seid=1300991805432371752)

[pipeline getting started](https://www.jenkins.io/doc/book/pipeline/getting-started/)

[Jenkins Best Practices - Practical Continuous Deployment in the Real World ](https://godaddy.github.io/2018/06/05/cicd-best-practices/)

[pipeline shared-libraries](https://jenkins.io/doc/book/pipeline/shared-libraries/)

## 
