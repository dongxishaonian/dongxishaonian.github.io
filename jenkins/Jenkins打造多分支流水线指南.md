[返回上层](index)

# Jenkins打造多分支流水线指南

## overview：

多分支工作流程带来了以下几个关键能力：

1. 在代码仓库中，每个新分支都有自己单独的工作流水线（job）。
2. 每个工作流水线都记录了对应分支的构建和变更历史。
3. 可以自定义设置流水线随着分支的删除而删除或修建。
4. 通过重写父属性（如果需要），可以灵活地单独配置分支流水线属性。

Jenkins pipeline-as-code 使您可以在项目/应用程序源代码存储库中维护CI / CD工作流逻辑，而无需在Jenkins中为每个分支维护其配置。用于构建/测试/部署的流水线代码始终和你的项目/应用程序源代码同步。在仓库中我们用jenkinsfile对流水线代码进行描述。关于jenkinsfile，其简介及语法可参考[官方文档](https://jenkins.io/zh/doc/book/pipeline/jenkinsfile/ )

---

# do it：

### 1.jenkins需要安装多分支流水线插件：

首先打开插件中心：jenkins>Manage Jenkins>Manage Plugins

并且安装如下两个插件（有可能已经安装了）：

![image-20200403153130615](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162121.png)

### 2.新建一个多分支流水线项目：

#### 2.1 jenkins>新建Item

![image-20200403153717166](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162126.png)

#### 2.2 填写项目，代码源相关信息

![image-20200403155115959](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162134.png)

创建完之后在首页会显示

![image-20200403155507717](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-163054.png)

创建完成。

### 3.接下来在我们的项目根目录添加jenkinsfile（以下用已经存在的项目做演示）

![image-20200403155829285](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162140.png)

然后在Jenkinsfile中编写流水线代码(pipeline代码语法请参考 [语法](https://jenkins.io/zh/doc/book/pipeline/jenkinsfile/ )，以下为示例

```json
pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '7', artifactNumToKeepStr: '10', daysToKeepStr: '5'))
        timeout(time: 12, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
    agent {
        label 'master'
    }

    environment {
        JOB_NAME = 'pipeline-demo'
    }

    parameters {
            booleanParam(name: 'FAST_MODE', defaultValue: false, description: '此操作将会跳过单元测试以及代码质量检查。')
    }

    stages {
        stage('pipeline环境准备') {

            steps {
                script {
                    echo "开始构建"
                    if(!env.BRANCH_NAME.startsWith('feature-') && !env.BRANCH_NAME.startsWith('release-')){
                        error("自动构建分支名称必须以feature-或release-开头，当前分支名称为: ${env.BRANCH_NAME}")
                    }

                    if (env.BRANCH_NAME.startsWith('feature-') ) {
                        env.env = "beta"
                    }
                    if (env.BRANCH_NAME.startsWith('release-')) {
                        env.env = "stage"
                    }

                    sh "echo 当前分支 : ${env.BRANCH_NAME}"
                    sh "echo 当前环境 : ${env.env}"
                    sh "echo 当前提交 : ${env.commit}"
                    sh "echo WORKSPACE : ${env.WORKSPACE}"
                    sh "echo GIT_BRANCH : ${env.GIT_BRANCH}"
                    sh "echo BUILD_NUMBER : ${env.BUILD_NUMBER}"
                    sh "echo JOB_NAME : ${env.JOB_NAME}"
                    sh "./mvnw -v"
                    sh "java -version"
                }
            }
        }

        stage("运行测试&收集报告"){
            when{
                expression {
                    params.FAST_MODE == false
                }
            }
           steps{
                script {
                    echo "开始运行测试"
                    sh "./mvnw clean test jacoco:report"
                }
           }
        }

        stage('代码静态检查') {
            when{
                expression {
                    params.FAST_MODE == false
                }
            }
            steps {
                withSonarQubeEnv( installationName: 'sonar_server') {
                    sh './mvnw sonar:sonar'
                }
            }
        }

        stage("检查结果分析") {
            when{
                expression {
                    params.FAST_MODE == false
                }
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage("发布应用") {
            steps {
                script {
                    echo "开始发布"
                    sh "curl --location --request POST 'http://0.0.0.0:8080/job/${env.JOB_NAME}/buildWithParameters' \
                        --header 'Authorization: ${env.ecarx_jenkins_auth}' \
                        --form 'env=${env.env}' \
                        --form 'branchname=origin/${env.BRANCH_NAME}'"
                }
            }
        }
    }
}
```

### 4.在我们的代码仓库中添加webhook

如下（示例中使用gitlab，如果是其他仓库，可参考各仓库文档）：

![http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162148.png](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162148.png)

之后点击add webhook保存。

⚠️：每个不同的代码仓库可能webhook地址的组成不同，所以添加前可查看各个仓库文档。

准备就绪。

### 5.push代码

将带有Jenkinsfile的项目代码push的远程代码仓库，回到jenkins控制台

![image-20200403161653851](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162155.png)

每当有分支push代码时，都会自动触发Jenkins的自动构建。

![image-20200403162112928](http://dxsn-1300740068.cos.ap-nanjing.myqcloud.com/2021-12-10-162202.png)

从而实现ci/cd。

### 6.总结：

jenkins能让我们轻松实现持续集成/持续部署（ci/cd）。ci/cd让我们实现代码质量内建，ci/cd中最重要的是测试自动化，没有自动化测试的持续集成只是一堆不会带来任何用处的垃圾。我们在流水线中嵌入测试自动化，代码质量检查来保证我们的开发质量。流水线能够及时给开发者反馈，这种反馈非常，当我们的流水线失败的时候，我们需要第一时间修复它，从而做到不积累结束债务，而不是继续开发别的功能。否则等到失败积累到一定程度，我们在去修复的时候，需要付出的成本将是更大的。


---
---
---


## 🤔  💭 👇👇👇

<script src="https://utteranc.es/client.js"
        repo="dongxishaonian/issue-posted"
        issue-term="pathname"
        label="🙂🙃😡🥶😬🤣😄"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

