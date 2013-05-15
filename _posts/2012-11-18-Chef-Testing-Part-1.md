---
layout: post
title: Chef Testing Part 1 with Jenkins, Vagrant among many others...
commentIssueId: 4
---

{{ page.title }}
================

<p class="meta">19 Nov 2012 - Los Angeles</p>

I know this post has been long-wanted but it's finally here. In this 5 part post series, I will show you how to roll-out your own Chef Testing framework with the following features.

#### Stage 1
<li> Ruby syntax, Cookbook Syntax, JSON and Lint checking via FoodCritic</li>
#### Stage 2
<li> Testing the upload of your cookbooks to a Chef on the Opscode hosted platform.</li>
#### Stage 3
<li> Ensure that Cookbooks are tested against any variety of platforms (e.g. Ubuntu, Fedora, CentOS and even Windows) via Vagrant,EC2 or RackSpace Cloud.</il>
#### Stage 4
<li>Minitest and ChefSpec's results are then gathered, evaluated and graphs generated.</li>

This setup will consist of 3 Jobs in Jenkins configured in a multi-job configuration. We will be using Vagrant in my examples. Below is a list of what you will need:

<li>1 Physical machine to run Jenkins and Vagrant(http://vagrantup.com/) using Virtualbox (http://virtualbox.org). (If your planning on using Vagrant for the Virtual machines creation, a physical machine is required) </il>
<li>Jenkins (https://jenkins-ci.org/)</li>
<li>Opscode Hosted Chef Account - Free Tier offers 5 nodes for free which is perfect for this post.</li>	

At the end of Part 5, I will provide a cookbook that setups the entire testing stack for you! :) Stay Tuned! - New post every week.