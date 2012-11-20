---
layout: post
title: Chef Testing Part 1 with Jenkins, Vagrant among many others...
---

{{ page.title }}
================

<p class="meta">19 Nov 2012 - Los Angeles</p>

Hey Everyone!

I know this post has been long-waited but it's finally coming. In this 5 Part post, I will show you how to roll-out your own Chef Testing framework with the following features:

<h3>Stage 1</h3>
<li> Ruby, Cookbook Syntax, JSON and Lint checking via FoodCritic</li>
<h3>Stage 2</h3>
<li> Testing the upload of your cookbooks to a Chef testing Org on the Opscode Hosted platform</li>
<h3>Stage 3</h3>
<li> Ensure that Cookbooks are tested against any variety of platforms (e.g. Ubuntu, Fedora, CentOS and even Windows) via Vagrant,EC2 or RackSpace Cloud for every role that exists in the roles directory of your Chef repo.</il>
<h3>Stage 4</h3>
<li>Minitests and ChefSpec's results are then gathered, evaluated and graphs generated.</li>

This setup will consist of 3 Jobs in Jenkins configured in a multi-job configuration. We will be using Vagrant in my examples. Below is a list of what you will need:

<li>1 Physical machine to run Jenkins and Vagrant(http://vagrantup.com/) using Virtualbox (http://virtualbox.org). (If your planning on using Vagrant for the Virtual machines creation, a physical machine is required) </il>
<li>Jenkins (https://jenkins-ci.org/)</li>
<li>Opscode Hosted Chef Account - Free Tier offers 5 nodes for free which is perfect for this post.</li>	

At the end of Part 5, I will provide a cookbook that setups the entire testing stack for you! :)

