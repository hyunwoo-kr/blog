---
title: "Demo - WEB/WAS Server Provisioning - 1차"
date: 2021-03-11T16:32:08+09:00
draft: true
categories:
- infra
tags:
- Apache HTTP Server
- Tomcat
- mod_jk
#thumbnailImage: //example.com/image.jpg
author: "hyunwoo"
---
Apache HTTP Server + Tomcat 구성을 Code로 관리 해 보자
<!--more-->


Getting Started
----------------------------------
이 데모는 최소한의 노력으로 WEB/WAS 환경을 구축하는 방법에 대한 안내를 제공 합니다.

다음 사항에 대하여 다룹니다.
 - Vargant 및 HyperV로 Server Provisioning
 - Ansible playbook 및 Host inventory 파일로 Application Provisioning
 - Apache HTTP Server와 Tomcat 연결
 - Tomcat Clustering 설정
 - Apache HTTP Server에 SSL 환경 구성하기.

Prerequisites
---------------------------
따라 하려면 다음 프로그램을 미리 사용 할 수 있어야 합니다.
 - Vagrant
 - Window 10 Hyper-V

Vagarnt 및 Hyper-v로 Server Provisioning 하기
-----------------------------------------------

![그림1](/img/infra/demo01/vagrant_server_provisioning.JPG)
**그림 1: Server Provisioning with Vagrant and HyperV.**

- Ansible: Application Provisioning에 사용
- Apache HTTP Server: 웹 서버
- Tomcat1: WAS1
- Tomcat2: WAS2
- Apache HTTP Server와 Tomcat Server간에는 mod_jk 방식으로 연결하고, Tomcat 1,2는 클러스터링 구성을 진행 한다

Vagrant configuraion File
--------------------------------------------
File location: [Github /infra/4_demo01/1_vagrant/config_multi_nodes.yaml](https://github.com/hyunwoo-kr/infra/tree/main/3_apache_http-server_compile_on_centos7)

```
vagrant_box: centos_openssl/7
vagrant_ip: 161.100.6.
vagarnt_hostname: vagrant
vagrant_memory: 1024
vagrant_directory: /vagrant
vagrant_cpu: 1
vagrant_box_check_update: false
vagrant_domain_name: sample.com
```

Building the servers with Vagrantfile
-----------------------------------------------------
4개의 이미지를 빌드
 - Apache HTTP Server
 - Tomcat Server 1
 - Tomcat Server 2
 - Ansible

File location: [Github /infra/4_demo01/1_vagrant/Vagrantfile](https://github.com/hyunwoo-kr/infra/tree/main/3_apache_http-server_compile_on_centos7)

```
require 'yaml'
dir = File.dirname(File.expand_path(__FILE__))
config_nodes = "#{dir}/config/config_multi_nodes.yaml"

if !File.exist?("#{config_nodes}")
  raise 'Configuration file is missing! Please make sure that the configuration exists and try again.'
end
vconfig = YAML::load_file("#{config_nodes}")

BRIDGE_NET = vconfig['vagrant_ip']
DOMAIN = vconfig['vagrant_domain_name']
RAM = vconfig['vagrant_memory']

servers=[
  {
    :hostname => "ansible." + "#{DOMAIN}",
    :ip => "#{BRIDGE_NET}" + "10",
    :ram => "#{RAM}",
	:install_ansible => "./scripts/install_ansible.sh",
	:config_ansible => "./scripts/config_ansible.sh",
	:source =>  "./.",
	:destination => "/home/vagrant/"
  },
  {
    :hostname => "dmz-web." + "#{DOMAIN}",
    :ip => "#{BRIDGE_NET}" + "21",
    :ram => "#{RAM}"
  },
  {
    :hostname => "was1." + "#{DOMAIN}",
    :ip => "#{BRIDGE_NET}" + "31",
    :ram => "#{RAM}"
  },
  {
    :hostname => "was2." + "#{DOMAIN}",
    :ip => "#{BRIDGE_NET}" + "32",
    :ram => "#{RAM}"
  }
]

Vagrant.configure(2) do |config|
    config.vm.synced_folder ".", vconfig['vagrant_directory'], :mount_options => ["dmode=777", "fmode=666"], disabled: true

    servers.each do |machine|
        config.vm.define machine[:hostname] do |node|
			node.vm.box = vconfig['vagrant_box']
            node.vm.box_check_update = machine[:vagrant_box_check_update]

			node.vm.hostname = machine[:hostname]
            node.vm.network "public_network", auto_config: false, bridge: "dmz", ip: machine[:ip]

            node.vm.provider "hyperv" do |vb|
                vb.auto_start_action= "StartIfRunning"
                vb.auto_stop_action = "TurnOff"

				vb.cpus = vconfig['vagrant_cpu']
				vb.memory = machine[:ram]
                vb.vmname = machine[:hostname]

                if (!machine[:install_ansible].nil?)
                  if File.exist?(machine[:install_ansible])
					node.vm.provision :shell, path: machine[:install_ansible]
                  end
                  if File.exist?(machine[:config_ansible])
					node.vm.provision :file, source: machine[:source] , destination: machine[:destination]
      			    node.vm.provision :shell, privileged: false, path: machine[:config_ansible]
                  end
                end
            end
        end
    end
end


```