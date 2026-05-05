---
title: "Detect Issues in Your Zabbix Instance Before It’s Too Late"
url: "https://blog.zabbix.com/detect-issues-in-your-zabbix-instance-before-its-too-late/32741/"
date: "Wed, 25 Mar 2026 08:42:31 +0000"
author: "Janis Eidaks"
feed_url: "https://blog.zabbix.com/feed/"
---
<img alt="" class="webfeedsFeaturedVisual wp-post-image" height="150" src="https://blog.zabbix.com/wp-content/uploads/2026/03/issue_detection_880x516-150x150.png" style="display: block; margin-bottom: 5px; clear: both;" width="150" /><p>In this blog post, I will show you how to detect performance issues in your Zabbix instance &#8211; in advance!</p>
<p><span id="more-32741"></span></p>
<p>You might be using Zabbix to monitor your infrastructure, devices, and applications, but are you also monitoring your own instance? It might seem unnecessary &#8211; after all, what’s there to monitor, right? Your instance just works, so everything is good. What else is needed?</p>
<p>Remember though, if your Zabbix database system runs out of disk space, data collection will come to a halt. If the data collectors are insufficient, the collected data will be inconsistent, and this will also affect the problem detection.</p>
<p>If you run out of cache space on your Zabbix server, depending on which cache is affected, your Zabbix server might crash immediately or experience degraded performance. A lot of things can go wrong, and you need to stay ahead of them! Here&#8217;s how.</p>
<h3 id="tune-your-database">Tune your database</h3>
<p>If you are using the default settings for your database, you are missing out on significant performance improvements that are just unused! Your actual instance performance is tied to the database performance. If the database performance is low, you will have a degraded Zabbix monitoring experience as well.</p>
<p>Do at least minimal fine-tuning, only change the settings you understand: read the documentation, check the official Zabbix blogposts, Zabbix community forum, and perform testing. Of course, you can use every tool at your disposal to make it work, such as AI, but always test the settings in the test environment.</p>
<p>The database tuning is a complex task. Initial parameters that you could tune for the MySQL DB are these:</p>
<pre>innodb_flush_log_at_trx_commit = 0
innodb_flush_method = O_DIRECT
optimizer_switch=index_condition_pushdown=off
innodb_buffer_pool_size= ~75% of RAM if only DB engine running or less if shared with other applications</pre>
<p>For a PostgreSQL database, you can use online tuner PGTUNE for initial configuration:</p>
<pre>https://pgtune.leopard.in.ua/</pre>
<h3 id="monitor-the-zabbix-database">Monitor the Zabbix database</h3>
<p>It is important to monitor your database. Zabbix offers several out-of-the-box options to monitor the most popular databases through different methods: either by Zabbix agent or Zabbix agent2, by ODBC checks, using Zabbix Java gateway or by HTTP checks. If an issue is detected, you will get a corresponding problem event. Don’t forget to manually update the old Zabbix templates to the current version after the Zabbix server upgrades.</p>
<figure class="wp-caption alignnone" id="attachment_32748" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_01-scaled.png"><img alt="" class="wp-image-32748 size-large" height="145" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_01-1024x232.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32748">Fig 1. Some of the available out-of-the-box templates for database monitoring</figcaption></figure>
<p>Of course, depending on the approach you have selected to monitor the database, you will need to do some additional steps for that to work. More information on how to configure it is available on the <a href="https://www.zabbix.com/integrations?cat=databases">Zabbix integration page.</a></p>
<figure class="wp-caption alignnone" id="attachment_32749" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_02.png"><img alt="" class="wp-image-32749 size-large" height="453" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_02-1024x725.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32749">Fig 2. Example of the configuration required to monitor the MySQL database with Zabbix agent2</figcaption></figure>
<h3 id="monitor-the-zabbix-server">Monitor the Zabbix server</h3>
<p>The next thing you should check is the Zabbix server host dashboards. In new instances, the Zabbix server host has already been included out of the box with two templates: Zabbix server health and Linux by Zabbix agent. If such a host has not been retained for some reason, now it’s time to create it and start monitoring your Zabbix server.</p>
<p>The Zabbix server health template uses Zabbix internal items that do not require any interface. The Linux by Zabbix agent template does require a running Zabbix agent on the Zabbix server system in order to gather the OS related metrics.</p>
<figure class="wp-caption alignnone" id="attachment_32750" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_03-scaled.png"><img alt="" class="wp-image-32750 size-large" height="97" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_03-1024x155.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32750">Fig 3. Zabbix server host with linked templates</figcaption></figure>
<h3 id="check-the-current-state-of-you">Check the current state of your Zabbix server</h3>
<p>Once you have such a host, go to the menu <strong>Monitoring &gt; Hosts</strong> and use the main filter to find the Zabbix server host and select its Host Dashboards.</p>
<figure class="wp-caption alignnone" id="attachment_32751" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_04-scaled.png"><img alt="" class="wp-image-32751 size-large" height="94" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_04-1024x150.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32751">Fig 4. Host dashboards</figcaption></figure>
<p>Select the Zabbix server health dashboard. Below, you will see the following pages under it &#8211; Performance, Processes, and Statuses.</p>
<figure class="wp-caption alignnone" id="attachment_32752" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_05-scaled.png"><img alt="" class="wp-image-32752 size-large" height="184" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_05-1024x294.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32752">Fig 5. Zabbix server health dashboard page: Performance</figcaption></figure>
<h4 id="check-the-cache-utilization">Check the cache utilization</h4>
<p>In the Performance page, you can see the usage of Zabbix server caches. You should make sure that all caches except the history cache are at least ~ 50 % free. Technically, you can make the caches as large as possible; at worst, they will just be under-utilised. So, adjust the cache sizes accordingly.</p>
<h4 id="consequences-of-running-out-of">Consequences of running out of configuration cache</h4>
<p>If you add a lot of hosts in an automated way and have a relatively small or default configuration cache size [configcache], you could fill this cache quickly. The consequences of it are:</p>
<ul>
<li>The Zabbix server will crash</li>
<li>The Zabbix server will be unable to start</li>
<li>The Zabbix server will not collect any data</li>
</ul>
<p>You will also see a warning message in the Zabbix frontend:</p>
<figure class="wp-caption alignnone" id="attachment_32753" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_06-scaled.png"><img alt="" class="wp-image-32753 size-large" height="230" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_06-1024x368.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32753">Fig 6. Zabbix server health dashboard page when running out of config cache</figcaption></figure>
<p>If the config cache does not fill up instantly, the problem event will be generated shortly after, and matching action operations will be executed while the Zabbix server is still running, for example, notifying admins about the issue. In the screenshot below, you can see that one action operation step was executed before the server crashed.</p>
<figure class="wp-caption alignnone" id="attachment_32754" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_07-scaled.png"><img alt="" class="wp-image-32754 size-large" height="136" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_07-1024x217.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32754">Fig 7. Generated problem event</figcaption></figure>
<p>When a Zabbix component is not working as expected, your best source of information is the log file, as it informs you about the issues. Here is the error message in the Zabbix Server log file below.</p>
<figure class="wp-caption alignnone" id="attachment_32755" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_08-scaled.png"><img alt="" class="wp-image-32755 size-large" height="104" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_08-1024x166.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32755">Fig 8. Zabbix server log file error: out of memory for config cache</figcaption></figure>
<p>The solution is very simple: just increase the configuration cache size (two times or more) and restart the Zabbix server. If you expect a significant increase in hosts in the near future, you can be more generous and allocate more memory. My current Zabbix server is monitoring approximately 400 hosts.</p>
<figure class="wp-caption alignnone" id="attachment_32756" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_09-scaled.png"><img alt="" class="wp-image-32756 size-large" height="261" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_09-1024x418.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32756">Fig 9. The system information of my Zabbix server</figcaption></figure>
<h4 id="consequences-of-running-out-of-0">Consequences of running out of value and history cache</h4>
<p>So, what happens if you run out of value cache? Zabbix server performance will degrade, and the frontend will become noticeably less responsive. Why is that? Value cache stores item values used for calculated items and evaluating triggers. Now, for each trigger calculation that does not contain an item metric in the history cache will be retrieved directly from the database.</p>
<figure class="wp-caption alignnone" id="attachment_32757" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_10.png"><img alt="" class="wp-image-32757 size-large" height="379" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_10-1024x606.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32757">Fig 10. Zabbix server health dashboard performance page for cache usage</figcaption></figure>
<p>The history cache stores historical data that will be written to the database. If it’s mostly full, it means you might have issues with your database performance – the Zabbix server is unable to write data fast enough to the database. This can trigger a cascading performance degradation with a negative feedback loop. In my case:</p>
<ul>
<li>A full value cache leads to additional DB read queries</li>
<li>DB performance drops, which leads to slow historical data writes to the database</li>
<li>The history cache also starts to fill up</li>
<li>The data collection is delayed due to the full history cache</li>
<li>As more data is collected, database read queries retrieve more data, progressively worsening the cycle</li>
</ul>
<p>Technically, it does not require your value cache to be 100% full to have this issue &#8211; if a lot of triggers use a long-time interval for evaluation, you could have a situation where your value cache is only 85% or 90% full, but the Zabbix server is unable to fit the required item history records in available memory.</p>
<p>The issue with running out of value cache will also be logged in the Zabbix server’s log file.</p>
<figure class="wp-caption alignnone" id="attachment_32758" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_11-scaled.png"><img alt="" class="wp-image-32758 size-large" height="213" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_11-1024x340.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32758">Fig 11. The Zabbix server log file with value cache error</figcaption></figure>
<p>The solution to this issue is simple: increase the value cache size and restart the Zabbix server.<br />
If you monitor your Zabbix server with the health template, problem events will be automatically generated when:</p>
<ul>
<li>Value cache works in a low memory mode</li>
<li>History cache utilization exceeds 75 %</li>
</ul>
<figure class="wp-caption alignnone" id="attachment_32759" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_12-scaled.png"><img alt="" class="wp-image-32759 size-large" height="49" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_12-1024x79.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32759">Fig 12. The generated problem events about the value cache issue</figcaption></figure>
<p>The issue with the Value cache working in low memory mode can also be seen in the graph below. Here you can see how many historical item values were present in cache, and how many had to be retrieved from the database directly.</p>
<figure class="wp-caption alignnone" id="attachment_32760" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_13.png"><img alt="" class="wp-image-32760 size-large" height="287" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_13-1024x459.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32760">Fig 13. Value cache effectiveness graph</figcaption></figure>
<p>Due to the terrible performance of the untuned database, when my history write cache fills up the data collectors are throttled, causing a pileup of delayed item collection.</p>
<figure class="wp-caption alignnone" id="attachment_32761" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_14.png"><img alt="" class="wp-image-32761 size-large" height="228" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_14-1024x364.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32761">Fig 14. Zabbix server performance graph</figcaption></figure>
<p>Slow database queries will appear in the Zabbix server log file.</p>
<figure class="wp-caption alignnone" id="attachment_32762" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_15-scaled.png"><img alt="" class="wp-image-32762 size-large" height="68" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_15-1024x108.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32762">Fig 15. The Zabbix server log file with slow query errors</figcaption></figure>
<h4 id="the-result-of-cache-tuning-and">The result of cache tuning and database tuning</h4>
<p>Increasing the value cache only partly solved one issue. After database tuning, database performance has improved significantly:</p>
<ul>
<li>The history cache is now empty</li>
<li>No more value cache misses</li>
<li>No more delayed items</li>
</ul>
<figure class="wp-caption alignnone" id="attachment_32763" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_16.png"><img alt="" class="wp-image-32763 size-large" height="631" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_16-1024x1010.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32763">Fig 16. The Cache usage, server performance, and value cache effectiveness graphs</figcaption></figure>
<p>After the Database tuning, the agent poller process and history syncer utilization also decreased to a low level.</p>
<figure class="wp-caption alignnone" id="attachment_32764" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_17.png"><img alt="" class="wp-image-32764 size-large" height="627" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_17-1024x1003.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32764">Fig 17. Data collector and internal process utilization graphs</figcaption></figure>
<h3 id="tune-the-zabbix-server-configu">Tune the Zabbix server configuration parameters</h3>
<p>Check the Processes page in the Zabbix Health dashboard and adjust the parameters accordingly. Only adjust the parameters that you understand. Changing the parameters arbitrarily can lead to the following:</p>
<ul>
<li>Wasted resources without effective performance improvement</li>
<li>Reduced Zabbix server performance</li>
<li>Zabbix server crashes</li>
</ul>
<p>For the data collectors, generally you require only a relatively small number of asynchronous data collectors, as they are very efficient, relatively larger number of synchronous data collectors. The graphs showing the utilisation of the gathering processes are extremely useful for determining which ones need to be increased &#8211; if they are close to 100% utilized, it’s now time for you to take action and add more.</p>
<h4 id="pitfall-of-misconfiguration">Pitfalls of misconfiguration</h4>
<p>Now, regarding the pitfalls of misconfiguration or lack of tuning. Here is a scenario: installed the Zabbix components, MySQL database without any configuration tuning, except the configuration cache to avoid the Zabbix server crashing immediately. The Zabbix server is monitoring around ~400 hosts. The Zabbix agent poller process and history syncers are utilized close to 100%, like in the Fig.17 before the tuning.</p>
<p>You might think that increasing both of these processes would improve the situation, for example, by doubling the count of them: more parallel agent processes should collect more data, and the more history syncers should write more data to the database.</p>
<p>After restarting the Zabbix server and checking the graph, both processes are close to 100% busy and the metric collection is significantly delayed. This is much worse.</p>
<figure class="wp-caption alignnone" id="attachment_32765" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_18.png"><img alt="" class="wp-image-32765 size-large" height="581" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_18-1024x929.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32765">Fig 18. Async data collector and internal process utilization graphs</figcaption></figure>
<p>By quadrupling both processes, the result is even worse, with significantly delayed item value collection.</p>
<figure class="wp-caption alignnone" id="attachment_32766" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_19.png"><img alt="" class="wp-image-32766 size-large" height="592" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Perf_issues_19-1024x947.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32766">Fig 19. Async data collector and internal process utilization graphs</figcaption></figure>
<p>So, what is happening behind the frontend? Just increasing the number of agent poller collectors and history syncers results in even worse performance. Seems counterintuitive, right: more data collectors should mean more data will be collected, and more history syncers – should allow more data to be written in parallel to the database.</p>
<p>However, increasing the data collector count in this specific situation will just make things much worse: you can collect more data at the same time, but will still face the same bottleneck: the database. Increasing the history syncers in this case makes the situation much worse, as more simultaneous queries to the database force it to slow down even further. So once again, tune your database engine and get more performance out of it.</p>
<h3 id="summary">Summary</h3>
<p>You should monitor all of your Zabbix components and react when issues occur. Also make sure that you receive the notifications in your preferred media type, so you can act immediately. The complete list of what to monitor is more extensive, but this blog post should provide you with some examples and inspiration. It is always a good idea to react proactively rather than deal with the issues after they occur.</p>
<p>&nbsp;</p>
<p>The post <a href="https://blog.zabbix.com/detect-issues-in-your-zabbix-instance-before-its-too-late/32741/">Detect Issues in Your Zabbix Instance Before It’s Too Late</a> appeared first on <a href="https://blog.zabbix.com">Zabbix Blog</a>.</p>
