# Jenkins必知必会


<!--more-->

转载请注明出处：`https://hts0000.github.io/`

欢迎与我联系：`hts_0000@sina.com`

# Jenkins介绍
`Jenkins`是一款开源`CI&CD`软件，用于自动化各种任务，包括构建、测试和部署软件。`Jenkins`本身并没有多少功能，而是使用插件来支持绝大多数功能。

## Jenkins部署
官方地址：https://www.jenkins.io/zh/download/

## Jenkins核心功能
### CICD
`Jenkins`最主要的功能就是`CICD`，`CICD`主要有三个阶段：`拉取`/`编译`/`打包`/`部署`。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121603410.png)

`Jenkins`本身并没有这些功能，而是通过插件的方式来灵活支持不同的系统。
- `拉取`阶段：支持从`git`/`github`/`gitlab`/`svn`等代码托管仓库拉取代码。
- `编译`阶段：根据编译代码的语言不同，来选择该语言的编译环境/代码审计/单元测试。
- `打包`阶段：根据不同语言的特点，可以通过插件将代码打包成二进制/jar/war/docker。
- `部署`阶段：通过ssh等方式将编译好的包部署到服务器上，或是推送docker镜像到镜像仓库中，再触发k8s更新资源。

### 运维管理
`Jenkins`通过`Role-based Authorization Strategy`插件，来支持细粒度的权限管理。比如可以控制某个角色能访问的项目和Job。

`Jenkins`通过`Credentials Binding`插件，来存储需要密文保护的密码/token。比如拉取阶段需要用到的github/gitlab token，编译阶段需要用到的数据库密码，部署阶段需要用到的ssh key/ssh密码等。

## 使用Jenkins构建github上的项目
### 使用Freestyle Project构建
使用`Freestyle Project`，适用于构建较简单的项目。官方推荐使用`Pipeline`进行构建，但是`Pipeline`有学习成本，需要学习`Jenkinsfile`的语法。

使用`Freestyle Project`进行项目构建，需要在创建item时，选择如下选项。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121717162.png)

这个配置用于显示描述信息，创建完成之后的Job详情页面，可以跳转到github仓库。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121723986.png)

这里进行github的配置，包括仓库url/拉取用户账户密码或token/仓库分支等。这里`Credentials`使用了`Credentials Binding`插件的能力，不需要明文写上账户密码了。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121725601.png)

这些都配置完之后保存，在首页启动这个Job。这里可以看到执行状态这一栏，有两个node，意味着可以同时执行两个构建任务。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121730776.png)

构建结束后，可以点击`#1`查看构建过程的输出日志，因为国内拉取github仓库经常失败，所以这里进行了多次构建，直到第三次才成功，所以可以看到版本已经到了`#4`。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121738360.png)

点击构建历史可以查看之前的构建记录，查看`#1`的输出，可以看到就是与github建立连接失败所导致的。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121739258.png)

再查看一下`#4`成功的构建记录，输出中显示拉取代码成功，仓库最后一次Commit的信息是`Update Jenkinsfile`。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121745537.png)

因为在创建Job时并没有对Build阶段进行配置，因此这个Job执行完之后，就只是简单的将代码拉取到本地的workspace中，可以在Job详情的工作空间中看到拉取的代码，也可以在本地中查看。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121748115.png)

接下来加上编译操作，将代码编译成二进制。点击配置，在Build Step中增加构建步骤。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121758253.png)

这里需要选择执行脚本的解释器，比如windows系统的就要选择windows batch command，linux选择shell。脚本内容就是编译代码的脚本。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121807721.png)

修改完成后保存，重新执行构建，构建成功后就可以在workspace中看到编译好的二进制程序。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121811741.png)

重新查看构建输出，可以看到执行的脚本内容和执行过程。  
![](https://cdn.jsdelivr.net/gh/hts0000/images/202307121812856.png)

构建成功后就到了分发部署的阶段，一般来说会将构建好的包通过ssh分发到测试环境或生产环境，或是打包成镜像push到镜像仓库，这种能力对应ssh插件：Publish Over SSH，打包成镜像也是通过shell脚本的方式打包并推送的。

### 使用Pipeline构建
使用Freestyle Project适合构建步骤比较简单的项目，如果构建步骤变得多而复杂，Freestyle Project这种图形化的管理无法直观的展示每一个构建的步骤和过程，也就是不够扁平化。

因此，对于构建步骤多而复杂的项目，Jenkins官方推荐使用Pipeline。Pipeline是一套插件，用于实现和集成持续发布的流水线到Jenkins中。

官方推荐做法是，将构建步骤组织成一个Jenkinsfile文件，存放在代码仓库中。当代码拉取下来时，Jenkins Pipeline会执行Jenkinsfile文件的内容进行构建。

Jenkinsfile文件使用一种叫做[Groovy](http://groovy-lang.org/semantics.html)的编程语言的语法来编写构建步骤，官方定义一个最简单的构建分为三大部分：Build/Test/Deploy，因此一个最简单的Jenkinsfile文件如下所示。
```Groovy
pipeline {
  // 指示Jenkins为流水线任意分配一个执行器和工作区
  agent any
  stages {
    stage('Build') { 
      steps {
        // 编写脚本内容，脚本命令执行返回非0就结束
        sh 'echo build'
      }
    }
    stage('Test') { 
      steps {
        // 编写脚本内容
        sh 'echo test'
      }
    }
    stage('Deploy') { 
      steps {
        // 编写脚本内容
        sh 'echo deploy'
      }
    }
  }
}
```

Jenkinsfile中可以定义变量，还可以读取环境变量/内置变量/参数变量等等。
```Groovy
pipeline {
  agent any
  environment {
    // 定义一个变量，全局生效
    name = 'Jay'
  }
  // 选项参数
  // 选项参数会自动注入到shell环境的env中，通过env | grep 变量名的方式可以过滤出来
  parameters {
    choice(name: "k8s_cluster", choices: [""], description: "k8s集群名称")
    choice(name: "k8s_namespace", choices: [""], description: "k8s命名空间")
  }
  stages {
    stage('Hello') {
      environment {
        // 定义一个变量，stage内生效
        age = '18'
      }
      steps {
        echo "I'm ${name}, i'm ${age} year old."
      }
    }
    stage('Env') {
      steps {
        // 过滤选项参数
        echo "选项参数:"
        sh "env | grep k8s_*"

        // 内置变量通过env来访问
        echo "JOB_NAME: ${env.JOB_NAME}"

        // 内置的env变量，查看所有支持的env变量：${YOUR_JENKINS_URL}/pipeline-syntax/globals#env
        echo "BUILD_ID: ${BUILD_ID}"  // 当前项目构建的id，比如第13次构建，就为13
        echo "BUILD_NUMBER: ${BUILD_NUMBER}" // 与BUILD_ID相同
        echo "BUILD_URL: ${BUILD_URL}"  // 构建完之后的结果url，example http://buildserver/jenkins/job/MyJobName/17/
        echo "JOB_NAME: ${JOB_NAME}"  // 项目的名称
        echo "BUILD_TAG: ${BUILD_TAG}"  // 等于${JOB_NAME}-${BUILD_NUMBER}

        // 参数变量通过params来访问，params是项目配置好的所需参数
        // 比如项目配置了一个字符参数key1和一个选项参数key2，就可以通过以下方式获取
        // 如果使用了改参数，但是项目并没有设置该参数，那么会直接报错中断运行
        echo "${key1}"
        echo "${key2}"
      }
    }
  }
}
```

parameters的用法
```Groovy
pipeline {
  parameters {
    choice(
      name: "app_git_url",
      choices: params.app_git_url ? params.app_git_url : [""],
      description: "应用gitlab仓库地址"
    )
    gitParameter(
      name: "branch_tag",
      type: "PT_BRANCH_TAG",
      defaultValue: "master",
      description: "分支或标签"
    )
    string(
      name: "dockerfile",
      defaultValue: params.dockerfile ? params.dockerfile : "",
      description: "dockerfile文件路径",
      trim: true
    )
    choice(
      name: "jdk_version",
      choices: ["", "/usr/local/java-se-1.7.0"],
      description: "使用的jdk路径，默认使用jdk1.8"
    )
    string(
      name: "mvn_arg",
      defaultValue: params.mvn_arg ? params.mvn_arg : "-s /var/lib/jenkins/.m2/settings-hd.xml clean deploy -U -Dmaven.test.skip=true -Dmaven.test.skip=true",
      description: "mvn命令参数",
      trim: true
    )
    choice(
      name: "k8s_cluster",
      choices: params.k8s_cluster ? params.k8s_cluster : [""],
      description: "k8s集群名称"
    )
    choice(
      name: "k8s_namespace",
      choices: params.k8s_namespace ? params.k8s_namespace : [""],
      description: "k8s命名空间"
    )
    choice(
      name: "k8s_app_name",
      choices: params.k8s_app_name ? params.k8s_app_name : [""],
      description: "k8s应用名称"
    )
    choice(
      name: "k8s_pod_number",
      choices: params.k8s_pod_number ? params.k8s_pod_number : ["1"],
      description: "k8s pod数量"
    )
    choice(
      name: "target_dir",
      choices: params.target_dir ? params.target_dir : [""],
      description: "打包后jar包target目录，默认为工程根目录的target"
    )
  }
}
```

when的用法
```Groovy
pipeline {
  stages {
    stage("Pre Check") {
      when {
        expression {
          fileExists(params.dockerfile)
        }
      }
      steps {
        echo "预检查通过"
      }
    }
  }
}
```

environment的值，可以动态从params获得

Jenkins Pipeline的一些示例：https://www.jenkins.io/doc/pipeline/examples/

Pipeline所用编程语言Groovy文档：http://groovy-lang.org/semantics.html

### 权限管理
下载`Role-Based Strategy`插件，在配置->Security->Authorization->Authorization-，选择Role-Based Strategy，然后在Manage Jenkins中就会多出一个Manage and Assign Roles配置。

role分为三种，global role(常用)，item role(常用)，agent role(不常用)。

给一个用户指定查看某些项目的限制：
1. 创建用户
2. 创建一个全局角色——read，只在Overall下拥有Read权限。(注意Job、View这两个块都不能设置Read权限，不然会覆盖Item role Pattern的配置)
3. 创建item role——hello，并配置希望匹配的pattern
4. 给用户赋予全局角色read和item role角色hello
5. 然后就能看到该用户只有指定项目的权限

#### 特殊用户
在给用户添加global role时，可以添加`authenticated`用户，这个特殊用户不需要创建，给他global role相当于给所有用户。

### API访问及构建
Jenkins提供了REST API来操作各个功能，比如job的crud等等。

使用REST API需要经过认证，可以在用户设置中的`API Token`生成`Token`，给后续的api使用。

#### 例子
**列出所有可用API**

```bash
curl -u hetiansheng:11b446b0d5a5b503f19f095cb290c3f718 http://xxx.miniso.com:8080/api/json?pretty=true
```

**对指定job进行构建**

```bash
# job没有参数
curl -u hetiansheng:11b446b0d5a5b503f19f095cb290c3f718 http://jenkins1.miniso.com:8080/job/${JOB_NAME}/build

# job有参数
# -d中的参数要满足需求，构建才会开始
# 比如key2是个选项参数，只有选中已有选项才合法
curl -u hetiansheng:11b446b0d5a5b503f19f095cb290c3f718 -d "key1=test1&key2=value-2" http://jenkins1.miniso.com:8080/job/hello-api/buildWithParameters
```

使用默认参数进行构建
curl -u hetiansheng:11b446b0d5a5b503f19f095cb290c3f718 http://jenkins1.miniso.com:8080/job/hello-api/buildWithParameters

#### pipeline进阶


### 根据tag来构建
在参数化构建选项中，添加git参数选项，需要git插件的支持。
![](https://cdn.jsdelivr.net/gh/hts0000/images/202403161146483.png)

添加完之后可能会出现拉取不到tag的情况。解决这个问题的方法：
1. 检查git仓库地址是否正确
2. 检查pipeline中使用的credientalId是否正确
3. 如果配置没问题，可以先手动配置默认值为最新tag，然后开始构建，后续就能自动拉取tag了

不同插件在pipeline中有不同表现，以`GitSCM`插件为例，在pipeline中根据tag来拉取代码。
```Groovy
pipeline{
  // 定义本次构建使用哪个标签的构建环境，本示例中为 “slave-pipeline”
  agent any
  // agent {
  //   node {
  //     label 'master'
  //   }
  // }

  // colorful message
  options {
    ansiColor('xterm')
  }

  // 定义groovy脚本中使用的环境变量
  environment{
    // 将构建任务中的构建参数转换为环境变量
    // 目标应用仓库地址
    LOCAL_APP_GIT_URL = sh(returnStdout: true, script: 'echo $app_git_url').trim()
    // 目标应用仓库分支
    LOCAL_BRANCH = sh(returnStdout: true, script: 'if [ -z "${branch}" ]; then echo master; else echo ${branch}; fi').trim()
    // GIT仓库秘钥ID
    LOCAL_CREDENTIALS_ID = "ci-jenkins"
    // maven参数
    LOCAL_MVN_ARG = sh(returnStdout: true,script: 'if [ -z "${mvn_arg}" ] ;then echo "clean package -U -B -DskipTests" ;else echo ${mvn_arg} ;fi').trim()
  }

  // "stages"定义项目构建的多个模块，可以添加多个 “stage”，可以多个 “stage” 串行或者并行执行
  stages {
    // 定义第一个stage， 完成克隆源码的任务
    stage('Git') {
      steps {
        // git branch: '${branch}', credentialsId: env.LOCAL_CREDENTIALS_ID, url: env.LOCAL_APP_GIT_URL
        checkout([
          $class: 'GitSCM',
          branches: [[name: "refs/tags/${branch}"]],
          doGenerateSubmoduleConfigurations: false,
          userRemoteConfigs: [[credentialsId: env.LOCAL_CREDENTIALS_ID, url: env.LOCAL_APP_GIT_URL]],
        ])
      }
    }

    // 添加第二个stage，运行源码打包命令，并由maven统一推送到镜像仓库
    stage('Package') {
      steps {
        script {
          sh "JAVA_HOME=${jdk_version} mvn ${LOCAL_MVN_ARG}"
        }
      }
    }
  }
}
```

### 常见函数
#### sh
`sh '${params.test}'`无法获取到参数，需要使用`sh "${params.test}"`。也支持另一种格式`sh(returnStdout: true, script: "echo ${params.test}")`

### 常用插件
#### [build user vars](https://plugins.jenkins.io/build-user-vars-plugin/)
用法
```Groovy
pipeline {
  stages {
    stage('Display Build User') {
      steps {
        wrap([$class: 'BuildUser']) {
          echo "$env.BUILD_USER_ID"
        }
      }
    }
  }
}
```
