---
layout: post
title: Chef Testing Part 2 with Jenkins, Vagrant among many others...
commentIssueId: 5
---
{{ page.title }}
================

<p class="meta">9 Dec 2012</p>

## Overview

**In part 1 of Chef Testing, we went over the prereqs of what is needed to get basic testing inplace. In this post we'll setup Our first jenkins job and setup food critic. **

Your Jenkins Instance will need Ruby or RVM(Recommended) (http://rvm.io) installed and Don't forget to configure the following git options as the Jenkins user


git config --global user.email "you@example.com"

git config --global user.name "Your Name"


## Step 1 - Setting up the Foundation!

** Let's begin by getting jenkins installed! **

Head over to <http://jenkins-ci.org/> and download the installer for your platform, I'm using the Mac OS X installer for this example. If your setting up on a Mac, here is the installation doc that I'm following: <http://goodliffe.blogspot.com/2011/09/how-to-set-up-jenkins-ci-on-mac.html>.

Now, let's get the plugins and parsers that we need installed and setup. 

Head over to the plugin manager and install the following plugins, For Jenkins installed on a Mac OS X, visit <http://localhost:8080/pluginManager/available>. 

#### Jenkins Plugins

1. MultiJob Plugin - <http://wiki.jenkins-ci.org/display/JENKINS/Multijob+Plugin>
2. Dashboard View - <http://wiki.jenkins-ci.org/display/JENKINS/Dashboard+View>
3. Warning Plugin - <http://wiki.jenkins-ci.org/x/G4CGAQ>
4. Git Plugin - <http://wiki.jenkins-ci.org/display/JENKINS/Git+Plugin>
5. Github Plugin - <http://wiki.jenkins-ci.org/display/JENKINS/Github+Plugin>
6. Jenkins Ruby Metrics - <http://wiki.jenkins-ci.org/display/JENKINS/Ruby+Metrics+Plugin>+Plugin>
7. Build Pipeline Plugin - <https://code.google.com/p/build-pipeline-plugin>
8. Jenkins Adaptive Plugin - <http://wiki.jenkins-ci.org/display/JENKINS/Jenkins+Adaptive+Plugin>

## Step 2 - Compile Warnings 
For Jenkins Mac, visit <http://localhost:8080/configure>


Next… Head over to the configure screen so we can add some parser warnings. Scroll down till you see **Compiler Warnings**.

##### FoodCritic - More Info <http://acrmp.github.com/foodcritic/#ci>
*Enter the field as follows:*

Name -  Foodcritic

Link name - Foodcritic recipe check results

Trend Report Name - Foodcritic Warnings

Regular Expression - `^(FC[0-9]+): (.*): ([^:]+):([0-9]+)$`

Mapping Script

```
import hudson.plugins.warnings.parser.Warning

String fileName = matcher.group(3)
String lineNumber = matcher.group(4)
String category = matcher.group(1)
String message = matcher.group(2)

return new Warning(fileName, Integer.parseInt(lineNumber), "Foodcritic Warning", category, message);
```
Example Log Message 

```FC001: Use strings in preference to symbols to access node attributes: ./recipes/innostore.rb:30``` 

You should get something like below as your output.

```
One warning found
file name: ./recipes/innostore.rb
line number: 30
priority: Normal Priority
category:  FC001
type: Foodcritic Warning
message: Use strings in prefe[...]ols to access node attributes

```
Add another Compiler warning #2 

#### Ruby Syntax Warning 

Name - Ruby Syntax Warnings

Link Name - Ruby syntax check results

Trend Report Name - Ruby syntax warnings

Regular Expression - `^Checking syntax of .+\n(.+):(\d+):(.*)$`

Mapping Script -

```
import hudson.plugins.warnings.parser.Warning

String fileName = matcher.group(1)
String lineNumber = matcher.group(2)
String category = "warn"
String message = matcher.group(3)

return new Warning(fileName, Integer.parseInt(lineNumber), "Ruby Syntax Warning", category, message);

```

Example Log Message - 

```
Checking syntax of ./cookbooks/ntp/recipes/default.rb
./cookbooks/ntp/recipes/default.rb:61: syntax error, unexpected ')'

```

Expected Output:

```
One warning found
file name: ./cookbooks/cobbler/recipes/default.rb
line number: 61
priority: Normal Priority
category:  warn
type: Ruby Syntax Warning
message: syntax error, unexpected ')'

```

## Step 3 - Validate Repository Job


Let's get the first job setup, the first job will clone our Chef repo from github and perform a ruby syntax check, foodcritic cookbook check. The Public key for our jenkins user will have to be added to your github ssh keys in your Github profile so it has access to our github repo. 

On linux systems, you can find Jenkins public key in `/var/lib/jenkins/.ssh` directory. If your using the Mac OS X installer, by default the Jenkins home directory is `/Users/Shared/Jenkins/Home`. Keep in mind that this directory is owned by Jenkins user and group Jenkins.


1. Create a new Job or visit <http://localhost:8080/view/All/newJob> if your using Jenkins on Max OS X as I am in this series. 

2. Add a name for the job, I named mine "Validate Chef Repo" and select **Build a free-style software project**

3. For Source Code Management, select Git. For this example, I'm using <https://github.com/opscode/chef-repo-workshop-sysadmin.git>
4. For Branch, I'm using "master", but you may use any branch you like. 
5. For Build Triggers, I selected "Poll SCM". Add your schedule, I used `*/5 * * * *`
6. Now lets add our three build Execute shell's. 

##### Ruby Syntax Check - Execute Shell 1:
 <script src="https://gist.github.com/4242705.js?file=gistfile1.txt"></script>

##### Vagrant Check - Execute Shell 2:
Execute Shell 2: (This Step requires a basic Vagrantfile in your Chef Repo, Feel free to ship this Execute Shell, we will visit Vagrant in a later post.)
<script src="https://gist.github.com/4246285.js?file=gistfile1.txt"></script>

##### JSON Check - Execute Shell 3:
Execute Shell 3: 
<script src="https://gist.github.com/4252209.js?file=gistfile1.txt"></script>


#### We are done with our first job!

This completes the second post of this series, please be aware that each jenkins setup can different and that all of these steps may not work in the same fashion. If you encounter any issues following this post, please post a comment and I'll try my best to assist you. Please stay tuned for next week's update!.




