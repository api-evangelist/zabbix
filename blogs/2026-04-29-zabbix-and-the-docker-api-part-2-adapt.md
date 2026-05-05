---
title: "Zabbix and the Docker API, Part 2: Adapt"
url: "https://blog.zabbix.com/zabbix-and-the-docker-api-part-2-adapt/32912/"
date: "Wed, 29 Apr 2026 07:26:15 +0000"
author: "Janis Eidaks"
feed_url: "https://blog.zabbix.com/feed/"
---
<img alt="" class="webfeedsFeaturedVisual wp-post-image" height="150" src="https://blog.zabbix.com/wp-content/uploads/2026/04/dockerpt1_880x516-150x150.png" style="display: block; margin-bottom: 5px; clear: both;" width="150" /><p>In this blog post, I will show you how to create a template for monitoring your Docker server with only API calls (without the Zabbix agent 2). Instead of creating a template, templated items, LLD rules, and trigger prototypes from scratch, we will adapt them from the existing template “Docker by Zabbix agent 2.”</p>
<h3 id="how-does-the-zabbix-agent-2-do">How does the Zabbix agent 2 do it?</h3>
<p>If you are wondering how the Zabbix agent 2 collects the data, you can look into the source code and see the magic behind the scenes: <a href="https://github.com/zabbix/zabbix/blob/master/src/go/plugins/docker/metrics.go">https://github.com/zabbix/zabbix/blob/master/src/go/plugins/docker/metrics.go</a>.</p>
<p>In short, it uses a Unix non-TCP socket, makes the Docker API requests on the host, and returns JSON objects. Hey, we already know how to use it ourselves from the previous blog post, right?</p>
<p>For the Zabbix agent 2 to work with the Docker template, it needs access to the Unix socket, either by adding the user Zabbix to the group: Docker or running the Zabbix-agent2 as root.</p>
<figure class="wp-caption alignnone" id="attachment_32926" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_1-scaled.png"><img alt="" class="wp-image-32926 size-large" height="205" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_1-1024x328.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32926">Fig 1. The Zabbix agent 2 Docker plugin source code</figcaption></figure>
<h3 id="how-we-will-do-it">How we will do it</h3>
<p>I can adapt this template, improvise whenever I encounter a non-existing metric, and overcome any technical challenge with effort! For the most part, in the template we have a few Zabbix agent items that collect data in bulk and a lot of dependent items (and dependent item prototypes from the LLD rules).</p>
<p>The path forward is quite straightforward &#8211; we will clone the template and replace the Zabbix agent item type with the HTTP agent type item and add additional parameters shown below. I will also add additional user macros on the template, including the Docker server IP address, port, CA, SSL certificate, and key file names so that these can be adjusted on the host level.</p>
<figure class="wp-caption alignnone" id="attachment_32923" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_2-scaled.png"><img alt="" class="wp-image-32923 size-large" height="153" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_2-1024x244.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32923">Fig 2. The workflow of the template modifications</figcaption></figure>
<h3 id="cloning-the-template-and-chang">Cloning the template and changing the item type</h3>
<p>First, clone the template &#8220;Docker by Zabbix agent 2&#8221; and give it a new name: &#8220;Docker stats by HTTP.&#8221; Next, modify the Templated Zabbix agent type item configuration with the following parameters:</p>
<figure class="wp-caption alignnone" id="attachment_32924" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_3-scaled.png"><img alt="" class="wp-image-32924 size-large" height="271" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_3-1024x433.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32924">Fig 3. The modification of the templated item configuration</figcaption></figure>
<p>Modify &#8220;Docker by HTTP&#8221; template items:</p>
<pre>● Modify item: Get containers
  &#x25aa; Type: HTTP agent
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/containers/json?all=true     
  &#x25aa; Type of inf: Text
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}</pre>
<pre>● Modify item: Get data_usage
  &#x25aa; Type: HTTP agent
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/system/df
  &#x25aa; Type of inf: Text
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}</pre>
<pre>● Modify item: Get images
  &#x25aa; Type: HTTP agent
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/images/json
  &#x25aa; Type of inf: Text
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}</pre>
<pre>●Modify item: Get info
  &#x25aa; Type: HTTP agent
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/info
  &#x25aa; Type of inf: Text
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}</pre>
<pre>● Modify item: Ping
  &#x25aa; Type HTTP agent
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/_ping
  &#x25aa; Type of inf: Text
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}
♯ Preprocessing (additional first step)
  &#x25aa; Boolean to Decimals</pre>
<p>We also need to modify the LLD rule configuration. Change item type from Zabbix agent type to HTTP agent type:</p>
<figure class="wp-caption alignnone" id="attachment_32925" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_4-scaled.png"><img alt="" class="wp-image-32925 size-large" height="201" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_4-1024x321.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32925">Fig 4. The modification of LLD discovery rule: containers discovery</figcaption></figure>
<pre>Modify LLD rule: Containers discovery
● Discovery rule
  &#x25aa; Type: HTTP agent
  &#x25aa; Key: docker.containers.discovery[true]
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/containers/json?all=true     
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}
  &#x25aa; Update interval: 1h
● LLD macros
  &#x25aa; {#ID}   $.Id
  &#x25aa; {#NAME} $.Names.first()</pre>
<pre>Modify LLD rule: Images discovery
● Discovery rule
  &#x25aa; Type: Dependent item
  &#x25aa; Master item: item&gt; Get images
● LLD macros
  &#x25aa; {#ID}   $.Id
  &#x25aa; {#NAME} $.RepoTags</pre>
<p>After that, we will also make changes in the LLD rule “Containers discovery” by modifying a few existing item prototypes (Zabbix agent type) and adding a new item. Below are the item prototypes that require modification:</p>
<pre>● In LLD rule Containers discovery, modify parameters for item prototype: Container {#NAME}: Get info
  &#x25aa; Type: HTTP agent
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/containers{#NAME}/json       
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}</pre>
<pre>● In LLD rule Containers discovery, modify item prototype: Container {#NAME}: Get stats
  &#x25aa; Type: HTTP agent
  &#x25aa; URL: https://{$DOCKER.IP}:{$DOCKER.PORT}/containers{#NAME}/stats?stream=false  
  &#x25aa; SSL verify peer: Checked
  &#x25aa; SSL verify host: Checked
  &#x25aa; SSL certificate file: {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file: {$SSL.KEY.FILE}
  &#x25aa; SSL key password: {$SSL.KEY.PASSWORD}</pre>
<pre>● In LLD rule Containers discovery, modify parameters for item prototype: Container {#NAME}: CPU percent usage
  &#x25aa; Type: Calculated
  &#x25aa; Formula: last(//docker.container_stats.cpu_usage.total.rate["{#NAME}"])/last(//docker.container_stats.system_cpu_usage.total.rate["{#NAME}"])*last(//docker.container_stats.online_cpus["{#NAME}"])*100
♯ Preprocessing (delete preprocessing step JSONPath)</pre>
<pre>● In LLD rule Containers discovery, add new item prototype: Container {#NAME}: System CPU total usage per second
  &#x25aa; Name: Container {#NAME}: System CPU total usage per second
  &#x25aa; Type: Dependent item
  &#x25aa; Key: docker.container_stats.system_cpu_usage.total.rate["{#NAME}"]
  &#x25aa; Type of information Numeric (float)
  &#x25aa; Master item    prototype &gt; Container {#NAME}: Get stats

&#x2666; Tags (name:value)        
  &#x25aa; component:cpu
  &#x25aa; container:{#NAME}      

♯ Preprocessing

  &#x25aa; JSONPath       $.cpu_stats.system_cpu_usage
  &#x25aa; Change per second
  &#x25aa; Custom multiplier: 1.0E-9</pre>
<h3 id="cloning-the-template-again-and">Cloning the template (again) and making minor modifications</h3>
<p>Next, I will clone the template &#8220;Docker statistics by HTTP&#8221; and give the copy a new name  &#8211; &#8220;Docker containers by HTTP.&#8221; In the template &#8220;Docker containers by HTTP,&#8221; delete the LLD rule Images discovery; delete templated items; from the LLD rule “Containers discovery” rule, delete all prototype entities (item prototypes), and add a filter in the LLD rule (shown below):</p>
<pre>In LLD rule Containers discovery rule, in Filter tab: add additional filter option
Filters [type of calculation: A and B and C]
   &#x25aa; {#ID} matches {HOST.HOST}</pre>
<p>I have also created a host group &#8220;Docker&#8221; where the discovered container hosts will be added. In the template &#8220;Docker statistics by HTTP&#8221; delete all item prototypes in the LLD rule &#8220;Containers discovery.&#8221; We will create a Host prototype in the LLD discovery rule &#8220;Containers discovery&#8221; &#8211; the parameters are shown below:</p>
<pre>Host prototype in LLD rule: Containers discovery
  &#x25aa; Host name:     {#ID}
  &#x25aa; Visible name:  {#NAME}
  &#x25aa; Templates:     Docker containers by HTTP</pre>
<figure class="wp-caption alignnone" id="attachment_32927" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_5-scaled.png"><img alt="" class="wp-image-32927 size-large" height="367" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_5-1024x587.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32927">Fig 5. Host prototype settings in LLD rule: Containers discovery</figcaption></figure>
<figure class="wp-caption alignnone" id="attachment_32928" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_6-scaled.png"><img alt="" class="wp-image-32928 size-large" height="95" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_6-1024x152.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32928">Fig 6. The cloned and modified templates</figcaption></figure>
<h3 id="creating-a-host-and-linking-th">Creating a host and linking the template</h3>
<p>Now all that is left is to create a host and link a template: &#8220;Docker statistics by HTTP.&#8221; Do not forget to add the correct Docker IP address or DNS name in the user macro.</p>
<p>I have created a new host &#8220;Docker server,&#8221; linked a template, and modified the user macro for the Docker IP address. This host will collect Docker overall statistics. After the LLD discovery execution, the container names will be automatically discovered and container hosts will be created with a linked template.</p>
<figure class="wp-caption alignnone" id="attachment_32929" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_7-scaled.png"><img alt="" class="wp-image-32929 size-large" height="197" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_7-1024x315.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32929">Fig 7. The Discovered container hosts with linked templates</figcaption></figure>
<p>If for some reason you are monitoring multiple Docker instances, you could have the same container names discovered, which will lead to LLD errors, as there can’t be hosts with the same name (container ID) or visible name (container name). Quick solution &#8211; for each Docker instance, add a prefix to each container name. Another solution &#8211; don’t split the template into two parts, then the items will be discovered under the same host, and you will not have this issue.</p>
<p>The Docker server host shows general information about the Docker server&#8217;s overall state and status:</p>
<figure class="wp-caption alignnone" id="attachment_32930" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_8-scaled.png"><img alt="" class="wp-image-32930 size-large" height="316" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_8-1024x505.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32930">Fig 8. Docker server hosts the latest data</figcaption></figure>
<p>A host will be created automatically for each discovered container and will collect the container-specific performance metrics:</p>
<figure class="wp-caption alignnone" id="attachment_32931" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_9-scaled.png"><img alt="" class="wp-image-32931 size-large" height="314" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI2_9-1024x503.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32931">Fig 9. The container: zabbix-agent2 latest data</figcaption></figure>
<h3 id="summary">Summary</h3>
<p>Now you and I know a little bit more about how Zabbix agent2 is collecting Docker metrics. This blog post has shown you how to improvise and adapt existing templates with different data collection methods. Zabbix is a very versatile tool that you can use in multiple ways to get the data if you have some technical constraints. The included template can also be used as is, or you can modify it to suit your needs.</p>
<p>The post <a href="https://blog.zabbix.com/zabbix-and-the-docker-api-part-2-adapt/32912/">Zabbix and the Docker API, Part 2: Adapt</a> appeared first on <a href="https://blog.zabbix.com">Zabbix Blog</a>.</p>
