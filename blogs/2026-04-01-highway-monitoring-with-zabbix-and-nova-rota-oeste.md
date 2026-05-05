---
title: "Highway Monitoring with Zabbix and Nova Rota Oeste"
url: "https://blog.zabbix.com/highway-monitoring-with-zabbix-and-nova-rota-oeste/32810/"
date: "Wed, 01 Apr 2026 07:02:54 +0000"
author: "Michael Kammer"
feed_url: "https://blog.zabbix.com/feed/"
---
<img alt="" class="webfeedsFeaturedVisual wp-post-image" height="150" src="https://blog.zabbix.com/wp-content/uploads/2026/03/Nova_Rota_Oeste_case_study_880x516-150x150.png" style="display: block; margin-bottom: 5px; clear: both;" width="150" /><p><a href="https://novarotadooeste.com.br/">Nova Rota do Oeste</a> (formerly Rota do Oeste) is a Brazilian highway concessionaire founded in 2014, responsible for managing more than 850 kilometers of highway that connects the cities of Sinop (MT), to the states of Mato Grosso and Mato Grosso do Sul.</p>
<p><span id="more-32810"></span></p>
<p>Due to the demands of ensuring road safety, Nova Rota do Oeste restructured its highway monitoring strategy by replacing a legacy tool with Zabbix, achieving greater visibility and operational control as well as 85% cost savings.</p>
<h3 id="the-challenge">The challenge</h3>
<p>Nova Rota do Oeste operates an enormous technological infrastructure, including:</p>
<ul>
<li>10 data centers</li>
<li>833 surveillance cameras</li>
<li>Radio repeater towers for vehicle communication</li>
<li>Electronic toll systems</li>
<li>An extensive optical fiber infrastructure</li>
<li>Intelligent signalling equipment</li>
</ul>
<p>In total, the company manages nine toll plazas and maintains 24/7 operations focused on traffic safety and flow. It is also responsible for the largest highway construction project in Brazil.</p>
<p>Nova Rota do Oeste previously relied on a licensed commercial legacy monitoring tool to support its infrastructure, but it did not meet the need for constant scalability.</p>
<p>In 2024, the company began searching for a new highway monitoring solution that met requirements such as scalability, robustness, ease of learning, and training availability. Key needs included the ability to:</p>
<ul>
<li>Monitor critical devices and services distributed along 850 km of highway</li>
<li>Provide operational visibility into communications status, connectivity, and systems</li>
<li>Reduce incident identification and response time</li>
<li>Create automated alerts for proactive action</li>
<li>Integrate infrastructure monitoring, road safety, and business systems</li>
</ul>
<p>After market research and internal technical analysis, the company concluded that Zabbix was the most suitable tool for its challenges. The migration process from the commercial legacy tool then began.</p>
<h3 id="the-solution">The solution</h3>
<p>The infrastructure monitoring transformation approach was structured in complementary phases, led by Zabbix Premium Delivery Partner <a href="https://www.jlcp.com.br/">JLCP</a> in partnership with the Nova Rota do Oeste IT team.</p>
<p><strong>Phase 1: Foundation and base configuration</strong></p>
<p>The first stage focused on gathering requirements and building the foundation. After analyzing the existing environment and cleaning up legacy data, the necessary architecture was defined, culminating in the implementation and validation of the Zabbix infrastructure.</p>
<p><strong>Phase 2: Priority-based expansion</strong></p>
<p>The second phase focused on the incremental onboarding of devices based on criticality, including:</p>
<ul>
<li>Multiple servers</li>
<li>833 surveillance cameras</li>
<li>Radio repeater towers</li>
<li>Toll system</li>
<li>Power infrastructure</li>
<li>Solar plants via customized API</li>
<li>Wi-Fi equipment (Unifi API integration</li>
<li> Optical fiber (testing phase)</li>
</ul>
<p><strong>Monitoring the implementation</strong></p>
<p>The highway monitoring system implemented at Nova Rota do Oeste follows a distributed, layered architecture that integrates different areas into Zabbix:</p>
<ul>
<li>IT infrastructure</li>
<li>Operating systems</li>
<li>Field devices</li>
<li>Business services</li>
</ul>
<p>Each layer has specific objectives, connected within a unified ecosystem of data and alerts. Data collected from monitored items is processed by the central Zabbix server, which applies triggers and conditions to identify anomalies. When a metric exceeds a configured threshold, Zabbix generates automatic alerts displayed in real time to the infrastructure team via dashboards that remain constantly open in the operations center.</p>
<p>Data is organized into visual dashboards that provide:</p>
<ul>
<li>Data center view (consolidated status of each of the 10 locations)</li>
<li>Device-type view (cameras, towers, servers, power)</li>
<li>Criticality view (essential versus secondary services)</li>
<li>Topological maps visually representing the highway and its components</li>
</ul>
<p>The team now has real-time visibility over more than 1,700 hosts distributed along the highway.</p>
<h3 id="results">Results</h3>
<p>The implementation of Zabbix delivered significant operational improvements, including:</p>
<ul>
<li>An 80–85% reduction in detection time – from an average of 1 hour to less than 15 minutes</li>
<li>85% cost savings compared to maintaining the legacy tool</li>
<li>Full coverage of critical infrastructure with 1,702 hosts currently monitored</li>
<li>Complete operational visibility over 10 data centers, 833 cameras, radio towers, power systems, and more</li>
<li>Internal notification before users or operators report issues</li>
<li>Direct dispatch of field teams or ticket opening with telecom providers</li>
<li>Early communication of potential impacts</li>
<li>Activation of contingency plans when necessary</li>
</ul>
<p>In addition to support for the development and integration of all supported Zabbix items, Nova Rota do Oeste also obtained a <a href="https://www.zabbix.com/support">Zabbix Technical Support Subscription</a>, expanding the tool’s scope and ensuring better usage and technical support.</p>
<h3 id="conclusion">Conclusion</h3>
<p>Monitoring with Zabbix increased the availability of radio towers for vehicle communication, enabled real-time surveillance camera status visualization, guaranteed connectivity for toll systems, and provided continuous monitoring of power infrastructure, all of which has directly contributed to better service and user safety.</p>
<p>&nbsp;</p>
<p>The post <a href="https://blog.zabbix.com/highway-monitoring-with-zabbix-and-nova-rota-oeste/32810/">Highway Monitoring with Zabbix and Nova Rota Oeste</a> appeared first on <a href="https://blog.zabbix.com">Zabbix Blog</a>.</p>
