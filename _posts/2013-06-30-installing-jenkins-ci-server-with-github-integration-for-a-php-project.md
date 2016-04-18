---
title: Installing Jenkins CI server with GitHub integration for a PHP project
categories:
  - Programming
tags:
  - Continuous Integration
  - PHP
  - Testing
---

Here are the details how to install Jenkins CI server on a CentOS server (version 6.4 in this case) and set it up with GitHub integration so pushing to GitHub automatically triggers a build. Our project is a PHP project so the build will have PHP related stuff and we are going to use Ant as the build system.

<!--more-->

## Install Jenkins

First open [https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+RedHat+distributions](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+RedHat+distributions) for installation instructions, here are the relevant points in short;
Check java version by running:

```
$ java -version
```

If you have the OpenJDK version you are good to go (I did). If not, follow instructions in the page linked above to install a compatible Java runtime.

Then it's time to proceed with the Jenkins install. Just add the Jenkins repo and install it with yum by running these commands:

```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo 
$ sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key 
$ sudo yum install jenkins
```

And it's done. Then start Jenkins by running the command:

```
$ sudo service jenkins start
```

If you have a firewall that blocks access by default, you may need to open the port 8080 to be able to access Jenkins. We had iptables running so I just added a rule to allow 8080 and restarted iptables. Done.

Next you can check that Jenkins is running by pointing a browser to http://serverAddress:8080

If you are greeted by "Welcome to Jenkins!", everything is fine. If not, then I can't help you since I was lucky enough to get it right the first time, sorry.

## Securing Jenkins

As you may have noted, there was no login or anything, so we need to do some securing so that not everybody can mess with our precious CI setup. Navigate to Manage Jenkins in the left sidebar and then Configure Global Security. Check Enable security, but **do not save yet** (otherwise Jenkins will lock you out). Under Security Realm select your preferred authentication method, I selected "Jenkins's own user database" and then select the preferred authorization type, I selected "Logged-in users can do anything". If you also choose to use Jenkins's own user database, make sure to keep the "Allow users to sign up" checked for now. Then hit save.

Jenkins will now kick you out, so to speak, and present you with a login screen. If you followed my example and chose the same authentication method, you should see "Create an account" link. Follow that and create a new account. After you have successfully created a new account, you are automatically logged in. Then go back to "Configure Global Security" and disable "Allow users to sign up" to prevent anonymous users from creating accounts.

## GitHub integration

Next we will integrate Jenkins with GitHub to automatically trigger builds in Jenkins when new code has been pushed to GitHub or new pull requests are opened.

For this we will need to install a few plugins. Go to "Manage Jenkins" and then "Manage Plugins". Select the Available tab and scroll or filter to find a plugin called "GitHub Plugin" and check it, click "Install without restart". On the next page you will see the progress of the install, you can check the "Restart Jenkins when installation is complete and no jobs are running" so you will not have to restart it manually.

Next go to "Manage Jenkins" and then "Configure System". Halfway down the page, you will find Git plugin with a button "Git installations...". If you have git installed by yum etc. you will not have to change anything here. Otherwise install git and type the install dir here. At the bottom you can find a section called GitHub Web Hook. Select "Let Jenkins auto-manage hook URLs" and add your GitHub username (or create a specific user in GitHub for Jenkins) and API key. After that, hit Save.

Finally we need to configure a Service Hook in GitHub. Go to your projects settings page and click "Service Hooks". Scroll down until you find "Jenkins (GitHub plugin)" and click it. Add the url of your jenkins server followed by /github-webhook/ (i.e. http://serverAddress:8080/github-webhook/) and Save.

## Create a job for the project

In Jenkins you will need to create a job for something you wish to build periodically. The job represents our project in GitHub that we wish to build. Click "New Job" and fill in a job name (it's going to be easier if you use only alphanumeric characters because Jenkins will create a folder with this name in the filesystem) and then select free-style software project.

After creating a job, you will be presented with the job configuration view. You will need your GitHub repository URL. Paste the HTTPS URL to the "GitHub project" field at the top and then under "Source Code Management" select Git and paste the SSH URL into the text field. Under "Build Triggers" check "Build when a change is pushed to GitHub" and save.

Now the job has been created but as you might have noticed, Jenkins is still not able to access the repository, at least if it is private. You will need to set up SSH keys in the Jenkins home directory. My setup did not allow me to "su" as user jenkins, so I had to do this as root. I created a new key normally for the root user, saved it to GitHub (I used the per repository "Deploy Keys" option) and then connected to the repository from command line (so that a known_hosts file was created). Then I copied the key files and the known_hosts file to /var/lib/jenkins/.ssh and chown'd them to jenkins:jenkins. After that the job configuration page warning did not appear anymore and everything was working fine. Note. you must pay attention to the file permissions of the files inside .ssh folder (and the folder itself) when moving from somewhere else. If you are not familiar with SSH keys or GitHub "Deploy Keys" feature, here a few helpful links:

* [Generating SSH Keys](https://help.github.com/articles/generating-ssh-keys) at GitHub Help
* [Managing Deploy Keys](https://developer.github.com/guides/managing-deploy-keys/#deploy-keys) at GitHub Developer

## Create a build file

Now that we have a job for our project, we need to have that job do something, e.g. build. Jenkins supports Ant and Maven builds by default, as well as executing bare command line commands. We are going to use Ant, so first we need to install it. I followed the steps outlined in this page to get it installed: [http://xmodulo.com/2013/03/how-to-install-apache-ant-on-centos.html](http://xmodulo.com/2013/03/how-to-install-apache-ant-on-centos.html). After that go to Jenkins configuration "Manage Jenkins" and then "Configure System" and click the "Ant installations..." button in the Ant section. Add a new Ant and give it a name (I named mine "Default") and point it to the ANT_HOME directory and Save.

Next we need to create a build file for our project. What tasks you want to do in a build really depends on the project, but some obvious choices for a PHP project are things like execute unit tests, analyse code coverage of test, generate API documentation and run static code analysis (like PHP_CodeSniffer, PHP_Depend, PHP Mess Detector, PHP Copy/Paste Detector and PHPLOC). A great resource for PHP related Jenkins setup can be found in [http://jenkins-php.org/](http://jenkins-php.org/). They also have a sample build.xml file that has configuration for all of the above tasks. You can start with that and modify it to your liking and the requirements for your project. The build.xml file should be placed at the root folder of the project, and pushed to GitHub. You can of course play around with it and test different tasks before committing. Also make sure that all the tools you are going to use in the build are installed on the server. A good resource on these is The PHP Quality Assurance Toolchain at [http://phpqatools.org/](http://phpqatools.org/)

## Configure Jenkins to execute the build

After we have a working build.xml committed to GitHub, we need to configure Jenkins to execute the build. Open the "Configure" page under the job created previously. Scroll down to "Build" section and click "Add build step" and select "Invoke Ant". In the "Targets" input, type the build targets you wish to execute. This really depends on your build file, but in case of the sample build file it should be "build" (or "build-parallel").

Now when you push new changes to GitHub Jenkins will automatically download the new code and execute the build tasks. Way to go! If you get a blue dot beside the project name in the Jenkins home page, your build was successful. If you get a red dot, you were not so lucky and you got some fixing to do. Either some of your build tasks failed to run at all, or you tests didn't pass etc. You can check the console output of all the executed tasks by clicking the build and the "Console Output" on the left sidebar. And if  you would like to have a green dot on successful builds rather than a blue one, there is a plugin for that.

## Continuous Inspection of code quality

Now that we have Continuous Integration set up, I am going to focus on [installing Sonar for running Continuous Inspection of code quality and integrating it with Jenkins](http://cvuorinen.net/2013/07/installing-sonarqube-with-jenkins-integration-for-a-php-project/).
