---
title: Hướng dẫn airflow ha
author: lonpgt 
date: 2001-02-18 20:52:00 +0700
categories: [bigdata]
tags: [HA, Airflow]
math: true
mermaid: true
img_path: images
---

Trong bài này mình sẽ hướng dẫn cài đặt Airflow và cài HA cho nó. môi trường sử dụng là máy ảo virtualbox

<details>
<summary>tạo 2 máy ảo với địa chỉ cố định, add user và cập quyền ssh từ host</summary>

```
VAGRANT_COMMAND = ARGV[0]

Vagrant.configure("2") do |config|

    if VAGRANT_COMMAND == "ssh"
      config.ssh.username = 'vagrant'
    end
    config.vm.box = "ubuntu/bionic64" # Chọn box bạn muốn sử dụng

    # Khởi tạo máy ảo thứ nhất
    config.vm.define "machine1" do |machine1|
    machine1.vm.network "private_network", ip: "192.168.56.2"
    machine1.vm.provider "virtualbox" do |vb|
          vb.memory = "2048" # 2GB RAM
          vb.cpus = 1       # 1 core CPU
        end

    machine1.vm.provision "shell", inline: <<-SHELL
          adduser airflow
          sudo su - airflow -c $'\
          whoami && \
          mkdir .ssh && \
          echo "ssh-rsa xxxx" > .ssh/authorized_keys && \
          chmod 700 .ssh && \
          chmod 600 .ssh/authorized_keys && \
          file_path=".ssh/authorized_keys" && \
          echo "cat file $file_path after make change" && \
          cat $file_path '
        SHELL
    end
end
```

</details>

![alo](2024-02-18_air.png)


