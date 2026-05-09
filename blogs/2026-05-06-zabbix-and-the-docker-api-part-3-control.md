---
title: "Zabbix and the Docker API, Part 3: Control"
url: "https://blog.zabbix.com/zabbix-and-the-docker-api-part-3-control/32961/"
date: "2026-05-06"
author: "Janis Eidaks"
feed_url: "https://blog.zabbix.com/feed/"
---
In this blog post, you will learn how to add a simple container remote control capability to Zabbix in order to start, stop, or restart containers from within the discovered host. You might be wondering, why spend the effort to create a host for each template? Well, that’s because we define a manual script to control the container from within the Zabbix frontend. That’s neat, right? And why stop there? We can also implement a trigger action that automatically restarts the container if it crashes for any reason.
