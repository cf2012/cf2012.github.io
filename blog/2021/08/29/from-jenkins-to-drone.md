# 从Jenkins到Drone

## 我与Jenkins战斗日子

2014年~2018年我在的公司使用使用[Jenkins](https://www.jenkins.io/) 与 [rundeck]([http://rundeck.org](http://rundeck.org/))实现CI/CD.   前期主力是`Jenkins`.  使用`Jenkins`经历了几个阶段:

1. 使用Web UI点点. 复杂的逻辑用`shell`脚本实现,由`Jenkins`调用
2. 使用`Jenkins`的[pipeline](https://www.jenkins.io/doc/book/pipeline/)功能,通过[groovy](https://groovy-lang.org/)脚本封装公共逻辑,减少重复配置
3. CI/CD阶段. 实现`git2jenkins`, 用户将代码更新后,触发自动构建,发布到测试环境. 部署包存放到指定目录. 通过`rundeck`部署在线环境

### 阶段1. 使用Jenkins插件加shell脚本

整个编译发布过程有以下节点:

* 节点1. 从SCM下载源码
* 节点2. 代码编译
* 节点3. 部署到测试环境
* 节点4. 部署到在线环境
* 节点5. 单元测试
* 节点6. `sonar`扫描

每个子系统一般分3条流水线:

| 流水线 | 作用 | 包含的节点/实现逻辑 | 权限控制 | 运行方式 | 分支 | 必选? |
| ------ | ---- | ------------------- | -------- | -------- | ------ | ------ |
| 流水线1 | 发布到测试环境 | 节点1,2,3 | 开发人员、组长、测试 | 手工 | 开发分支 | Y |
| 流水线2 | 发布到生产环境 | 节点1,2,4 | 组长、测试、运维 | 手工 | master | Y |
| 流水线3 | 质量控制 | 节点1, 5,6 | 运维 | 每天定时运行 | Master | N |

### 阶段2. 使用pipeline功能

**w h y** : 后来公司开始做外部项目,需要配置的东西多.  正好有时间,所以就研究了`pipeline`功能.

**how**: 使用 `pipeline`实现了:

- 支持通过[shared-libraries](https://jenkins.io/doc/book/pipeline/shared-libraries/)复用代码
- 在SCM工具中可以看到历史版本. 方便回退.
- 方便在不同项目部署Jenkins

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





参考资料: 

[pipeline getting started](https://www.jenkins.io/doc/book/pipeline/getting-started/)

[Jenkins Best Practices - Practical Continuous Deployment in the Real World ](https://godaddy.github.io/2018/06/05/cicd-best-practices/)

[pipeline shared-libraries](https://jenkins.io/doc/book/pipeline/shared-libraries/)

## 为什么选择 Drone

(动机)

## 用Drone 如何解决之前的问题

## Drone 架构

## 写Drone插件

## 使用Drone exec练手

## 参考资料

B站视频[【用 Drone 改善團隊自動化測試及部署流程】吳柏毅 (R2-Day1) MOPCON 2018](https://www.bilibili.com/video/BV1H741137Uy?from=search&seid=1300991805432371752)

