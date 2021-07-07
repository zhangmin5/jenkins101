# 在 OpenShift 4.x 上使用 Jenkins 创建经典管道

## 要求

* IBM Cloud 帐户  
* 一个至少有 2 个工作节点的 OpenShift 4.x 集群  
* 使用occli 和ibmcloudcli访问终端，使用Skills Network或IBM Cloud shell。有关说明，请转到此处。  
* OpenShift 4.x 上的 Jenkins 实例，请参阅下面的设置，  
* https://github.com/remkohdev/spring-client的 Github 分支，  
* Github 个人访问令牌  

## 在 OpenShift 4.x 上设置 Jenkins  

转到在 OpenShift 4.x 上设置 Jenkins以使用 Jenkins Operator在 OpenShift 4.x上完成 Jenkins 设置和配置。  

## spring-client在 Github 中fork应用程序

1. 要创建 spring-client 存储库的分支： 

1. 转到https://github.com/remkohdev/spring-client， 

1. 单击右侧的Fork按钮在您自己的 GitHub 组织中创建一个分支，例如https://github.com/<username>/spring-client 

1. 查看包含在 Spring Client 存储库中的 Jenkinsfile，  

1. 编辑 Jenkinsfile， 

1. 将登录命令复制到您的 OpenShift 集群.

    ![OpenShift Copy Login Command](../images/copy-login-command.png)

    ```console
    oc login https://c100-e.us-south.containers.cloud.ibm.com:30645 --token=CgwpwTu12sJV3u45iFFWd-6V7JsD8b90JBoJk1zGR2I
    ```

    1. 在 的environment部分中Jenkinsfile，将LOGIN_URL和更改LOGIN_PORT为匹配
    ```console
    pipeline {
    agent any
    tools { 
      maven 'maven'
    }
    environment {
        LOGIN_URL = 'https://c100-e.us-south.containers.cloud.ibm.com'
        LOGIN_PORT = '30645'
    }  
    ```

    注意：将配置详细信息留在您的存储库中是不合适的，更不用说在公共 Github 上了，但为了简单起见，我在这里定义了 URL 和 NodePort。  

将更改提交Jenkinsfile到您的 Github 分支。Jenkins 管道将使用您Jenkinsfile将分叉部署spring-client到您自己的 OpenShift 集群。  
## 创建 Github 个人访问令牌  

1. 在https://github.com/登录您的 Github 帐户，  

1. 转到https://github.com/settings/tokens，  

1. 点击Generate new token,  

1. 在Note添加下github-access-token-for-jenkins-on-openshift，  

1. 选择repo、read:repo_hook、 和的范围user，  

1. 点击Generate token,  

1. 复制令牌并保存，您需要它来从 Github 源创建 Jenkins 管道，  

    ![github personal access token 1](../images/github-personal-access-token-1.png)

    ![github personal access token 2](../images/github-personal-access-token-2.png)

1. 例如创建一个环境变量 GITHUB_TOKEN，  
    ```console
    export GITHUB_TOKEN=<your token>
    ```

## 配置 Jenkins 对 OpenShift 的访问  

1. 再次转到 OpenShift Web 控制台或再次使用Copy Login Command之前的,  
1. 从登录的用户配置文件下拉列表中，单击Copy Login Command,

    ```console
    oc login https://<your-openshift-url>:<your-openshift-port> --token=<your-openshift-api-token>
    ```

1. 复制 OpenShift API token value, e.g. aaHYcMwUyyusfNaS45aAiQer_Kas1YUa45YTA2AxsNI,

1. 转到 Jenkins dashboard,
1. 点击 Credentials, or
1. 转到 Jenkins > Manage Jenkins > Configure Credentials

    ![OpenShift Jenkins credentials](../images/jenkins-credentials.png)

1. 转到 Credentials > System,

    ![OpenShift Jenkins credentials](../images/jenkins-credentials-system.png)

1. 在 System view,选择r Global credentials (unrestricted),

    ![OpenShift Jenkins credentials](../images/jenkins-credentials-system-add.png)
    
1. 从下拉列表中，单击Add credentials，

1. Jenkinsfile 期望一个名为 的 OpenShift API 令牌凭证可用openshift-login-api-token，

1. 对于种类选择Username with password，

1. 对于范围选择Global，

1. 对于用户名输入token，

1. 对于密码，从 OpenShift Web 控制台登录命令粘贴 OpenShift API 令牌，

1. 对于 ID 输入openshift-login-api-token，这是 Jenkinsfile 将查找的 ID，

1. 对于描述输入openshift-login-api-token， 
    ![Jenkins credentials](../images/jenkins-new-credentials.png)

1. 点击 OK,

    ![Jenkins new credentials](../images/jenkins-new-credential.png)

## 创建 Jenkins Pipeline

1. 确保springclient-nsOpenShift 中存在一个项目,
1. 如不存在，新建一个项目

    ```console
    oc new-project springclient-ns
    ```

1. 或者通过 UI，打开 OpenShift Web 控制台,
1. 从顶部导航下拉菜单中，转到Cluster Console,
1. 转到 Administration > Projects,
1. Filter projects by `springclient-ns`,
1. 如果没有这个项目，点击Create Project创建,

    ![OpenShift Jenkins credentials](../images/jenkins-new-project.png)

1. spring-client 应用程序的 Jenkinsfile 定义了删除和创建springclient-ns项目的阶段。当它尝试删除的项目丢失时，删除步骤会导致错误,

1. 转到 the Jenkins dashboard. 
1. 转到 `Application Console`, and go to the project `jenkins`,
1. 点击 the Route for External Traffic 打开jenkins,
1. 点击 `Log in with OpenShift`,
1. 在 Jenkins Dashboard, 点击 `Open Blue Ocean`  打开 the Blue Ocean editor.

    ![Jenkins Open Blue Ocean](../images/jenkins-open-blue-ocean.png)

1. 如果欢迎使用 Jenkins 弹出窗口显示，请单击Create a new Pipeline按钮， 

    ![Jenkins Create new pipeline](../images/jenkins-welcome-create-pipeline.png)

1. 否则，单击New Pipeline“管道”窗口中的按钮。这将创建一个新的多分支管道， 

    ![Jenkins New pipeline](../images/jenkins-new-pipeline.png)

1. 选择 GitHub 选项， 
    ![Jenkins Select SCM](../images/jenkins-select-scm.png)

1. In the Connect to GitHub section, paste the personal access token you created in your Github account,

    ![Jenkins Select Organization](../images/jenkins-which-org.png)

1. 点击 Connect, 
1. Select the organization to where you forked the `spring-client` repository,
1. 搜索和选择 `spring-client` repo,

    ![Jenkins Choose Repo](../images/github-token-choose-repo.png)

1. 单击创建 Pipeline,
1. 管道创建完成后，会自动触发构建,

    ![Jenkins Run Pipeline](../images/jenkins-run-pipeline.png)

1. 您应该看到管道的成功构建 ,
1. 如果发生错误，可以通过展开阶段上的红色十字指示来调试管道，表示该阶段管道失败,
1. 查看日志,

    ![Jenkins Successful Build](../images/jenkins-error-1.png)

    ![Jenkins Successful Build](../images/jenkins-error-2.png)

1. 对 Github 存储库的任何更新，例如推送更新 Jenkinsfile、Spring Boot 应用程序的源代码或 README.md 文件，都将自动触发管道的新构建,

    ![Jenkins Successful Build](../images/jenkins-build-trigger.png)

1. 使用项目 `oc project springclient-ns`,

    ```console
    oc project springclient-ns
    ```

    outputs,

    ```console
    $ oc project springclient-ns
    Now using project "springclient-ns" on server "https://c100-e.us-south.containers.cloud.ibm.com:30645".
    ```

1.获取路由并测试部署 ,

    ```console
    ROUTE="$(oc get route springclient -o json | jq -r '.spec .host')"
    curl -X GET http://$ROUTE/api/hello?name=you
    ```

    输出,

    ```console
    ROUTE="$(oc get route springclient -o json | jq -r '.spec .host')"
    curl -X GET http://$ROUTE/api/hello?name=you
    { "message" : "Hello you" }
    ```
