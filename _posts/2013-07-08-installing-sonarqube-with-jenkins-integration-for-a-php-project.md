---
title: Installing SonarQube with Jenkins integration for a PHP project
categories:
  - Web Development
tags:
  - Continuous Integration
  - PHP
  - Sonar
  - SonarQube
---

In this second part of my Continous Integration setup I will detail the steps required to install SonarQube (previously called just Sonar, renamed to SonarQube with 3.6 release just a few days ago) and integrate it with the Jenkins server from the [previous post](http://cvuorinen.net/2013/06/installing-jenkins-ci-server-with-github-integration-for-a-php-project/) so SonarQube will run a daily analysis of our PHP project. In the previous post I covered the installation of Jenkins on a CentOS server and integrated it with GitHub, so if you do not have Jenkins set up you might want to start [there](http://cvuorinen.net/2013/06/installing-jenkins-ci-server-with-github-integration-for-a-php-project/).

So why do we need Sonar if Jenkins is already setup with all the static code analysis reports as in the "[Template for Jenkins Jobs for PHP Projects](http://jenkins-php.org/)" ? Jenkins is great for running builds and it can be setup in a way that it generates useful reports about the source code, but Sonar has been specifically designed for this. It has a great user interface and gives so much better statistics. You can really easily see what direction the project is going and how much technical debt you are accumulating. You can drill down to different parts of the project and compare them, you can see how the code base has changed over time and compare different versions etc. This is especially useful with large projects that have a long lifespan so you can better keep track of code quality. You can also take action on the issues Sonar has found by assigning them to people. All the dashboards are customizable with loads of different widgets and all the widgets are also customizable so you can really make it your own and make the data that you care about the most easily accessible. SonarQube really takes the "static" out of static code analysis and makes it dynamic and interactive. You have to see it in action to really experience it, so I suggest you either watch a [screencast](http://www.sonarqube.org/sonar-tv-a-short-video-for-every-key-feature/) or click around in the [live demo](http://nemo.sonarsource.org/).

<!--more-->

## Installing SonarQube

First, install SonarQube using yum by running these commands:

```
$ sudo wget -O /etc/yum.repos.d/sonar.repo http://downloads.sourceforge.net/project/sonar-pkg/rpm/sonar.repo
$ yum install sonar
```

Although this post focuses on CentOS installation, you can find [packages for most popular Linux distributions](http://sonar-pkg.sourceforge.net/).

Next we have to edit the Sonar config file */opt/sonar/conf/sonar.properties* to change the db to MySQL. You can of course use whatever database you like, but I had MySQL already installed on the machine so it was an obvious choice. Just comment out the line about the embedded H2 database (which is not intended for production use) and uncomment the MySQL line, save and exit. After that you also have to create an empty database called sonar and create a user named sonar that has access to the database.

This can be done by running the following commands on mysql:

```sql
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8generalci; 
CREATE USER 'sonar' IDENTIFIED BY 'sonar';
GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';
GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
FLUSH PRIVILEGES;
```

I also had to change the default port of Sonar since I had Nginx with PHP running on the same server and php-fpm was already using port 9000 (which I discovered after Sonar refused to start and I had to take a look at the Sonar log file in */opt/sonar/logs/sonar.log*). So I changed Sonar to use port 9090 (in the Sonar config file).

After that you can go ahead and start Sonar using the following command:

```
$ sudo service sonar start
```

Note, do not use the script in /opt/sonar/bin... to start Sonar (instructed in Sonar documentation) as root user because you will run into file permission issues later since the init.d script runs it as the user "sonar" (I had to learn this the hard way, so just trying to save you the trouble).

If you have a firewall that blocks access by default, you may need to open the port 9000 (or 9090 if you had to change the port like me) to be able to access Sonar from outside. We had iptables running so I just added a rule to allow 9090 and restarted iptables.

If everything went fine, SonarQube should be running and we can point a browser to: http://serverAddress:9000 (or http://serverAddress:9090 in my case).

## Securing Sonar

You can log in with the default account "admin" with password "admin". The first thing you should do is of course change the password on the page under "Administrator" > "My Account". After that you can disable access from anonymous users in the "Settings" > "Configuration" > "Security" page by selecting True in the "Force user authentication" setting.

## Installing SonarQube Runner

Installation instructions can be found here: [http://docs.codehaus.org/display/SONAR/Installing+and+Configuring+SonarQube+Runner](http://docs.codehaus.org/display/SONAR/Installing+and+Configuring+SonarQube+Runner)

First download and uncompress the file (check the above link for the latest version) and create a symbolic link to this specific version by executing the following commands:

```
$ cd /opt
$ sudo wget http://repo1.maven.org/maven2/org/codehaus/sonar/runner/sonar-runner-dist/2.2.2/sonar-runner-dist-2.2.2.zip
$ sudo unzip sonar-runner-dist-2.2.2.zip
$ sudo ln -s sonar-runner-2.2.2 sonar-runner
```

Next we need to edit the configuration file and update database information and server URL. In our case the file is */opt/sonar-runner/conf/sonar-runner.properties* and you need to set the correct sonar.host.url, uncomment the line for MySQL as the database, uncomment the database username and password. You will also have to uncomment and update the login details if you selected the "Force user authentication" setting in the previous section.

Next we need to create a new environment variable called SONAR_RUNNER_HOME and add the sonar runner bin directory to the PATH environment variable, this can be done with the following command:

```
$ sudo echo -e '#!/bin/bash\nexport SONAR_RUNNER_HOME=/opt/sonar-runner\nexport PATH=$PATH:$SONAR_RUNNER_HOME/bin' > /etc/profile.d/sonar-runner.sh
```

And if you don't want to log out and back in again for this to take effect, you can run these commands to add the environment variables for the current session:

```
$ export SONAR_RUNNER_HOME=/opt/sonar-runner
$ export PATH=$PATH:$SONAR_RUNNER_HOME/bin
```

Now the runner has been installed, and you can test it by executing this command:

```
$ sonar-runner -v
```

If everything worked correctly, you should receive the Sonar Runner version number as output.

## Installing PHP environment for SonarQube

SonarQube relies on some external tools for PHP analysis, so we need to make sure everything is installed on our system and also install the PHP plugin for Sonar. SonarQube requires PHP Depend, PHPMD, PHP_CodeSniffer and PHPUnit which you already might have installed on your system if you followed my previous post and installed these for the build tasks. If not, check [http://phpqatools.org/](http://phpqatools.org/) for a quick install of everything in one command. SonarQube also requires Xdebug, so we will need to install that with the following command:

```
$ yum install php-pecl-xdebug
```

Next log into SonarQube and head to "Settings" > "System" > "Update Center". Click "Available Plugins" and click "PHP" in the list of available languages and then click Install. After that we need to restart Sonar by executing the following command:

```
$ sudo service sonar restart
```

## Integrating SonarQube with Jenkins

Now that we have SonarQube all set up, it's time to integrate code analysis with our Continuous Integration set up from the [previous post](http://cvuorinen.net/2013/06/installing-jenkins-ci-server-with-github-integration-for-a-php-project/). For this we need to go back to Jenkins and install the SonarQube Jenkins Plugin. Log into Jenkins as administrator and go to "Manage Jenkins" > "Manage Plugins", click the "Available" tab and type "sonar" to the filter input in the top right, select Jenkins Sonar Plugin and install it and restart Jenkins.

Then we need to tell Jenkins where our Sonar installation is, so head on to "Manage Jenkins" > "Configure System" and find the "Sonar Runner" section and click "Add Sonar Runner". Give your Sonar Runner a name (I called mine "Default") and point it to the SONAR_RUNNER_HOME directory (*/opt/sonar-runner*). Then find the "Sonar" section and click "Add Sonar". Give your Sonar installation a name (I called mine "SonarQube"), click "Advanced" and fill in your server URL, login account and database details. Finally hit Save.

Next create a file called *sonar-project.properties* in the root directory the project you want to analyse with SonarQube. This file is needed to configure the SonarQube Runner. You can find an example file for a PHP project here: [https://github.com/SonarSource/sonar-examples/blob/master/projects/languages/php/php-sonar-runner-unit-tests/sonar-project.properties](https://github.com/SonarSource/sonar-examples/blob/master/projects/languages/php/php-sonar-runner-unit-tests/sonar-project.properties). Set the properties according to your project and save and commit.

Now we need to add a build step to run Sonar analysis on each build. Open the project in Jenkins and click "Configure" on the left sidebar. Scroll down to the "Build" section and click "Add build step" and select "Invoke Standalone Sonar Analysis". If you used the default name and location for the *sonar-project.properties* file, you will not have to specify it's path. In case you selected the "Force user authentication" setting when securing Sonar, you will have to provide login credentials for the runner. This can be done in the "Project properties" text input field by setting the sonar.login and sonar.password properties (not sure if this is really required since they are already in the Sonar runner configuration file, but I added them anyway). Then Save.

Now everything is set up and you can try it out by running a build.

## Optimizing and finishing up

Since Jenkins was already running all the static code analysis with PHPMD, PHP Depend and PHP_CodeSniffer etc. in my Ant build, there really is no need for Sonar to run the same stuff again. Luckily there are properties that you can set in the *sonar-project.properties* file to disable generating the reports again and only analyze existing reports. More details can be found here: [http://docs.codehaus.org/display/SONAR/PHP+Plugin#PHPPlugin-Reusingexistingreports](http://docs.codehaus.org/display/SONAR/PHP+Plugin#PHPPlugin-Reusingexistingreports)

Another thing is that if you have the *failonerror* parameter in the Ant *build.xml* file set to "true" for unit tests, Jenkins will not run Sonar at all if any tests fail, since the build already failed in the first step. So I removed the attribute because it really is not needed, at least not in my case since I have the "XUnit test results report" Post-build action configured (from the [Template for Jenkins Jobs for PHP Projects](http://jenkins-php.org/)) and it will mark the build as failed when there are failed unit tests, but the good thing in this case is that it will do so only after all the build steps have been executed, allowing Sonar runner to run as well.
