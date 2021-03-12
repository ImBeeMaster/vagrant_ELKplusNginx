# vagrant_ELKplusNginx

Vagrant virtualbox configuration for the ELK stack<br/>
(Elasticsearch,Logstash,Kibana + filebeat) and two nginx web servers as log producers

Architecture:
ELK stack consists of 3 nodes, to separate E,L&K from each other; Such approach is solely educational 
and no other need was pusued but to try out distributed system. 
Elasticsearch consists of single master node.<br/>

System requirements:<br/>

RAM: 8192+2048+512*3 = up to 12 GB <br/>
(requested for vb's, but actual consumption depends on actual workload)<br/>

CPU:<br/>
4.5 Cores<br/>
(requested for vb's, but actual consumption depends on actual workload)<br/>

To do:<br/>
System logs gathering<br/>
Destributed Elasticsearch cluster<br/>
