# Creating the devops Environment For the Estafet Microservices Scrum Demo Application
## Contents

* [Overview](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#prerequisites#overview)
* [Prerequisites](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#prerequisites)
* [Forking the GitHub Repositories](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#forking-the-github-repositories)
* [Creating the Infrastructure](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#creating-the-infrastructure)
* [Deploying OpenShift](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#deploying-openshift)
* [Creating the Devops Environments](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#creating-the-devops-environments)
* [Configuring Jenkins](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#configuring-jenkins)

## <a name="overview"/>Overview

The Devops environment for the Estafet Microservices Scrum (EMS) demo application runs on Amazon Web Services (AWS).

> Note:- The Terraform scripts in [the original GitHub repository](https://github.com/Estafet-Ltd/estafet-microservices-scrum "The original GitHub repository")
> require an AWS Elastic IP (EIP) for each host. Because the global IPV4 Address pool is nearly exhausted, AWS impose a
> strict limit on static IP V4 allocation - our AWS account has a limit of five EIP's across the _whole account_. For this
> reason, the environment created by [this fork](https://github.com/Estafet-Ltd/estafet-microservices-scrum "Steve Brown's Fork")
> uses only two EIP's: one for the master node and one for the bastion.

The AWS environment consists of four nodes:

1. The `bastion`, from which the EMS deployments are controlled.
1. The Master node for OpenShift
1. Two OpenShift compute nodes - `node1` and `node2`

There is also a PostgreSQL database, implemented on AWS [RDS](https://aws.amazon.com/rds/).

## <a name="prerequisites"/>Prerequisites

1. You must have access to an AWS account. Estafet employees can be added to the Estafet AWS account.

1. You must configure your [AWS IAM Security Credentials](https://eu-west-2.console.aws.amazon.com/console/home?region=eu-west-2# "AWS settings menu"):

   ![AWS settings menu](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/aws_settings_menu.png)

   Choose `My Security Credentials` and follow the instructions. You only need to configure your IAM security credentials.

   You must create the `~/.aws/credential` file:

   ```
   [default]
   aws_access_key_id = <your AWS access key>
   aws_secret_access_key = <your AWS secret access key>
   ```

1. You must have Ansible, Terraform and the Openshift client installed on your laptop:

   | Package          | Version       | Details |
   | -------------    |:--------------|:--------|
   | Ansible          | `2.8.4`       | [Ansible Documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-release-via-dnf-or-yum)
   | Terraform        | `v0.11.14`    | [Terraform download page](https://releases.hashicorp.com/terraform/0.11.14/terraform_0.11.14_linux_amd64.zip)
   | Openshift Client | `v3.11`       | [OKD GitHub Releases page](https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz)

   Use yum to install Ansible.

   Terraform and the Openshift Client Tools are shipped as executables. Uncompress the downloaded files and put the
   executables in `/usr/local/bin'`

1. You must have cloned [the original GitHub repository](https://github.com/Estafet-Ltd/estafet-microservices-scrum "The original GitHub repository"), made
a GitHub fork of it, or cloned a GitHub fork of it.

    > Note:-  If you have forked the original GitHub repository, you must fork all the sub modules and make sure that your fork
    > cannot update [the original GitHub repository](https://github.com/Estafet-Ltd/estafet-microservices-scrum "The original GitHub repository") __or any of its submodules__

## <a name="forking-the-github-repositories"/>Forking the GitHub Repositories

The policy for the ESM demo is to take forks of the original repositories, so as to avoid inadvertently breaking the ESM demo.

> Note: In this document, the owner of the new forks is `newowner` . Replace `newowner` with your GitHub login name.

To fork the GitHub repositories:

1. Login to GitHub Fork the [the original GitHub repository](https://github.com/Estafet-Ltd/estafet-microservices-scrum "The original GitHub repository")

   Click on `Fork` and follow the instructions

1. <a name="microservices-list"/> Fork the GitHub repository for each microservice:

    | Repository        | Description |
    | ----------------- |-------------|
    | [estafet-microservices-scrum-api-project](https://github.com/Estafet-LTD/estafet-microservices-scrum-api-project) | Microservices for managing scrum projects. |
    | [estafet-microservices-scrum-api-project-burndown](https://github.com/Estafet-LTD/estafet-microservices-scrum-api-project-burndown) | Microservices for aggregating and generating project burndown reports. |
    | [estafet-microservices-scrum-api-sprint](https://github.com/Estafet-LTD/estafet-microservices-scrum-api-sprint) | Microservices for managing sprints. |
    | [estafet-microservices-scrum-api-sprint-board](https://github.com/Estafet-LTD/estafet-microservices-scrum-api-sprint-board) | Microservices for aggregating and rendering a sprint board. |
    | [estafet-microservices-scrum-api-sprint-burndown](https://github.com/Estafet-LTD/estafet-microservices-scrum-api-sprint-burndown) | Microservices for aggregating and generating sprint burndown reports. |
    | [estafet-microservices-scrum-api-story](https://github.com/Estafet-LTD/estafet-microservices-scrum-api-story) | Microservices for managing stories. |
    | [estafet-microservices-scrum-api-task](https://github.com/Estafet-LTD/estafet-microservices-scrum-api-task) | Microservices for managing tasks. |
    | [estafet-microservices-scrum-basic-ui](https://github.com/Estafet-LTD/estafet-microservices-scrum-basic-ui) | Basic User Interface that uses the scrum microservices. |
    | [estafet-microservices-scrum-qa](https://github.com/Estafet-LTD/estafet-microservices-scrum-qa) | Cross cutting Quality Assurance tests. |

1. Clone your new fork of [the original GitHub repository](https://github.com/Estafet-Ltd/estafet-microservices-scrum "The original GitHub repository"):

   ```
    $ git clone --recurse-submodules git@github.com:newowner/estafet-microservices-scrum.git estafet-microservices-scrum-newowner
   ```
    or
   ```
    $ git clone https://github.com/newowner/estafet-microservices-scrum.git estafet-microservices-scrum-newowner
   ```
    then:
   ```
    $ cd estafet-microservices-scrum-newowner
   ```

1. Make sure you are on the master branch:

   ```
    $ git checkout master
   ```
1. Edit the .gitmodules file:

   Change all occurrences of "`Estafet-LTD`" to your GitHub login name, e.g. for the `estafet-microservices-scrum-api`
   submodule:

   ```
   [submodule "estafet-microservices-scrum-api-project"]
    path = estafet-microservices-scrum-api-project
    url = git@github.com:newowner/estafet-microservices-scrum-api-project.git

   The rest of the file has been omitted for brevity.
   ```
1. Synchronise Git to use your forks, rather than the original repositories:

   ```
    $ git submodule sync
   ```
1.  Commit and push this change:

   ```
    $ git commit .gitmodules -m "Switch to my forks."
    $ git push
   ```
1. Ensure all the submodules are on the master branch:

   ```
    $ git submodule foreach 'git checkout master || :'
   ```
1. Ensure that there are no hardcoded references to `Estafet-LTD` in any of your repositories:

    Change all occurrences of `Estafet-LTD` to your GitHub login name in all instances of these files:

    * pom.xml
    * *.groovy
    * *.yml
    * *.sh

1. Commit and push all submodule changes:

   ```
    $ git submodule foreach git 'commit -a -m "Switch hard-coded GitHub references to my forks." || :'
    $ git submodule foreach 'git push || :'
   ```

1. Commit and push all changes to the main repository:

   ```
    $ git commit -a -m "Switch hard-coded GitHub references to my forks."
    $ git push
   ```

Your GitHub repositories are now completely separated from the original repositories and you cannot inadvertently make changes
to any of the original repositories.

## <a name="creating-the-infrastructure"/>Creating the Infrastructure

To create the AWS EC2 instances and the AWS RDS database servers:

   ```
    $ cd  estafet-microservices-scrum-newowner
    $ make infrastructure
   ```
 The makefile takes about 30 minutes to run.

 ## <a name="deploying-openshift"/>Deploying OpenShift

To deploy OpenShift to the infrastructure you just created:

   ```
    $ make openshift
   ```
 The makefile takes about 30 minutes to run. The makefile deploys OpenShift to the infrastructure created in the
 previous step. It also runs the `scripts/postinstall-master.sh` and `scripts/postinstall-node.sh` scripts.

 This will install OpenShift on the AWS nodes.

 ## <a name="creating-the-devops-environments"/> Creating the Devops Environments

 The DevOps environments must be created from the bastion server. This is because the compute nodes have no static public
 IP addresses or DNS names. The private IP addresses and DNS names of AWS EC2 instances are static.

1. Get the OKD console URL:

   ```
   [stevebrown@6r4nm12 estafet-microservices-scrum]$ make master-url
   echo $(terraform output master-url)
   https://3.9.50.47.xip.io:8443
   [stevebrown@6r4nm12 estafet-microservices-scrum]$

   ```
1. Get the RDS database DNS name:

   ```
   (vaws) [stevebrown@6r4nm12 estafet-microservices-scrum]$ aws rds describe-db-instances \
   --db-instance-identifier microservices-scrum | \
   jq -r '.DBInstances[] | select(.DBName=="scrumdb") | .Endpoint.Address'
   microservices-scrum.cfzuhvugfmcm.eu-west-2.rds.amazonaws.com
   (vaws) [stevebrown@6r4nm12 estafet-microservices-scrum]$
   ```
1. SSH onto the bastion:

   The makefile has targets to ssh to the bastion and the master node.

   ```
   ssh-bastion:
       ssh -t -A ec2-user@$$(terraform output bastion-public_ip)
   ssh-master:
       ssh -t -A ec2-user@$$(terraform output bastion-public_ip) ssh master.openshift.local
   ```

   ssh on to the bastion:

   ```
   $ make ssh-bastion
   [stevebrown@6r4nm12 estafet-microservices-scrum]$ make ssh-bastion
   ssh -t -A ec2-user@$(terraform output bastion-public_ip)
   Last login: Tue Sep 24 07:32:50 2019 from xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

           __|  __|_  )
            _|  (     /   Amazon Linux AMI
          ___|\___|___|

    https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/

    [ec2-user@ip-10-0-1-136 ~]$
   ```
    The Ansible script to create the DevOps environments on AWS is `ansible/create-devops-environments-playbook.yml`

1. Create the inventory file from the template:

   Substitute the values from steps 1 and 2.

   ```
   $ cp inventory.template inventory
   $ vi inventory
   localhost ansible_connection=local openshift=<OKD Console URL> database=<RDS Database DNS name>
   ```
1. Run the Ansible playbook to create the DevOps environments:

   ```
   $ cd estafet-microservices-scrum/ansible
   $ ansible-playbook -vv create-devops-environments-playbook.yml
   ```
   The script creates these environments:

   * `dev`
   * `cicd`
   * `test`
   * `prod`

   All the EMS microservices are deployed to the `dev` environment and the database for each microservice is created in
the RDS database instance.

   Jenkins and Nexus are deployed to the `cicd` environment. The `cicd` environment contains the build and release
pipelines for each of the EMS microservices.

1. <a name="get-jenkins-host"/>Get the Jenkins Host

   ```
   [ec2-user@ip-10-0-1-136 ansible]$ oc get route jenkins -n cicd -o jsonpath="{.spec.host}";echo ""
   jenkins-cicd.3.9.50.47.xip.io
   [ec2-user@ip-10-0-1-136 ansible]$
   ```

## <a name="configuring-jenkins"/>Configuring Jenkins

Point a browser at `https://<jenkins host>`. The login credentials are `admin` and `123`.

There are some manual steps needed to configure Jenkins.

* [Configure the Jenkins Maven Settings File](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#configure-jenkins-maven-settings-file)
* [Install the Jenkins Pipeline Maven Plugin](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#install-jenkins-pipeline-maven-plugin)
* [Create GitHub credentials](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#create-github-credentials)
* [Configure GitHub Webhooks](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#configure-github-webhooks)

### <a name="configure-jenkins-maven-settings-file"/> Configure the Jenkins Maven Settings File

The Jenkins main screen is:

![Jenkins main page](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_main_page.png)

Choose the `cicd` link:

![Jenkins cicd page](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_cicd_page.png)

Choose `Config Files`:

![Jenkins cicd add config files page](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_cicd_config_files_page.png)

Choose `Add a new Config`:

![Jenkins add Config file page](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_cicd_add_config_file_page.png)

1. Select `Global Maven settings.xml`
1. Set the ID field to `microservices-scrum`
1. Click on `Submit`

![Jenkins edit Jenkins Maven configuration](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_edit_global_maven_config_file.png)

1. Set the ID field to `microservices-scrum`
1. Set the Name field to `Microservices Scrum Global Maven Settings`
1. Set the comment to `Jenkins Maven Settings`
1. Check `Replace All`
1. Paste the contents of `estafet-microservices-scrum-newowner/maven-settings/jenkins-settings.xml` into
`Content` field. In the screen shot, all the comments in the file have been removed for clarity.
1. Click on `Submit`:

![Jenkins Global Maven configuration file](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_global_maven_config_file.png)

### <a name="install-jenkins-pipeline-maven-plugin"/> Install the Jenkins Pipeline Maven Plugin

The EMS demo `cicd` environment requires a particular version of the Jenkins Maven Pipeline Plugin: version `3.5.9`. To
download this version:

1. Download the plugin file
    ```
    $ cd /tmp
    $ curl -w "HTTP code is %{http_code}\n" -Ss -k -LJO \
    https://updates.jenkins.io/download/plugins/pipeline-maven/3.5.9/pipeline-maven.hpi
    HTTP code is 200
    $ cd -
    ```
    If the HTTP code is not `200`, the download failed.

2. Follow the steps in [Upload and install the plugin](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#upload-and-install-jenkins-plugin).
The driver `.hpi` file is `/tmp//pipeline-maven.hpi`.

### <a name="install-jenkins-cucumber-reports-plugin"/> Install the Jenkins Cucumber Reports Plugin

The Estafet Microservices Scrum `cicd` environment requires version 4.6.0 of the Cucumber Report Jenkins plugin.

1. Download the Cucumber Report Jenkins plugin file:

   ```
   [stevebrown@6r4nm12 estafet-microservices-scrum]$ cd /tmp
   [stevebrown@6r4nm12 tmp]$ curl -w "HTTP code is %{http_code}\n" -Ss -k -LJO https://updates.jenkins.io/download/plugins/cucumber-reports/4.6.0/cucumber-reports.hpi
   HTTP code is 200
   [stevebrown@6r4nm12 tmp]$
   ```

2. Follow the steps in [Upload and install the plugin](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#upload-and-install-jenkins-plugin).
The driver `.hpi` file is `/tmp/cucumber-reports.hpi`.

#### <a name="upload-and-install-jenkins-plugin"/>Upload and install a Jenkins plugin

2. From the Jenkins dropdown menu, choose `Manage Jenkins`, then `Manage Plugins`, then the `Advanced` tab:

![Jenkins Manage Plugins Advanced Tab](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_manage_plugins_advanced_tab.png)

1. Choose the driver `.hpi` file to upload
2. Choose `Upload`:

This screenshot shows the result of installing the Jenkins Pipeline Maven plugin:

![Jenkins Pipeline Maven Plugin Success](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_pipeline_maven_plugin_success.png)

### <a name="install-jenkins-http-request-plugin"/> Install the Jenkins HTTP Request Plugin

From the Jenkins dropdown menu, choose `Manage Jenkins`, then `Manage Plugins`, then the `Available` tab:

![Jenkins Manage Plugins Available Tab](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_manage_plugins_available_tab.png)

1. Enter `HTTP Request` in the `Filter` field
2. Check ``HTTP Request`` on thhe `Available` tab
3. Click on `Install without restart`

![Jenkins HTTP Request plugin installed](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_http_request_plugin_installed.png)

### <a name="create-github-credentials"/> Create GitHub credentials

From the Jenkins dropdown menu, choose `Credentials`:

![Jenkins Credentials page](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_credentials_page.png)

Click on the `global` dropdown list in the `Credentials` section. Then choose `Add credentials`:

![Jenkins Add Credential page](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_add_github_username_password.png)

Leave the `Kind` and `Scope` fields as shown.

1. Enter your GitHub username in the `Username` field.
1. Enter your GitHub password in the `Password` field.
1. Enter `microservices-scrum` in the `ID` field.
1. Enter `GitHub username and password` in the `Description` field.
1. Click on `OK` button:

![Jenkins GitHub Credential page](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_github_credentials_page.png)

### <a name="configure-github-webhooks"/> Configure GitHub Webhooks

Please see [WEBHOOKS.md](https://github.com/stericbro/estafet-microservices-scrum/blob/master/WEBHOOKS.md)

## Restart Jenkins.

To restart Jenkins, the URL is `https://jenkins-cicd.3.9.50.47.xip.io/restart` from [Get Jenkins Host](https://github.com/stericbro/estafet-microservices-scrum/blob/master/DEVOPS.md#get-jenkins-host).

Click on `OK` when prompted, then wait for Jenkins to complete restarting. You will then be asked to login again.

## Enable GitHub Webhooks For All Microservices

These are the build pipelines in Jenkins in the `cicd` environment:

| Build pipeline        | GitHub Repository URL                                                          |
|:----------------------|:-------------------------------------------------------------------------------|
| `ci-basic-ui`         | `https://github.com/newowner/estafet-microservices-scrum-basic-ui`             |
| `ci-project-api`      | `https://github.com/newowner/estafet-microservices-scrum-api-project-burndown` |
| `ci-project-burndown` | `https://github.com/newowner/estafet-microservices-scrum-api-sprint`           |
| `ci-sprint-api`       | `https://github.com/newowner/estafet-microservices-scrum-api-sprint-board`     |
| `ci-sprint-board`     | `https://github.com/newowner/estafet-microservices-scrum-api-sprint-burndown`  |
| `ci-sprint-burndowm`  | `https://github.com/newowner/estafet-microservices-scrum-api-story`            |
| `ci-story-api`        | `https://github.com/newowner/estafet-microservices-scrum-api-task`             |
| `ci-task-api`         | `https://github.com/newowner/estafet-microservices-scrum-api-discovery`        |
| `qa-pipeline`         | `https://github.com/newowner/estafet-microservices-scrum-api-gateway`          |

[WEBHOOKS.md](https://github.com/stericbro/estafet-microservices-scrum/blob/master/WEBHOOKS.md) shows how to configure
the `ci-basic-ui` build pipeline and the `estafet-microservices-scrum-basic-ui` GitHub repository.

Each of these GitHub repositories must be setup with a [GitHub Webhook](https://github.com/stericbro/estafet-microservices-scrum/blob/master/WEBHOOKS.md#configuring-github).

In Jenkins, each build pipeline must be [configured to use the GitHub WebHook](https://github.com/stericbro/estafet-microservices-scrum/blob/master/WEBHOOKS.md#configure-build-pipeline-to-use-webhook).

## <a name="release-pipeline-issues"/>Release Pipeline Issues

The first time a release pipeline is run, the following error will occur:

```
[cicd-release-basic-ui] Running shell script
+ oc start-build qa-pipeline -n cicd
build.build.openshift.io/qa-pipeline-2 started
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (increment version)
[Pipeline] readFile
Scripts not permitted to use new groovy.util.XmlSlurper. Administrators can decide whether to approve or reject this signature.
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException: Scripts not permitted to use new groovy.util.XmlSlurper
        at org.jenkinsci.plugins.scriptsecurity.sandbox.whitelists.StaticWhitelist.rejectNew(StaticWhitelist.java:271)
        at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor.onNewInstance(SandboxInterceptor.java:174)
        at org.kohsuke.groovy.sandbox.impl.Checker$3.call(Checker.java:200)
        at org.kohsuke.groovy.sandbox.impl.Checker.checkedConstructor(Checker.java:205)
        at com.cloudbees.groovy.cps.sandbox.SandboxInvoker.constructorCall(SandboxInvoker.java:21)
        at WorkflowScript.run(WorkflowScript:60)
        at ___cps.transform___(Native Method)

Some of the stacktrace has been omitted for brevity.

Finished: FAILURE
```

1. Click on `Administrators can decide whether to approve or reject this signature`:

   ![Jenkins approve in-process script class](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_approve_script_class.png)

1. Click on the `approve` button  to approve the `groovy.util.XmlSlurper` class:

   ![Jenkins in-process script class approved](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_script_class_approved.png)

1. Run the release pipeline again:

   The following error occurs:

   ```
   [Pipeline] // stage
   [Pipeline] stage
   [Pipeline] { (increment version)
   [Pipeline] readFile
   Scripts not permitted to use method groovy.util.XmlSlurper parseText java.lang.String. Administrators can decide whether to approve or reject this signature.
   [Pipeline] }
   [Pipeline] // stage
   [Pipeline] }
   [Pipeline] // node
   [Pipeline] End of Pipeline
   org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException: Scripts not permitted to use method groovy.util.XmlSlurper parseText java.lang.String
           at org.jenkinsci.plugins.scriptsecurity.sandbox.whitelists.StaticWhitelist.rejectMethod(StaticWhitelist.java:262)
           at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor.onMethodCall(SandboxInterceptor.java:161)
           at org.kohsuke.groovy.sandbox.impl.Checker$1.call(Checker.java:158)
           at org.kohsuke.groovy.sandbox.impl.Checker.checkedCall(Checker.java:162)
           at com.cloudbees.groovy.cps.sandbox.SandboxInvoker.methodCall(SandboxInvoker.java:17)
           at WorkflowScript.run(WorkflowScript:60)
           at ___cps.transform___(Native Method)

   Some of the stacktrace has been omitted for brevity.

   Finished: FAILURE
   ```
1. Click on `Administrators can decide whether to approve or reject this signature`

1. Click on the `approve` button  to approve the `groovy.util.XmlSlurper parseText java.lang.String` method:

   ![Jenkins in-process script method approved](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_script_method_approved.png)

1. Run the build again:

   ```
   build.build.openshift.io/qa-pipeline-6 started
   [Pipeline] }
   [Pipeline] // stage
   [Pipeline] stage (hide)
   [Pipeline] { (increment version)
   [Pipeline] readFile
   Scripts not permitted to use method groovy.lang.GroovyObject getProperty java.lang.String (groovy.util.slurpersupport.NodeChild.version). Administrators can decide whether to approve or reject this signature.
   [Pipeline] }
   [Pipeline] // stage
   [Pipeline] }
   [Pipeline] // node
   [Pipeline] End of Pipeline
   org.jenkinsci.plugins.scriptsecurity.sandbox.RejectedAccessException: Scripts not permitted to use method groovy.lang.GroovyObject getProperty java.lang.String (groovy.util.slurpersupport.NodeChild.version)
           at org.jenkinsci.plugins.scriptsecurity.sandbox.whitelists.StaticWhitelist.rejectMethod(StaticWhitelist.java:267)
           at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor$11.reject(SandboxInterceptor.java:358)
           at org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.SandboxInterceptor.onGetProperty(SandboxInterceptor.java:409)
           at org.kohsuke.groovy.sandbox.impl.Checker$6.call(Checker.java:291)
           at org.kohsuke.groovy.sandbox.impl.Checker.checkedGetProperty(Checker.java:295)
           at com.cloudbees.groovy.cps.sandbox.SandboxInvoker.getProperty(SandboxInvoker.java:29)
           at com.cloudbees.groovy.cps.impl.PropertyAccessBlock.rawGet(PropertyAccessBlock.java:20)
           at WorkflowScript.run(WorkflowScript:60)
           at ___cps.transform___(Native Method)

   Some of the stacktrace has been omitted for brevity.

   Finished: FAILURE
   ```
1. Approve the `groovy.lang.GroovyObject getProperty java.lang.String (groovy.util.slurpersupport.NodeChild.version)`
method, as above. The approved classes and methods should be:

   ![Jenkins approved script classes and methods](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_approved_script_classes_methods.png)

1. Run the build again - it should succeed.

   ![Jenkins successful release pipeline](https://github.com/stericbro/estafet-microservices-scrum/blob/master/md_images/devops/jenkins_successful_release_pipeline.png)
