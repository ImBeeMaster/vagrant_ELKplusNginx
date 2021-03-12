# vagrant_ELKplusNginx

Vagrant virtualbox configuration for the ELK stack
(Elasticsearch,Logstash,Kibana + filebeat) and two nginx web servers as log producers

Architecture:
ELK stack consists of 3 nodes, to separate E,L&K from each other; Such approach is solely educational 
and no other need was pusued but to try out distributed system. 
Elasticsearch consists of single master node.

System requirements:

RAM: 8192+2048+512*3 = up to 12 GB 
(requested for vb's, but actual consumption depends on actual workload)

CPU:
4.5 Cores
5(requested for vb's, but actual consumption depends on actual workload)

To do:
System logs gathering
Destributed Elasticsearch cluster
