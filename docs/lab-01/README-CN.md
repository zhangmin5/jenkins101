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

在https://github.com/登录您的 Github 帐户，  

转到https://github.com/settings/tokens，  

点击Generate new token,  

在Note添加下github-access-token-for-jenkins-on-openshift，  

选择repo、read:repo_hook、 和的范围user，  

点击Generate token,  

复制令牌并保存，您需要它来从 Github 源创建 Jenkins 管道，  

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

1. Go to the Jenkins dashboard,
1. Click Credentials, or
1. Go to Jenkins > Manage Jenkins > Configure Credentials

    ![OpenShift Jenkins credentials](../images/jenkins-credentials.png)

1. Go to Credentials > System,

    ![OpenShift Jenkins credentials](../images/jenkins-credentials-system.png)

1. In the System view, select the dropdown for Global credentials (unrestricted),

    ![OpenShift Jenkins credentials](../images/jenkins-credentials-system-add.png)

1. From the drowdown, click `Add credentials`,
1. The Jenkinsfile expects an OpenShift API token credential to be available named `openshift-login-api-token`,
1. For Kind select `Username with password`,
1. For Scope select `Global`,
1. For Username enter `token`,
1. For Password paste the OpenShift API token from the OpenShift web console login command,
1. For ID enter `openshift-login-api-token`, which is the ID that the Jenkinsfile will look for,
1. For Description enter `openshift-login-api-token`,

    ![Jenkins credentials](../images/jenkins-new-credentials.png)

1. Click OK,

    ![Jenkins new credentials](../images/jenkins-new-credential.png)

## 创建 Jenkins Pipeline

1. Make sure a project `springclient-ns` exists in OpenShift,
1. if no `springclient-ns` project exists, create it from the cloud shell,

    ```console
    oc new-project springclient-ns
    ```

1. Or via the UI, open the OpenShift web console,
1. From the top navigation dropdown, go to the `Cluster Console`,
1. Go to Administration > Projects,
1. Filter projects by `springclient-ns`,
1. If there is no such project, click `Create Project` to create it,

    ![OpenShift Jenkins credentials](../images/jenkins-new-project.png)

1. The Jenkinsfile of the spring-client application defines a stage to delete and create the `springclient-ns` project. The delete step causes an error when the project it tries to delete is missing,

1. Go back to the Jenkins dashboard. If you closed Jenkins,
1. Go to the `Application Console`, and go to the project `jenkins`,
1. Click the Route for External Traffic to open the Jenkins instance,
1. Click `Log in with OpenShift`,
1. In the Jenkins Dashboard, click `Open Blue Ocean` to open the Blue Ocean editor.

    ![Jenkins Open Blue Ocean](../images/jenkins-open-blue-ocean.png)

1. If the Welcome to Jenkins popup window shows, click the `Create a new Pipeline` button,

    ![Jenkins Create new pipeline](../images/jenkins-welcome-create-pipeline.png)

1. Otherwise, click the `New Pipeline` button in the Pipelines window. This will create a new Multibranch Pipeline,

    ![Jenkins New pipeline](../images/jenkins-new-pipeline.png)

1. Select the GitHub option,

    ![Jenkins Select SCM](../images/jenkins-select-scm.png)

1. In the Connect to GitHub section, paste the personal access token you created in your Github account,

    ![Jenkins Select Organization](../images/jenkins-which-org.png)

1. Click Connect,
1. Select the organization to where you forked the `spring-client` repository,
1. Search for and select the `spring-client` repo,

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
