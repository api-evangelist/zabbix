---
title: "Zabbix and the Docker API, Part 1: Inspect"
url: "https://blog.zabbix.com/zabbix-and-the-docker-api-part-1-inspect/32860/"
date: "Wed, 22 Apr 2026 08:04:04 +0000"
author: "Janis Eidaks"
feed_url: "https://blog.zabbix.com/feed/"
---
<img alt="" class="webfeedsFeaturedVisual wp-post-image" height="150" src="https://blog.zabbix.com/wp-content/uploads/2026/04/dockerpt1_880x516-150x150.png" style="display: block; margin-bottom: 5px; clear: both;" width="150" /><p>In this blog post, I will show you how to configure Zabbix to securely gather Docker API metrics using the Zabbix HTTP agent item with certificate authentication. This guide will cover configuring the Docker API and the Zabbix server side to gather data more securely.</p>
<h3 id="getting-the-data-to-zabbix-fro">Getting the data to Zabbix from the Docker API</h3>
<p>By default, Docker API uses a non-network socket for security reasons, and there are several valid reasons for this. It is not advised to expose your Docker environment over TCP to localhost, and even less to the internet. Exposing the Docker API without any security to the internet is just inviting hackers for free lunch, as anyone (bots included) who can access your Docker API will also be able to do malicious operations with it (make changes, launch malicious containers, try to take over your environment, and do a lot of harm in general) !</p>
<p>So, make sure to harden your environment&#8217;s security and use this guide at your own judgment. Also, set up your firewall so only the Zabbix server can access the Docker API port! By default, you can check if the Docker service is active and if you can get a response to the Docker API by running the curl command:</p>
<pre># systemctl is-active docker
# curl --silent --show-error --unix-socket /var/run/docker.sock http://localhost/info |jq
</pre>
<figure class="wp-caption alignnone" id="attachment_32862" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_1-scaled.png"><img alt="" class="wp-image-32862 size-large" height="224" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_1-1024x359.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32862">Fig 1. Output of the Docker API call in CLI</figcaption></figure>
<h3 id="generating-the-certificates-fo">Generating the certificates for the Docker and Zabbix server</h3>
<p>You can use the right tool for the job, such as an HTTP agent for the Docker API requests with proper certificate authentication. You will require the CA private key and CA certificate; private key and certificate for the Docker server; private key and certificate for the Zabbix server (for simplicity, you can generate all of them on the Docker server and copy the appropriate files to the Docker server and Zabbix server directories).</p>
<p>A guide you can follow to generate the certificate files is located here: <a href="https://docs.docker.com/engine/security/protect-access/#use-tls-https-to-protect-the-docker-daemon-socket">https://docs.docker.com/engine/security/protect-access/#use-tls-https-to-protect-the-docker-daemon-socket.</a></p>
<h4 id="deploying-the-certificate-file">Deploying the certificate files and configuring the services</h4>
<p>On the Docker server, copy the CA and server certificate files to /etc/docker directory:</p>
<pre># cp -v {ca,server-cert,server-key}.pem /etc/docker</pre>
<p>The Docker daemon also requires JSON configuration with additional settings (allow TCP/ Unix socket, TLS options):</p>
<pre># nano /etc/docker/daemon.json</pre>
<pre>{
  "hosts": ["tcp://0.0.0.0:2376","unix:///var/run/docker.sock"],
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/etc/docker/ca.pem",
  "tlscert": "/etc/docker/server-cert.pem",
  "tlskey": "/etc/docker/server-key.pem"
}
</pre>
<p>We will have to add the Docker service override to remove the Unix socket from the Docker systemd service, then reload the daemon, and restart the Docker service.</p>
<pre># mkdir -p /etc/systemd/system/docker.service.d</pre>
<pre># nano /etc/systemd/system/docker.service.d/override.conf</pre>
<pre>[Service]
ExecStart=
ExecStart=/usr/bin/dockerd</pre>
<pre># systemctl daemon-reload
# systemctl restart docker</pre>
<p>On the Zabbix server side, create directories for certificate files. Then, copy the relevant certificate files from the Docker server. In my case, I generated all of the certificate files on the Docker host (replace docker in the scp command with IP/DNS name of the Docker server): ca.pem, client-cert.pem, client-key.pem, to their respective directories and change their permissions.</p>
<pre># mkdir -pv /etc/zabbix/ssl/{ca,certs,keys}
# scp root@docker:/root/dockercerts/ca.pem /etc/zabbix/ssl/ca/
# scp root@docker:/root/dockercerts/client-cert.pem /etc/zabbix/ssl/certs/
# scp root@docker:/root/dockercerts/client-key.pem /etc/zabbix/ssl/keys/
# chmod -v 0400 /etc/zabbix/ssl/keys/client-key.pem
# chmod -v 0444 /etc/zabbix/ssl/ca/ca.pem /etc/zabbix/ssl/certs/client-cert.pem
# chown zabbix:zabbix -R /etc/zabbix/ssl</pre>
<p>Check if you can get data in the Zabbix server from the Docker server with HTTPS request (replace $HOST with your Docker server address):</p>
<pre># curl -sS https://$HOST:2376/info \
  --cert /etc/zabbix/ssl/certs/client-cert.pem \
  --key /etc/zabbix/ssl/keys/client-key.pem \
  --cacert /etc/zabbix/ssl/ca/ca.pem |jq</pre>
<figure class="wp-caption alignnone" id="attachment_32863" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_2-scaled.png"><img alt="" class="wp-image-32863 size-large" height="219" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_2-1024x351.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32863">Fig 2. Executing the HTTPS request to the Docker server from the Zabbix server machine</figcaption></figure>
<p>If everything works so far, then it is time to modify the Zabbix server configuration file and specify the location of the certificate file directories. After that, restart the Zabbix server service.</p>
<pre># nano /etc/zabbix/zabbix_server.conf</pre>
<pre>SSLCertLocation=/etc/zabbix/ssl/certs
SSLKeyLocation=/etc/zabbix/ssl/keys
SSLCALocation=/etc/zabbix/ssl/ca</pre>
<pre># systemctl restart zabbix-server</pre>
<p>You will also need to copy the Docker CA file to the trusted CA directory and update the CA list.</p>
<pre># cd /
# cp /etc/zabbix/ssl/ca/ca.pem /etc/pki/ca-trust/source/anchors/
# update-ca-trust extract</pre>
<figure class="wp-caption alignnone" id="attachment_32864" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_3-scaled.png"><img alt="" class="wp-image-32864 size-large" height="126" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_3-1024x201.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32864">Fig 3. The location of certificate files in the directories for each server</figcaption></figure>
<h3 id="configuring-the-monitoring-in-">Configuring the monitoring in the Zabbix frontend</h3>
<p>If you have read this far and decided that this is too much work or this approach is not feasible in your environment (company policy or some other technical limitation), don’t be discouraged so fast! There is another way to get the metrics without changing the Docker configuration, creating certificates, and configuring the Zabbix server config file &#8211; simply use an SSH agent-type item to gather the data.</p>
<p>To prepare for both approaches, I will create a host with multiple user macros, which will store the IP address, port, SSH user, SSH password, and SSL certificate information.</p>
<figure class="wp-caption alignnone" id="attachment_32865" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_4-scaled.png"><img alt="" class="wp-image-32865 size-large" height="314" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_4-1024x502.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32865">Fig 4. Creating new host</figcaption></figure>
<figure class="wp-caption alignnone" id="attachment_32866" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_5-scaled.png"><img alt="" class="wp-image-32866 size-large" height="314" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_5-1024x502.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32866">Fig 5. Adding user macros to the host</figcaption></figure>
<h4 id="the-easy-way:-ssh-agent">The easy way: SSH agent items</h4>
<p>However, what to do if the company policy prohibits installing additional applications to gather data, such as the Zabbix agent (or changing Docker configuration settings, as in this case)? In this instance, you can use other, seemingly simpler ways to gather metrics, such as using the SSH agent item.</p>
<p>If the only tool you have is a hammer (SSH access), you tend to see every problem as a nail. The old adage “do not fix what is not broken” is still prevalent in this era! In that case, create an SSH agent-type item. Specify the IP address and SSH port in the item key, the username and password for the Docker host, and specify a command to gather the data. For those fields, I will use the previously defined user macros.</p>
<p>Here is an example of the SSH item configuration:</p>
<pre>Host: Docker server items 
Item #1
  &#x25aa; Name:          Get info ssh
  &#x25aa; Type           SSH agent
  &#x25aa; Key:           ssh.run[docker.infos,{$DOCKER.IP}]  
  &#x25aa; Type of inf:   text
  &#x25aa; Username:      {$SSH.USER}
  &#x25aa; Password:      {$SSH.PASSWORD}
  &#x25aa; Ex. script:    curl --unix-socket /var/run/docker.sock http://localhost/info</pre>
<figure class="wp-caption alignnone" id="attachment_32867" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_6-scaled.png"><img alt="" class="wp-image-32867 size-large" height="268" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_6-1024x429.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32867">Fig 6. Example of SSH agent item configuration for executing a script on the Docker server</figcaption></figure>
<p>You can also test the item and obtain the same data in JSON format, shown in Fig. 1.</p>
<figure class="wp-caption alignnone" id="attachment_32868" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_7-scaled.png"><img alt="" class="wp-image-32868 size-large" height="393" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_7-1024x628.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32868">Fig 7. Result of the item test</figcaption></figure>
<h4 id="the-right-way:-http-agent">The right way: HTTP agent</h4>
<p>For the other approach, we will be using an HTTP agent item to collect the data in bulk, using Docker API calls. For this, I don’t need to install the Zabbix agent on the Docker server. The authentication of this item will be performed using the certificates that have been copied over. Here are the important parameters in the item:</p>
<pre>Host: Docker server items 
Item #1
  &#x25aa; Name:             Get info
  &#x25aa; Type              SSH agent
  &#x25aa; Key:              docker.info    
  &#x25aa; Type of inf:      text
  &#x25aa; URL:              https://{$DOCKER.IP}:{$DOCKER.PORT}/info
  &#x25aa; SSL verify peer: check
  &#x25aa; SSL verify host: check
  &#x25aa; SSL cert. file:  {$SSL.CERTIFICATE.FILE}
  &#x25aa; SSL key file:    {$SSL.KEY.FILE}</pre>
<p>Do not forget to test the item (collected data should be the same as in Fig. 2) and add the item. If you have also encrypted the client private key (client-key.pem), you will also need to provide an SSL key password in the item configuration.</p>
<figure class="wp-caption alignnone" id="attachment_32869" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_8-scaled.png"><img alt="" class="wp-image-32869 size-large" height="817" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_8-802x1024.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32869">Fig 8. Example of the configured HTTP agent item</figcaption></figure>
<figure class="wp-caption alignnone" id="attachment_32870" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_9-scaled.png"><img alt="" class="wp-image-32870 size-large" height="69" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_9-1024x110.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32870">Fig 9. HTTP agent item collecting the data</figcaption></figure>
<h3 id="extracting-the-data">Extracting the data</h3>
<p>Now we can extract the important metrics by creating dependent items using the master item: Get info. Add a few dependent items to extract metrics, such as the total count, running, stopped, and paused containers. Item configuration parameters are given below the dependent item examples. The item &#8220;Containers running&#8221; parameter screenshots are shown below, together with the configuration parameters listed.</p>
<figure class="wp-caption alignnone" id="attachment_32871" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_10-scaled.png"><img alt="" class="wp-image-32871 size-large" height="263" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_10-1024x420.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32871">Fig 10. Dependent item tab to get the number of running containers</figcaption></figure>
<p>Tagging an item will also make your life easier for filtering when you have a legion of items.</p>
<figure class="wp-caption alignnone" id="attachment_32872" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_11-scaled.png"><img alt="" class="wp-image-32872 size-large" height="166" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_11-1024x266.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32872">Fig 11. Dependent item tag tab to get the number of running containers</figcaption></figure>
<p>In the preprocessing tab, we can use the JSONPath preprocessing step to extract the number of running containers from the master item.</p>
<figure class="wp-caption alignnone" id="attachment_32873" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_12-scaled.png"><img alt="" class="wp-image-32873 size-large" height="168" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_12-1024x269.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32873">Fig 12. Dependent item preprocessing tab to get the number of running containers</figcaption></figure>
<pre>Docker Host items
● Item #1
  &#x25aa; Name: 	Containers running	
  &#x25aa; Type 		Dependent item
  &#x25aa; Key: 		docker.containers.running	
  &#x25aa; Type of inf: 	Numeric (unsigned)
  &#x25aa; Master item	Docker: Get info
  &#x25aa; Units: 	!containers
&#x2666; Tags (name:value) 	
  &#x25aa; component:containers	
♯ Preprocessing
  &#x25aa; JSONPath  	$.ContainersRunning

● Item #2
  &#x25aa; Name: 	Containers paused	
  &#x25aa; Type 		Dependent item
  &#x25aa; Key: 		docker.containers.paused	
  &#x25aa; Type of inf: 	Numeric (unsigned)
  &#x25aa; Master item	Docker: Get info
  &#x25aa; Units: 	!containers
&#x2666; Tags (name:value) 	
  &#x25aa; component:containers	
♯ Preprocessing
  &#x25aa; JSONPath  	$.ContainersPaused

● Item #3
  &#x25aa; Name: 	Containers stopped	
  &#x25aa; Type 		Dependent item
  &#x25aa; Key: 		docker.containers.stopped	
  &#x25aa; Type of inf: 	Numeric (unsigned)
  &#x25aa; Master item	Docker: Get info
  &#x25aa; Units: 	!containers
&#x2666; Tags (name:value) 	
  &#x25aa; component:containers	
♯ Preprocessing
  &#x25aa; JSONPath  	$.ContainersStopped

● Item #4
  &#x25aa; Name: 	Containers total	
  &#x25aa; Type 		Dependent item
  &#x25aa; Key: 		docker.containers.total	
  &#x25aa; Type of inf: 	Numeric (unsigned)
  &#x25aa; Master item	Docker: Get info
  &#x25aa; Units: 	!containers
&#x2666; Tags (name:value) 	
  &#x25aa; component:containers	
♯ Preprocessing
  &#x25aa; JSONPath  	$.Containers
</pre>
<h3 id="creating-the-trigger">Creating the trigger</h3>
<p>I can also configure a trigger to receive a problem event in case some containers are not running. The screenshot of the trigger and parameter configuration is shown below.</p>
<figure class="wp-caption alignnone" id="attachment_32874" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_13-scaled.png"><img alt="" class="wp-image-32874 size-large" height="231" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_13-1024x370.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32874">Fig 13. Trigger configuration</figcaption></figure>
<pre>Trigger
◘ Trigger 
  &#x25aa; Name: 		Some containers are not running
  &#x25aa; Operational data: 	Total: {ITEM.LASTVALUE1}, Running: {ITEM.LASTVALUE2}
  &#x25aa; Severity: 		Warning
  &#x25aa; Expression: 		last(/Docker server/docker.containers.total)last(/Docker server/docker.containers.running)
  &#x25aa; PROBLEM event generation mode: Single
  &#x25aa; OK event closes: All problems
</pre>
<h3 id="getting-more-data">Getting more data</h3>
<p>Docker Engine also includes previous API versions. If no version of the API is specified in the URL, then the latest installed version will be used (using the API without a version is deprecated and will be removed in a future release). So even if you have the latest Docker installed (and you should always update to the latest version!), you can still use the older API calls by specifying the version (but once again, check what works).</p>
<p>Docker API offers several API calls that can be used to collect information about containers, images, container performance statistics, networks, volumes, or make changes to them.</p>
<p>Also, for more API calls, please explore this page: <a href="https://docs.docker.com/reference/api/engine/latest/">https://docs.docker.com/reference/api/engine/latest/.</a><br />
As an example, I will create another item to gather specific container information. The item configuration will differ from the one in the example in Fig.8 with the following parameters: different URL, item name, and key.</p>
<p>Here is an example of the ULR field (replace {$CONTAINER} with the existing container name):</p>
<pre>https://{$DOCKER.IP}:{$DOCKER.PORT}/containers/{$CONTAINER}/json</pre>
<figure class="wp-caption alignnone" id="attachment_32875" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_14-scaled.png"><img alt="" class="wp-image-32875 size-large" height="246" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_14-1024x393.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32875">Fig 14. HTTP agent item to get low-level information about a specific container: tomcat</figcaption></figure>
<p>You can also get the container performance data with a different URL. The item configuration will differ from the one in an example in Fig.8 with the following parameters: URL, item name and key. Here is an example of ULR field (replace {$CONTAINER} with the existing container name):</p>
<pre>https://{$DOCKER.IP}:{$DOCKER.PORT}/containers/{$CONTAINER}/stats?stream=false</pre>
<figure class="wp-caption alignnone" id="attachment_32876" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_15-scaled.png"><img alt="" class="wp-image-32876 size-large" height="220" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_15-1024x352.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32876">Fig 15. HTTP agent item to get performance information about a specific container: zabbix-server-mysql</figcaption></figure>
<h3 id="testing-the-trigger">Testing the trigger</h3>
<p>We can test if the data returned by the Docker API is as it seems, right? I have five containers created using the ‘docker run’ command, and one using the ‘docker compose’ command. Let&#8217;s stop the container made from the ‘docker run’ command and check if it will be reflected in the collected metrics.</p>
<figure class="wp-caption alignnone" id="attachment_32877" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_16-scaled.png"><img alt="" class="wp-image-32877 size-large" height="213" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_16-1024x340.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32877">Fig 16. The latest item data when stopping a Docker Compose container</figcaption></figure>
<p>As you can see in Figure 13, the stopped container shows up in the metrics collected by Zabbix through Docker API and in the Docker CLI. The Docker host item shows 1 stopped container and 5 running containers; the total number of containers is 6.</p>
<p>If you use the command &#8220;docker compose down&#8221; instead, the container will be stopped and removed altogether. That means, the total number of containers will also decrease by one, along with its status, as shown in Fig. 17. Therefore, make sure you understand what each command does and how it will impact your monitoring data.</p>
<figure class="wp-caption alignnone" id="attachment_32878" style="width: 640px;"><a href="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_17-scaled.png"><img alt="" class="wp-image-32878 size-large" height="219" src="https://blog.zabbix.com/wp-content/uploads/2026/04/DockerAPI1_17-1024x351.png" width="640" /></a><figcaption class="wp-caption-text" id="caption-attachment-32878">Fig 17. The latest item data when using Docker Compose down for a container</figcaption></figure>
<h3 id="in-summary">In summary</h3>
<p>Now you know more about how to collect the data from Docker using HTTP requests. Similar approaches can also be used to collect data from other applications through an API. You can select what metrics you want to extract, create triggers, graphs, or make a template if you wish.</p>
<p>&nbsp;</p>
<p>The post <a href="https://blog.zabbix.com/zabbix-and-the-docker-api-part-1-inspect/32860/">Zabbix and the Docker API, Part 1: Inspect</a> appeared first on <a href="https://blog.zabbix.com">Zabbix Blog</a>.</p>
