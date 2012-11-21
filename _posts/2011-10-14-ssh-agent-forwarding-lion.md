---
layout: post
title: Enable SSH Agent (key) forwarding on Mac OS Lion
commentIssueId: 3
---

{{ page.title }}
================

<p class="meta">14 oct 2011 - Los Angeles</p>

It looks like in some occasions ssh agent (key) forwarding doesn’t work anymore after upgrading to Mac OS Lion.

Gladly the solution is easy, you just need to run the following command in your Terminal and reboot after that.

**ssh-add**

This adds your identity back to the authentication agent again.