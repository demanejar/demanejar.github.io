---
title: Hướng dẫn airflow HA
author: longpt 
date: 2024-02-18 20:52:00 +0700
categories: [bigdata]
tags: [HA, Airflow]
math: true
mermaid: true
img_path: images
---

Trong bài này mình sẽ hướng dẫn cài đặt Airflow và cài HA cho nó. môi trường sử dụng là máy ảo virtualbox

<details markdown="1">
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


# Khái niệm 

- cho phép xây dựng flow các task
- cung cấp ui thuận tiện cho việc theo dõi và quản lí tập trung các task
- hạn chế: không sử dụng để truyền dữ liệu lớn giữa các task. không sử dụng với các task vô hạn (như streaming)

# Các thành phần của airflow

scheduler

executor -> worker 

<details markdown="1">
<summary>executor</summary>

các loại executor 

- local: local executor, sequential executor 
- remote: celery executor, kubernetes executor 

nên dùng celery executor vì nó có thể scale được số lượng các worker thông qua celery backend (rabbitMQ, redis). executor như kiểu một cách thức để task được giao từ scheduler tới worker (đứng giữa nơi lập lịch và nơi thực thi task)

![](https://longpt233.github.io/images/2024-02-22%2021-44-01.png)

</details>

ngoài ra: metadata db, dag dir, web server


# Cài airflow

<details markdown="1">
<summary>Cài airflow từ pip</summary>

```
sudo apt update
sudo apt install python3-pip -y

export AIRFLOW_HOME=~/airflow

AIRFLOW_VERSION=2.5.1
PYTHON_VERSION="$(python3 --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

echo $CONSTRAINT_URL

python3 -m pip install --upgrade pip (khi lỗi setup tools)
pip3 install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

hoặc đơn giản hơn (xong lỗi nhiều :v)
python3.8 -m pip install --upgrade pip
python3.8 -m pip install apache-airflow==2.5.1

```

</details>

<details markdown="1">
<summary>Khởi tạo db</summary>

```

python3 -m airflow db init 

# lỗi ModuleNotFoundError: No module named 'apt_pkg'
sudo apt-get install python-apt

# lúc này mới tạo ra cái thưu mục AIRFLOW_HOME
# mặc định sql_alchemy_conn = sqlite:////home/airflow/airflow2/airflow.db

# sửa db sang mysql rồi chạy lại db init
CREATE DATABASE airflow_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'airflow_user' IDENTIFIED BY 'airflow_pass';
GRANT ALL PRIVILEGES ON airflow_db.* TO 'airflow_user';
FLUSH PRIVILEGES;

vi airflow/airflow.cfg
mysql+mysqldb://airflow_db:airflow_pass@192.168.56.1:3306/airflow_db  # lỗi jh ấy
mysql+mysqlconnector://airflow_user:airflow_pass@192.168.56.1:3306/airflow_db
```

</details>

<details markdown="1">
<summary>Chạy lên các thành phần webserver , scheduler</summary>

```
airflow users create \
    --username admin \
    --firstname Peter \
    --lastname Parker \
    --role Admin \
    --email spiderman@superhero.org

# sau đó nhập mật khẩu

airflow webserver --port 8080 [-D, -h]

airflow scheduler [-D, -h]

# kill daemon
kill $(ps -ef | grep "gunicorn" | awk '{print $2}')

```

</details>

<details markdown="1">
<summary>Cài rabitmq</summary>

```
sudo apt update && sudo apt upgrade -y
sudo apt install rabbitmq-server -y
sudo systemctl enable rabbitmq-server
sudo systemctl start rabbitmq-server
sudo systemctl status rabbitmq-server
sudo rabbitmq-plugins enable rabbitmq_management
sudo rabbitmqctl add_user airflow airflow
sudo rabbitmqctl set_user_tags airflow administrator
```

</details>

<details markdown="1">
<summary>Đổi sang CeleryExecutor</summary>

tuy nhiên kiểu jh chạy cách kia cũng lỗi thôi ```The scheduler does not appear to be running. Last heartbeat was received 14 minutes ago. ```

đổi sang chuẩn executor khác xem sao

```
executor = CeleryExecutor
```

đổi celery broker + ressult backend

```
broker_url = redis://redis:6379/0
broker_url = amqp://airflow:airflow@192.168.56.10:5672/
result_backend = db+mysql://airflow_user:airflow_pass@192.168.56.1:3306/airflow_db

# set mysql+mysqlconnector lỗi
# cài thêm celery nếu chưa có 
pip install 'apache-airflow[celery]'


```

</details>

<details markdown="1">
<summary>Chạy flower để view job</summary>

Monitoring Tasks

```
# flower
# Celery Flower is a sweet UI for Celery. Airflow has a shortcut to start
# it ``airflow celery flower``. This defines the IP that Celery Flower runs on
flower_host = 0.0.0.0

# The root URL for Flower
# Example: flower_url_prefix = /flower
flower_url_prefix = /flower

# This defines the port that Celery Flower runs on
flower_port = 5556

# lỗi amqp.exceptions.NotAllowed: Connection.open: (530) NOT_ALLOWED - access to vhost '/' refused for user 'airflow'

sudo rabbitmqctl list_permissions -p / 
sudo rabbitmqctl set_permissions -p "/" "airflow" ".*" ".*" ".*"

```

lỗi mấy cái linh ta linh tinh vì chưa cài cái 

```
sudo apt-get install python3.8-dev
# Duy bảo mỗi lần cài phải đi theo bộ python3-pip python3.10-dev python3.10-venv

```

job khi chạy sẽ được đẩy vào queue

![](https://longpt233.github.io/images/2024-03-01%2014-56-52.png)

trên ui của airflow sẽ thấy task bị queue mà k chạy


tuy nhiên bên flower chưa hiện thị task do mình chưa bật worker

</details>

<details markdown="1">
<summary>Chạy worker</summary>

```
airflow celery worker

# chú ý: worker có thể được gắn với 1 queue
airflow celery worker -q spark,quark
```

![](https://longpt233.github.io/images/2024-03-01%2015-18-45.png)



</details>

# Mô hình ha


Web server. chạy trên con thứ 2 với config i hệt: vẫn lên được, chứng tỏ cứ chạy 2 con cùng lúc ha ip là dc. chứng tỏ webserver chỉ cần db là được ? (đồng bộ qua db)

Scheduler. chạy trên con chứ 2 mọi thứ vẫn bthg ở con 1 (flower, rabitMQ, worker trên con 1 vẫn nhận)

<details markdown="1">
<summary>Web server, Scheduler trên con thứ 2</summary>

```
ImportError: No module named 'mysql'
pip install mysql-connector-python-rf

sqlalchemy.exc.NotSupportedError: (mysql.connector.errors.NotSupportedError) Authentication plugin 'caching_sha2_password' is not supported
pip install mysql-connector-python

```

```
ModuleNotFoundError: No module named 'MySQLdb'
pip install mysqlclient
# vẫn lỗi 
sudo apt-get install python3.8-dev
# vẫn lỗi
sudo apt-get install python3-dev default-libmysqlclient-dev build-essential pkg-config

# cài thêm celery nếu chưa có 
pip install 'apache-airflow[celery]'

```

</details> 



Message queue, mysql có cách HA riêng 

Vì Scheduler lúc 2 con chạy job sẽ bị x2, nên bắt buộc chỉ có 1 con chạy thui -> cần HA Scheduler


```
pip3 install git+https://github.com/teamclairvoyant/airflow-scheduler-failover-controller.git@v1.0.8
scheduler_failover_controller init  # append them config HA vào airflow.cfg
scheduler_failover_controller start

# mấy cái lệnh chạy, stop có trong file config
# chú ý config scheduler_nodes_in_cluster List of potential nodes that can act as Schedulers (Comma Separated List)
# chuyển log về debug sẽ thấy cái scheduler_failover_controller check bằng 
Running Command: ps -eaf | grep 'airflow scheduler'
```

<details markdown="1">
<summary>lỗi không thể chạy lên </summary>


```

in configuration.py:     


def get_sql_alchemy_conn(self):
        return self.get_config("core", "SQL_ALCHEMY_CONN")


but maybe airflow.cfg, the section is [database]. so that engine can be null

add sql_alchemy_conn both [database], [core] in airflow.cfg can fix this issue

https://github.com/teamclairvoyant/airflow-scheduler-failover-controller/issues/43
```
</details> 



# Note 

- xcom (cross-communications) để truyền thông tin giữa các task trong dag
- get_pty=True để kill được task trên ui
- do_xcom_push=False để tránh trường hợp xcom lưu output> 65kb dẫn đến task fail(sử  dụng print hoặc return function)



# Ref 

[lotus doc - hiephm](https://lotus.vn/w/blog/gioi-thieu-ve-airflow-va-trien-khai-kien-truc-ha-348074727123714048.htm)

[git  ha](https://github.com/teamclairvoyant/airflow-scheduler-failover-controller)

[doc](https://airflow.apache.org/docs/apache-airflow/stable/start.html)

[phân loại executor](https://viblo.asia/p/hieu-don-gian-ve-airflow-executor-3kY4g52yLAe)

[bài viết gốc - longpt233](https://longpt233.github.io/airflow-ha/)






