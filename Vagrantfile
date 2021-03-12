#Reserverd ports:
# Logstash: 5044
# Kibana: 5601
# Elasticsearch: 9200
# Nginx: 80

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-18.04"
  config.ssh.insert_key = false



  config.vm.provider "virtualbox" do |box|
     
     box.linked_clone = true
    # Display the VirtualBox GUI when booting the machine
     box.gui = false

  end  
  config.vm.define "elasticsearch" do |vb|
    # vb.vm.synced_folder ".", "/vagrant", type: "smb", smb_username: "Oleksii_Pasichnyk", mount_options: ["username=Oleksii_Pasichnyk"]
    vb.vm.synced_folder ".", "/home/vagrant/shared"
    vb.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "8192"]
      v.cpus = 2
      end
    vb.vm.network "private_network", ip: "10.0.1.2"
    vb.vm.hostname = "elasticsearch"
    vb.vm.provision "file", source: "./hosts", destination: "/tmp/hosts"
    vb.vm.provision "shell", inline: <<-SHELL
      sed -i 's|#DNS=|DNS=8.8.8.8|' /etc/systemd/resolved.conf && sed -i 's|#FallbackDNS=|FallbackDNS=1.1.1.1|' /etc/systemd/resolved.conf && systemctl restart systemd-resolved
      cat /tmp/hosts >> /etc/hosts

      wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
      apt install apt-transport-https
      echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
      apt update && apt install elasticsearch

    SHELL
    vb.vm.provision "elasticsearch_update_conf", type:"file", source: "./elasticsearch/elasticsearch.yml", destination: "/tmp/elasticsearch.yml", run: "always"
    vb.vm.provision "elasticsearch_apply_conf", type: "shell", inline: "mv /tmp/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml && chown root:root /etc/elasticsearch/elasticsearch.yml && chmod 644 /etc/elasticsearch/elasticsearch.yml", run: "always"

    vb.vm.provision "shell", inline: <<-SHELL
      systemctl daemon-reload 
      systemctl enable elasticsearch
      systemctl restart elasticsearch
      systemctl status elasticsearch
      sleep 10 && curl 10.0.1.2:9200
    SHELL

  end
  config.vm.define "kibana" do |vb|
    # vb.vm.synced_folder ".", "/vagrant", type: "smb", smb_username: "Oleksii_Pasichnyk", mount_options: ["username=Oleksii_Pasichnyk"]
    vb.vm.synced_folder ".", "/home/vagrant/shared"
    vb.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "2048"]
      end
    vb.vm.network "private_network", ip: "10.0.1.3"
    vb.vm.network "forwarded_port", guest_ip: "10.0.1.3", guest: 5601, host_ip: "127.0.0.1", host: 5601
    vb.vm.hostname = "kibana"
    vb.vm.provision "file", source: "./hosts", destination: "/tmp/hosts"
    vb.vm.provision "shell", inline: <<-SHELL
      sed -i 's|#DNS=|DNS=8.8.8.8|' /etc/systemd/resolved.conf && sed -i 's|#FallbackDNS=|FallbackDNS=1.1.1.1|' /etc/systemd/resolved.conf && systemctl restart systemd-resolved
      cat /tmp/hosts >> /etc/hosts
    SHELL

    vb.vm.provision "shell", inline: <<-SHELL
      wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
      echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list
      apt update && apt install kibana
    SHELL
    vb.vm.provision "kibana_update_conf", type: "file", source: "./kibana/kibana.yml", destination: "/tmp/", run: "always"
    vb.vm.provision "shell", inline: 'mv /tmp/kibana.yml /etc/kibana/', run: "always"
    vb.vm.provision "kibana_conf_backup", type: "file", source: "./kibana/index_backup.ndjson", destination: "/tmp/index_backup.ndjson"

    vb.vm.provision "shell", run: "always", inline: <<-SHELL
      systemctl daemon-reload 
      systemctl enable kibana
      systemctl restart kibana
      systemctl status kibana
      curl -X GET "elasticsearch:9200/_cat/health?v&pretty"
    SHELL
    vb.vm.provision "shell", inline: 'sleep 20 && curl -X POST kibana:5601/api/saved_objects/_import?overwrite=true -H "kbn-xsrf: true" --form file=@/tmp/index_backup.ndjson'
  end

  config.vm.define "logstash" do |vb|

    vb.vm.synced_folder ".", "/home/vagrant/shared"
    vb.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "512", "--cpuexecutioncap", "50"]
      end
    vb.vm.network "private_network", ip: "10.0.1.4"
    vb.vm.hostname = "logstash"
    
    vb.vm.provision "file", source: "./hosts", destination: "/tmp/hosts"
    vb.vm.provision "shell", inline: <<-SHELL
      sed -i 's|#DNS=|DNS=8.8.8.8|' /etc/systemd/resolved.conf && sed -i 's|#FallbackDNS=|FallbackDNS=1.1.1.1|' /etc/systemd/resolved.conf && systemctl restart systemd-resolved
      cat /tmp/hosts >> /etc/hosts
    SHELL

    vb.vm.provision "shell", inline: <<-SHELL
      wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
      echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list
      apt update && apt install logstash
    SHELL
    vb.vm.provision "logstash_copy_conf", type: "file", source: "./logstash/conf.d/", destination: "/tmp/logstash/", run: "always"
    vb.vm.provision "logstash_apply_conf", type: "shell", inline: "mv /tmp/logstash/conf.d/* /etc/logstash/conf.d/ && chown root:root /etc/logstash/conf.d/* && chmod 644 /etc/logstash/conf.d/*", run: "always"

    vb.vm.provision "logstash_update_main_conf", type: "file", source: "./logstash/logstash.yml", destination: "/tmp/logstash.yml", run: "always"
    vb.vm.provision "shell", inline: "mv /tmp/logstash.yml /etc/logstash/logstash.yml && chown root:root /etc/logstash/logstash.yml* && chmod 644 /etc/logstash/logstash.yml", run: "always"
    vb.vm.provision "shell", inline: <<-SHELL
      systemctl daemon-reload 
      systemctl enable logstash
      systemctl restart logstash
      systemctl status logstash
    SHELL
  end

  config.vm.define "nginx1" do |vb|
    vb.vm.synced_folder ".", "/home/vagrant/shared"
    vb.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "512", "--cpuexecutioncap", "50"]
      end
    vb.vm.network "private_network", ip: "10.0.1.5"
    vb.vm.hostname = "web-server-1"

    vb.vm.provision "file", source: "./hosts", destination: "/tmp/hosts"
    vb.vm.provision "shell", inline: <<-SHELL
      sed -i 's|#DNS=|DNS=8.8.8.8|' /etc/systemd/resolved.conf && sed -i 's|#FallbackDNS=|FallbackDNS=1.1.1.1|' /etc/systemd/resolved.conf && systemctl restart systemd-resolved
      cat /tmp/hosts >> /etc/hosts
    SHELL
    vb.vm.provision "shell", inline: <<-SHELL
      wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
      echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
      apt update
      apt-get install -y nginx filebeat
    SHELL
    vb.vm.provision "filebeat_update_conf", type: "file", source: "./filebeat/filebeat.yml", destination: "/tmp/", run: "always"
    vb.vm.provision "shell", inline:  "mv /tmp/filebeat.yml /etc/filebeat/filebeat.yml && chown root:root /etc/filebeat/filebeat.yml && chmod 644 /etc/filebeat/filebeat.yml", run: "always"

    vb.vm.provision "shell", run: "always", inline: <<-SHELL
      systemctl enable filebeat
      systemctl restart filebeat
      systemctl enable nginx
      systemctl start nginx
    SHELL
    #execute nginx check, will send logs to logstash so will check elk as well
    vb.vm.provision "shell", path: "./nginx_check.sh", run: "always"
  end

  config.vm.define "nginx2" do |vb|

    vb.vm.synced_folder ".", "/home/vagrant/shared"
    vb.vm.provider :virtualbox do |v|
      v.customize ["modifyvm", :id, "--memory", "512", "--cpuexecutioncap", "50"]
      end
    vb.vm.network "private_network", ip: "10.0.1.6"
    vb.vm.hostname = "web-server-2"

    vb.vm.provision "file", source: "./hosts", destination: "/tmp/hosts"
    vb.vm.provision "shell", inline: <<-SHELL
      sed -i 's|#DNS=|DNS=8.8.8.8|' /etc/systemd/resolved.conf && sed -i 's|#FallbackDNS=|FallbackDNS=1.1.1.1|' /etc/systemd/resolved.conf && systemctl restart systemd-resolved
      cat /tmp/hosts >> /etc/hosts
    SHELL

    vb.vm.provision "shell", inline: <<-SHELL
      wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
      echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
      apt update
      apt-get install -y nginx filebeat
    SHELL
    vb.vm.provision "filebeat_update_conf", type: "file", source: "./filebeat/filebeat.yml", destination: "/tmp/", run: "always"
    vb.vm.provision "shell", inline:  "mv /tmp/filebeat.yml /etc/filebeat/filebeat.yml && chown root:root /etc/filebeat/filebeat.yml && chmod 644 /etc/filebeat/filebeat.yml", run: "always"

    vb.vm.provision "shell", run: "always", inline: <<-SHELL
      systemctl enable filebeat
      systemctl restart filebeat
      systemctl enable nginx
      systemctl start nginx
    SHELL
    #execute nginx check, will send logs to logstash so will check elk as well
    vb.vm.provision "shell", path: "./nginx_check.sh", run: "always"
  end
end
